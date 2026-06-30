# Network Layer — Core Concepts (Beyond IP Addressing)

---

## 1. What the Network Layer Does

Moves packets from source host to destination host across multiple hops (routers). Sits between Data Link (hop-by-hop) and Transport (end-to-end).

Two jobs:

- **Forwarding** — move packet from router's input interface to correct output interface. Local, per-router decision. Done in nanoseconds via forwarding table lookup.
- **Routing** — determine the end-to-end path packets should take. Network-wide logic. Routing algorithms populate forwarding tables.

> Analogy: Routing = planning the trip. Forwarding = taking each turn at intersections.

---

## 2. Network Service Models

| Model | Example | Guarantees |
|-------|---------|------------|
| **Connectionless (datagram)** | IP | Best-effort. No bandwidth, ordering, or reliability guarantees. |
| **Connection-oriented (virtual circuit)** | ATM, MPLS | Per-call bandwidth, in-order delivery, bounded delay. |

Internet chose datagram model. Simple network core, complexity pushed to edges (end hosts handle reliability via TCP).

### Virtual Circuit Setup

1. Source sends setup request (like dialing a call).
2. Each router along path allocates VC number, reserves resources.
3. Data flows using short VC IDs in packet headers (not full addresses).
4. Teardown when done.

**Trade-off:** VC gives predictable performance but underutilized resources when idle. Datagram gives statistical multiplexing gain — bursty traffic shares links efficiently.

---

## 3. Router Architecture

```
Input Port → Switching Fabric → Output Port
     ↓              ↑               ↓
  Routing      (backplane/      Output buffer
  Processor    crossbar/bus)    (queuing)
```

### Input Port
- Physical layer termination
- Link-layer decapsulation
- **Longest prefix match** lookup in forwarding table (fast → done in TCAM hardware)
- Queuing if fabric is busy (head-of-line blocking possible)

### Switching Fabric
Three types:

| Type | Speed | Contention |
|------|-------|------------|
| **Memory-based** | Slow (shared bus to CPU) | High — packet copied through system memory |
| **Bus-based** | Moderate | One packet at a time on shared bus |
| **Crossbar (interconnection network)** | Fast | Multiple packets simultaneously if different in/out ports |

### Output Port
- Queuing when arrival rate > line rate
- **Drop policy:** Tail drop (simplest), RED (Random Early Detection — drop probabilistically before full), WRED (weighted by precedence)
- **Scheduling discipline:** FCFS, Priority Queuing, Round Robin, Weighted Fair Queuing (WFQ)

### Where Queuing Happens

- **Input queuing:** Head-of-line (HOL) blocking — packet at front blocks others behind it even if their destination ports are free. Cuts throughput to ~58% under heavy load.
- **Output queuing:** Packets wait when output link is congested. Need buffer = RTT × link capacity (rule of thumb).

---

## 4. Routing Algorithms

### Classification

| Axis | Types |
|------|-------|
| **Global vs Decentralized** | Link-state (global) vs Distance-vector (decentralized) |
| **Static vs Dynamic** | Manual config vs auto-adapting to topology/load |
| **Load-sensitive vs Load-insensitive** | OSPF uses fixed link costs; some older algorithms varied by congestion |

### 4.1 Link-State (Dijkstra's Algorithm)

Every router has complete topology map. Used by OSPF.

**Step-by-step:**
1. Each router discovers neighbors, measures link cost (by bandwidth, delay, or admin preference).
2. Floods this info as **Link State Advertisement (LSA)** to *all* routers in the network.
3. Each router runs Dijkstra to compute shortest-path tree with itself as root.
4. Forwarding table built from the tree.

**Complexity:** O(n²) naive, O(n log n + m) with priority queue (n = routers, m = links).

**Oscillation problem:** When link cost varies with load, Dijkstra can oscillate. Routers all shift traffic to lightly-loaded path → it becomes overloaded → everyone shifts back. Fix: don't route based on real-time load; use fixed administrative costs.

### 4.2 Distance-Vector (Bellman-Ford)

Each router knows only its neighbors. Used by RIP, BGP (path-vector variant).

**Core equation (Bellman-Ford):**
```
d_x(y) = min_v { c(x,v) + d_v(y) }
```
where `d_x(y)` = cost from x to y, `c(x,v)` = direct cost to neighbor v.

Routers exchange distance vectors periodically. Converges eventually.

**Count-to-infinity problem:** Bad news travels slowly. If link to destination breaks, routers increment costs slowly toward infinity. Fixes:
- **Split horizon:** Don't advertise route back to the neighbor you learned it from.
- **Split horizon with poison reverse:** Advertise it back with cost = ∞.
- **Hold-down timers:** Ignore worse-cost updates for a cooldown period.

