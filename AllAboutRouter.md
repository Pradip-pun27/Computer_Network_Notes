# All About Routers

> Detailed notes from Practical Networking's three-part router lesson:
> 1. `AzXys5kxpAM` — what routers are, router interfaces, routing tables, route sources
> 2. `Ep-x_6kggKA` — ARP tables, hop-by-hop forwarding through routers
> 3. `zmxLg4jV0ts` — router hierarchy, scaling, route summarization

---

## 1. Big Picture

A **router** is a network device whose main job is **routing**: moving packets between networks.

A host usually sends/receives packets for itself. A router forwards packets that are **not addressed to the router itself**. That forwarding behavior is the core difference between a normal host and a router.

```text
Switching = moving frames within a network
Routing   = moving packets between networks
```

Routers make the internet possible because the internet is not one giant flat LAN. It is many networks connected together.

---

## 2. Router vs Host

| Device | Main purpose | Forwards packets not addressed to itself? |
|---|---|---|
| Host | Uses the network to send/receive its own data | No |
| Router | Connects networks and forwards traffic between them | Yes |

If a normal host receives a packet whose destination IP is not its own IP, it normally drops it. If a router receives such a packet, it checks its routing table and tries to forward it.

---

## 3. Routers Connect Networks

A router is connected to multiple networks. For each network it connects to, it has an interface.

Each router interface usually has:

- An IP address in that connected network
- A MAC address for Layer 2 communication on that network

Example:

```text
Network A: 10.0.4.0/24     Router R1 interface: 10.0.4.1, MAC R1-A
Network B: 10.0.5.0/24     Router R1 interface: 10.0.5.1, MAC R1-B
```

Important: a router does not have just “one IP address.” It can have one IP address per interface/network.

---

## 4. Router Interfaces

A router interface is the router's connection point into a network.

```text
Host A ---- Network 10.0.4.0/24 ---- [R1 interface]
                                      [R1 interface] ---- Network 10.0.5.0/24 ---- Host B
```

The interface is what makes the router a member of that network. If a router has an interface in `10.0.4.0/24`, then the router is directly connected to that network.

---

## 5. Routing Table

A **routing table** is the router's map of networks.

Each route is an instruction like:

```text
To reach network X, send the packet out interface Y or to next-hop router Z.
```

Example routing table idea:

| Destination network | Direction / next hop |
|---|---|
| 10.0.4.0/24 | directly connected, left interface |
| 10.0.5.0/24 | directly connected, right interface |
| 10.0.6.0/24 | via Router 2 |

If a router receives a packet and has no matching route, it drops the packet.

---

## 6. Routes Can Enter the Routing Table in 3 Ways

The video explains that routes can be populated in three broad ways.

### 1. Directly connected routes

When an interface is configured with an IP address and is up, the router knows about that connected network.

Example:

```text
R1 interface IP: 10.0.4.1/24
Router learns:  10.0.4.0/24 is directly connected
```

### 2. Static routes

An administrator manually configures a route.

Example idea:

```text
To reach 10.0.6.0/24, send traffic to next-hop 10.0.5.2.
```

Static routes are simple and predictable, but they do not scale well for huge networks.

### 3. Dynamic routes

Routers learn routes automatically through routing protocols.

Examples of routing protocols:

- RIP
- OSPF
- EIGRP
- IS-IS
- BGP

Dynamic routing is useful when networks are large or change often.

---

## 7. Routing Table vs ARP Table

Routers use both routing tables and ARP tables, but they solve different problems.

| Table | Layer | Purpose | Populated how? |
|---|---|---|---|
| Routing table | Layer 3 | Which network/next hop should receive the packet? | Connected, static, dynamic routes |
| ARP table | Layer 2/3 link | What MAC address belongs to this local IP? | Dynamically by ARP |

Routing table tells the router **where to send next**.

ARP table tells the router **what MAC address to use on the outgoing local network**.

---

## 8. ARP Tables on Routers

