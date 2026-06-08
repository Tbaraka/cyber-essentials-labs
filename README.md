# Lab 01 – NetFlow Traffic Observation

## Overview

This lab uses Cisco Packet Tracer to generate network traffic and analyse the corresponding NetFlow flow records captured by a NetFlow collector. The focus is on understanding how NetFlow represents traffic unidirectionally, how different protocols produce different flow signatures, and how DNS resolution adds observable flows when a URL is used instead of a raw IP address.

**Tool:** Cisco Packet Tracer  
**Topic:** Network Traffic Monitoring  
**Cyber Essentials Control:** Network Firewalls — understanding traffic flows is foundational to monitoring, filtering, and securing network boundaries.

---

## Network Topology

\[PC-1\]──┐

\[PC-2\]──┤                          ┌──\[Web Server 192.0.2.100\]

\[PC-3\]──┼──\[Switch\]──\[Edge Router\]─┤

\[PC-4\]──┘                          └──\[NetFlow Collector\]

         LAN: 10.0.0.0/24    WAN: 192.0.2.0/24

| Device | Interface | IP Address | Role |
| :---- | :---- | :---- | :---- |
| Edge Router | Gig0/0 | 10.0.0.1 | Default gateway / NetFlow exporter |
| Edge Router | Se0/0/1 | 192.0.2.254 | WAN interface |
| PC-1 | NIC | 10.0.0.10 | Client host |
| PC-2 to PC-4 | NIC | 10.0.0.x | Additional client hosts |
| Web Server | NIC | 192.0.2.100 | HTTP server |
| NetFlow Collector | NIC | 10.0.0.100 | Flow record display |

**NetFlow exporter configuration:** Gig0/0 monitors inbound LAN traffic. Se0/0/1 monitors inbound WAN traffic. This captures both directions of sessions that cross the router.

---

## Part 1 – ICMP Flow Records (One Direction)

### What I did

Pinged the default gateway (10.0.0.1) from PC-1, then from PC-2, PC-3, and PC-4 in turn.

### Key observations

**Single host ping (PC-1 → 10.0.0.1)**

| Field | Value |
| :---- | :---- |
| Source IP | 10.0.0.10 |
| Destination IP | 10.0.0.1 |
| Source Port | 0 |
| Destination Port | 0 |
| IP Protocol | 1 (ICMP) |
| TCP Flags | 0x00 |
| Counter Packets | 4 |
| Interface Input | Gig0/0 |
| Interface Output | Null |

**Why output is Null:** The ping destination *is* the router interface itself. Packets never exit through an output interface — they terminate on the router. NetFlow only records the ingress direction here.

**Why ports are 0:** ICMP does not use transport layer ports. NetFlow still records the field but populates it with zero.

**Multi-host pings**

Each host that pinged the gateway created a **separate flow segment** in the pie chart rather than merging into the existing flow. This is because NetFlow identifies a flow by the 5-tuple:

Source IP | Destination IP | Source Port | Destination Port | Protocol

PC-2's source IP differs from PC-1's, so it is a distinct flow. With four hosts pinging, the collector showed four equal segments (25% each).

**Repeat ping from PC-1**

Repeating the ping from PC-1 did not create a new flow  it updated the packet and byte counters in the existing flow record. Same 5-tuple \= same flow.

---

## Part 2 – HTTP Session Flows (Bi-directional)

### Step 1 – Access by IP address (192.0.2.100)

**Predicted flows before testing:**

| Flow | Source | Destination | Port | Protocol |
| :---- | :---- | :---- | :---- | :---- |
| 1 | 10.0.0.10 | 192.0.2.100 | 80 | TCP |
| 2 | 192.0.2.100 | 10.0.0.10 | 1025 | TCP |

**Actual result:** Two HTTP flow segments appeared as predicted — the request leaving the LAN (captured on Gig0/0) and the response entering from the WAN (captured on Se0/0/1).

**HTTP Request flow (PC-1 → Web Server)**

| Field | Value |
| :---- | :---- |
| Source IP | 10.0.0.10 |
| Destination IP | 192.0.2.100 |
| Source Port | 1025 (ephemeral) |
| Destination Port | 80 |
| IP Protocol | 6 (TCP) |
| TCP Flags | 0x02 |
| Interface Input | Gig0/0 |
| Interface Output | Se0/0/1 |