DV convergence is slower than LS in large networks.

### 4.3 Path-Vector (BGP)

Instead of distances, advertise the full AS-path to destination. Detects loops (if you see your own AS in the path, discard). Enables **policy-based routing** — choose path based on business relationships, not just shortest cost.

### 4.4 Hierarchical Routing

Internet can't run one routing algorithm flat — too many routers. Solution:
- **Autonomous Systems (AS):** Group of routers under single administrative control.
- **Intra-AS (IGP):** RIP, OSPF, EIGRP, IS-IS — within an AS.
- **Inter-AS (EGP):** BGP — between ASes.

**Hot-potato routing:** Within an AS, send packet to the nearest exit point toward the destination, even if a farther exit point gives shorter total path. Minimizes internal resource use.

---

## 5. Key Protocols at Network Layer

### 5.1 ICMP (Internet Control Message Protocol)

Carried inside IP datagrams (protocol = 1). Used for error reporting and diagnostics, **not** for making IP reliable.

| Type | Code | Purpose |
|------|------|---------|
| 0 | 0 | Echo reply (ping response) |
| 3 | 0–15 | Destination unreachable (net, host, port, protocol, frag needed, etc.) |
| 4 | 0 | Source quench (deprecated — congestion notification) |
| 5 | 0–3 | Redirect (better next-hop exists) |
| 8 | 0 | Echo request (ping) |
| 11 | 0–1 | Time exceeded (TTL=0 — traceroute uses this) |
| 12 | 0–1 | Parameter problem (bad header field) |

**Traceroute:** Sends UDP packets with TTL = 1, 2, 3, ... Each intermediate router drops packet when TTL=0 and sends ICMP Time Exceeded back. Source maps the path by collecting these ICMP messages. Final hop sends ICMP Port Unreachable (UDP port is unlikely high).

### 5.2 ARP (Address Resolution Protocol)

Glue between network layer (IP) and link layer (MAC). Broadcast "Who has IP X?" — owner responds with its MAC address.

- ARP table cached in each host (view with `arp -a`).
- No authentication — **ARP spoofing** trivial: anyone can claim any IP↔MAC mapping.

### 5.3 DHCP (Dynamic Host Configuration Protocol)

Assigns IP addresses, subnet masks, default gateway, DNS servers dynamically.

**Four-message handshake:**
1. **DHCP Discover** — client broadcasts (0.0.0.0 → 255.255.255.255)
2. **DHCP Offer** — server(s) respond with offered IP
3. **DHCP Request** — client picks one offer, requests it
4. **DHCP ACK** — server confirms lease

Lease renewal at T/2, rebinding at 7T/8.

---

## 6. NAT (Network Address Translation)

Maps private addresses to public addresses. Masks entire internal network behind one (or few) public IPs.

```
Internal: 10.0.0.5:43210 → NAT router rewrites → Public: 203.0.113.1:50001
```

**NAT translation table** stores mapping. Entries created on outbound packet, used for inbound replies.

**Types:**
- **Static NAT:** 1:1 permanent mapping.
- **Dynamic NAT:** Pool of public IPs, mapped on demand.
- **PAT / NAPT (Port Address Translation):** Many private IPs share one public IP, differentiated by port numbers. Most common.

**Controversy:** NAT breaks end-to-end principle — hosts behind NAT are not directly reachable. Breaks protocols that embed IPs in payload (FTP, SIP). P2P, VoIP need workarounds (STUN, TURN, ICE). But it slowed IPv4 exhaustion significantly.

---

## 7. Fragmentation and Reassembly

When IP datagram exceeds link MTU, router fragments it. **Reassembly only at destination** — not at intermediate routers.