Because router interfaces have IP addresses, routers also use ARP.

A router keeps ARP mappings for nodes on directly connected networks.

Example:

```text
R1 connected to Network A:
Host A IP -> Host A MAC

R1 connected to Network B:
Host B IP -> Host B MAC
R2 IP     -> R2 MAC
```

ARP tables usually start empty and are populated dynamically when communication requires a MAC address.

---

## 9. What Happens When a Packet Reaches a Router?

When a router receives an Ethernet frame, it processes it roughly like this:

1. Check the Layer 2 frame.
2. If destination MAC matches the router interface, accept the frame.
3. Remove the Layer 2 Ethernet header/trailer.
4. Look at the IP packet destination address.
5. Search the routing table for the best matching route.
6. Choose the outgoing interface and/or next-hop IP.
7. Use ARP if it does not know the next-hop MAC.
8. Build a new Ethernet frame for the next hop.
9. Forward the packet.

The router does not keep the original Ethernet header. It creates a new one for the next link.

---

## 10. Hop-by-Hop Forwarding

Imagine this topology:

```text
Host A ---- R1 ---- R2 ---- Host C
```

Host A wants to send data to Host C.

### On Host A

```text
IP packet:
Source IP      = Host A
Destination IP = Host C

Ethernet frame:
Source MAC      = Host A
Destination MAC = R1 local interface
```

Host A sends to its default gateway R1.

### On Router R1

R1 receives the frame, removes the Ethernet header, checks the IP destination, and consults its routing table.

If the route says Host C's network is reachable through R2, then R1 forwards toward R2.

```text
IP packet remains:
Source IP      = Host A
Destination IP = Host C

New Ethernet frame:
Source MAC      = R1 outgoing interface
Destination MAC = R2 interface
```

### On Router R2

R2 receives the frame, checks the destination IP, sees the destination network is directly connected, and forwards to Host C.

```text
IP packet remains:
Source IP      = Host A
Destination IP = Host C

New Ethernet frame:
Source MAC      = R2 outgoing interface
Destination MAC = Host C
```

### Core rule

```text
IP source/destination stay end-to-end.
MAC source/destination change every hop.
```

---

## 11. If the Router Does Not Know the Route

If a router receives a packet and no matching route exists, the router drops the packet.

This is different from ARP behavior:

- Missing ARP entry: router can ARP to discover local MAC.
- Missing route: router does not know where to send the packet, so it drops it.

Routing tables must have usable routes before packets can be forwarded successfully.

---

## 12. Why Routers Need MAC Addresses Too

Routers work at Layer 3, but to send data on Ethernet networks they still need Layer 2.

A router forwarding an IP packet onto an Ethernet link must build an Ethernet frame:

```text
Ethernet Header + IP Packet
```

So the router needs:

- Source MAC = router's outgoing interface MAC
- Destination MAC = next-hop router MAC or final host MAC

That destination MAC is learned using ARP on the outgoing local network.

---

## 13. Router Hierarchy

Routers are not usually deployed as one long line forever. Large networks use hierarchy.

Simple hierarchy example:

```text
                 Core / Aggregation Router
                  /          |           \
          Dept Router   Dept Router   Dept Router
             |             |             |
          Users A       Users B       Users C
```

A hierarchy helps because adding a new network does not require every router to connect directly to every other router.

---

## 14. Why Hierarchy Scales Better

Without hierarchy, networks can become messy and hard to manage. Every expansion may require many new links and many route updates.

With hierarchy:

- New department/site networks connect to a local router.
- Local routers connect upward to aggregation/core routers.
- Other parts of the network can often keep simpler routing information.

This gives:

1. Easier expansion
2. Cleaner topology
3. More consistent connectivity
4. Better route summarization

---

## 15. Route Summarization

**Route summarization** means representing multiple smaller networks with one larger route.

Example individual routes:

```text
10.0.4.0/24
10.0.5.0/24
10.0.6.0/24
```

If designed properly, these may be summarized into a larger prefix such as:

```text
10.0.4.0/22
```

The exact summary depends on binary boundaries, but the concept is: one route can stand in for several more-specific routes.

### Why summarization is useful

- Smaller routing tables
- Less memory/CPU usage on routers
- Fewer route advertisements
- Cleaner troubleshooting
- Better scaling for large organizations and the internet

---

## 16. Understanding `/24`

The videos briefly explain that an IPv4 address is 32 bits.

A `/24` prefix means the first 24 bits identify the network portion.

IPv4 example:

```text
10.0.4.9
```

IPv4 has four octets:

```text
10 . 0 . 4 . 9
8b   8b  8b  8b  = 32 bits total
```

`/24` means the first three octets are the network portion:

```text
Network part: 10.0.4
Host part:             .9
```

So `10.0.4.9/24` belongs to network `10.0.4.0/24`.

---

## 17. Forwarding Decision Example

Router receives packet:

```text
Destination IP: 10.0.6.55
```

Routing table:

```text
10.0.4.0/24 -> left interface
10.0.5.0/24 -> right interface
10.0.6.0/24 -> next-hop R2
```

Router selects:

```text
10.0.6.0/24 -> next-hop R2
```

Then it checks ARP for R2's MAC on the outgoing network. If missing, it ARPs, learns the MAC, creates a new frame, and forwards.

---

## 18. End-to-End vs Hop-to-Hop

This is one of the most important router concepts.

| Information | Scope | Changes at each router? |
|---|---|---|
| Source IP | End-to-end | Usually no |
| Destination IP | End-to-end | Usually no |
| Source MAC | Hop-to-hop | Yes |
| Destination MAC | Hop-to-hop | Yes |
| Frame | One local link | Rebuilt each hop |
| Packet | Across routed path | Forwarded through routers |

Note: NAT can change IP addresses, but basic routing does not require changing source/destination IP.

---

## 19. Router Troubleshooting Checklist

When routing fails, check:

1. Are router interfaces up?
2. Does each interface have the correct IP/prefix?
3. Are directly connected routes present?
4. Are static/dynamic routes present for remote networks?
5. Is the next hop reachable?
6. Does ARP resolve next-hop MAC addresses?
7. Are packets being dropped because of missing routes?
8. Are firewall/ACL rules blocking traffic?
9. Is there a return route back to the source network?

Useful commands on many systems/vendors:

```bash
show ip route
show arp
show interfaces
ping <next-hop>
traceroute <destination>
ip route
ip neigh
```

---

## 20. Common Mistakes

### Mistake 1 — Thinking routers only have one IP

Routers usually have one IP per interface/network.

### Mistake 2 — Thinking routers forward frames unchanged

Routers strip the old Layer 2 frame and create a new one for the next hop.

### Mistake 3 — Thinking ARP table and routing table are the same

Routing table chooses the path/network. ARP table resolves local IP to MAC.

### Mistake 4 — Forgetting return routes

If Host A can send to Host C but Host C's side has no route back to Host A, communication still fails.

### Mistake 5 — Assuming missing ARP and missing route are handled the same way

Missing ARP can be discovered dynamically. Missing route usually means packet drop.

---

## 21. Exam / Interview Points

- A router forwards packets not addressed to itself.
- Routers connect different IP networks.
- Each router interface belongs to a network and has its own IP/MAC.
- Routing tables contain routes to destination networks.
- Routes can be directly connected, static, or dynamically learned.
- ARP tables map local IP addresses to MAC addresses.
- A router rebuilds Layer 2 headers at every hop.
- Destination IP identifies the final destination; destination MAC identifies the next hop.
- Hierarchical router design improves scalability.
- Route summarization reduces routing table size and route complexity.

---

## 22. One-Line Summary

A router connects networks by reading each packet's destination IP, choosing the best route from its routing table, resolving the next-hop MAC with ARP when needed, rebuilding the Layer 2 frame, and forwarding the packet hop by hop through a scalable network hierarchy.