**HTTP Response flow (Web Server → PC-1)**

| Field | Value |
| :---- | :---- |
| Source IP | 192.0.2.100 |
| Destination IP | 10.0.0.10 |
| Source Port | 80 |
| Destination Port | 1025 (ephemeral) |
| IP Protocol | 6 (TCP) |
| TCP Flags | 0x02 |
| Interface Input | Se0/0/1 |
| Interface Output | Gig0/0 |

**Notable:** Source and destination ports are exactly swapped between request and response flows. The interfaces are also inverted — what was input becomes output. This demonstrates NetFlow's unidirectional nature: two records are needed to fully represent one session.

**Copyrights page link**

Clicking the Copyrights link generated a new set of flow segments with a **different ephemeral source port** on the client side. Even though the same hosts were communicating, the browser opened a new TCP connection with a new port number, which produced a new 5-tuple and therefore a new flow record.

---

### Step 2 – Access by URL ([www.example.com](http://www.example.com))

**Predicted flows before testing:**

Accessing by URL requires DNS resolution first, so I expected at least one additional flow for DNS before the HTTP flows appeared.

**Actual result:** Four flows appeared — DNS query, DNS response, HTTP request, HTTP response.

**DNS Response flow (most revealing)**

| Field | Value |
| :---- | :---- |
| Source IP | 192.0.2.254 |
| Destination IP | 10.0.0.10 |
| Source Port | 53 |
| Destination Port | 1027 |
| IP Protocol | **17 (UDP)** |
| TCP Flags | 0x00 |
| Counter Packets | 1 |
| Interface Input | Se0/0/1 |
| Interface Output | Gig0/0 |

**Key difference from HTTP flows:** Protocol is 17 (UDP) not 6 (TCP). DNS uses UDP by default for standard queries. TCP flags are 0x00 — no TCP session involved. A single packet carried the DNS response.

**Why the source IP is 192.0.2.254 (the router) not 192.0.2.100 (the web server):** The router's WAN interface is acting as the DNS resolver or forwarding the DNS response back into the LAN. The DNS server and web server are separate services at different addresses.

---

## Protocol Summary

| Protocol | IP Protocol Number | Transport | Ports |
| :---- | :---- | :---- | :---- |
| ICMP | 1 | — | 0 / 0 |
| TCP | 6 | TCP | Varies |
| UDP | 17 | UDP | 53 (DNS) |

---

## What I Learned

1. **NetFlow is unidirectional.** A single TCP session always produces at least two flow records — one for each direction. Monitoring only one interface direction gives an incomplete picture of traffic.

2. **The 5-tuple uniquely identifies a flow.** Any change in source IP, destination IP, source port, destination port, or protocol creates a new flow. This is why each host and each new browser connection produced separate records.

3. **Protocol choice is visible in NetFlow.** ICMP, TCP, and UDP each leave a distinct signature. Protocol 17 with port 53 is immediately recognisable as DNS without needing deep packet inspection.

4. **URL vs IP access reveals DNS traffic.** Accessing a server by hostname exposes DNS resolution in the flow data. In a real environment, unexpected DNS flows (wrong source, unusual destination, high frequency) are a common indicator of compromise or data exfiltration.

5. **Ephemeral ports change per connection.** Each new TCP session from a client uses a fresh source port, creating a new flow record even between the same two hosts. Understanding this prevents false assumptions that "same hosts \= same flow."

## What I Would Do Differently

- Configure the NetFlow exporter to capture both ingress and egress on a single interface, then compare the output to the split-interface approach used here.  
- Extend the lab to generate UDP-based application traffic (e.g. DNS queries to an external resolver) and observe how the lack of TCP handshake affects flow record completeness.  
- Use Wireshark alongside NetFlow to correlate raw packet captures with flow summaries, reinforcing where NetFlow loses granularity.

---

## Files

| File | Description |
| :---- | :---- |
| `topology/netflow-lab.pkt` | Packet Tracer source file |
| `topology/topology.png` | Network diagram |
| `screenshots/` | Flow record captures for each part |