**Fragmentation fields in IP header:**
- **Identification:** Same ID for all fragments of original datagram.
- **Flags:** DF (Don't Fragment — if set and packet exceeds MTU, drop and send ICMP "fragmentation needed"), MF (More Fragments).
- **Fragment Offset:** Byte position of this fragment in original datagram (units of 8 bytes).

If any fragment lost → entire datagram discarded by destination. TCP avoids fragmentation via **Path MTU Discovery** (sets DF bit, listens for ICMP "frag needed" responses, adjusts MSS downward).

---

## 8. Tunneling

Encapsulate a packet as payload inside another packet. Key enabler for VPNs, IPv6-over-IPv4 transition.

```
[Outer IP Header | Inner IP Header | Payload]
```

- **IP-in-IP:** Protocol = 4 in outer header.
- **GRE (Generic Routing Encapsulation):** More flexible, carries any protocol.
- **IPsec tunnel mode:** Encrypted + authenticated outer tunnel.

Use cases: VPNs, connecting IPv6 islands across IPv4 internet, Mobile IP home-agent forwarding.

---

## 9. IP Multicast (Brief)

Unicast = one-to-one. Broadcast = one-to-all. Multicast = one-to-many (efficiently).

- Uses Class D addresses.
- **IGMP (Internet Group Management Protocol):** Host tells local router "I want group G." Router tracks which groups have listeners.
- **PIM (Protocol Independent Multicast):** Routers build distribution trees. Dense mode (flood-and-prune) vs Sparse mode (explicit join via Rendezvous Point).
- **Source-Specific Multicast (SSM):** Receiver specifies (source, group) — simpler, no RP needed.

---

## 10. Software-Defined Networking (SDN)

Decouple control plane from data plane.

```
Traditional: Each router runs routing protocol + forwards packets. Distributed control.
SDN: Controller computes rules centrally → pushes flow table entries to switches. Switches just match+forward.
```

### Key Ideas

- **OpenFlow:** Protocol between controller and switches. Match on header fields (MAC, IP, port, VLAN, ...) → apply actions (forward, drop, rewrite, send-to-controller).
- **Network OS (controller):** Centralized logic — ONOS, OpenDaylight, Ryu, Floodlight.
- **Benefits:** Programmable, easier traffic engineering, network-wide visibility, faster innovation (no waiting for router vendor firmware updates).

### Match-Action Abstraction

```
match(src_ip=10.0.0.0/8, dst_port=80) → action(forward: port 3)
match(dst_ip=203.0.113.5)            → action(drop)
default                               → action(send_to_controller)
```

---

## 11. Quality of Service (QoS) at Network Layer

### Traffic Classes

- **IntServ (Integrated Services):** Per-flow resource reservation using RSVP. Fine-grained, doesn't scale to internet core (too many flows).
- **DiffServ (Differentiated Services):** Coarse-grained. Packets marked with DSCP (Differentiated Services Code Point) in IP header. Routers apply **Per-Hop Behaviors (PHB):**
  - **EF (Expedited Forwarding):** Low loss, low latency, low jitter. Premium service.
  - **AF (Assured Forwarding):** Four classes × three drop precedences. Bronze/silver/gold tiers.
  - **BE (Best Effort):** Default.

### Policing vs Shaping

| | Policing | Shaping |
|---|----------|---------|
| **Action on excess** | Drop or remark (lower DSCP) | Buffer and delay |
| **Output rate** | Strictly bounded (sawtooth-like) | Smoothed |
| **Use case** | Network edge (enforce contract) | Egress link (match link rate) |

**Token bucket** is the canonical algorithm: tokens arrive at rate r, bucket capacity b. Packet needs token(s) to transmit. Burst up to b, sustained rate r.

---

## 12. Generalized Forwarding (Beyond IP)

Match not just on destination IP. Match on **any header field** across multiple layers:

```
match(
  src_mac=..., dst_mac=...,
  src_ip=..., dst_ip=...,
  src_port=..., dst_port=...,
  vlan_id=...,
  mpls_label=...,
  ...
) → action(
  forward(port),
  rewrite_src_mac(...),
  rewrite_dst_ip(...),
  push_vlan(...),
  encapsulate_mpls(...),
  ...
)
```

This is the heart of SDN and modern programmable switches (P4 language, Intel Tofino, etc.). The forwarding plane becomes a compiler target rather than fixed-function ASIC logic.

---

## 13. MPLS (Multi-Protocol Label Switching)

Lies between Layer 2 and Layer 3 ("Layer 2.5").

- Packets get a **label** at ingress router. Core routers forward by label lookup only (fast, fixed-length, exact match — no longest-prefix match).
- Label-switched paths (LSPs) pre-established. Enables **traffic engineering** — route flows along explicit paths, not just shortest-path.
- Labels can stack → nested tunnels (e.g., MPLS VPN with BGP/MPLS).

---

## Summary: Data Plane vs Control Plane

| | Data Plane | Control Plane |
|---|---|---|
| **What** | Forwarding — move packets | Routing — compute paths |
| **Where** | Hardware (TCAM, ASIC) | Software (routing daemons) or SDN controller |
| **Speed** | Per-packet (nanoseconds) | Per-event (milliseconds to seconds) |
| **Examples** | Forwarding table lookup, NAT, ACL | OSPF, BGP, RIP, OpenFlow controller |

Understanding this split is the conceptual foundation for all modern networking — from traditional routers to cloud VPCs to Kubernetes CNI plugins.
