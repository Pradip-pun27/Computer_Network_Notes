# Everything a Host Does to Speak on the Internet

> Detailed notes from Practical Networking's two-part lesson on what hosts do to communicate:
> 1. `gYN2qN11-wE` — same-network host communication
> 2. `JI9Zm2tbUoE` — foreign-network communication through a router

---

## 1. Big Picture

A **host** is an end device with an IP address: laptop, phone, server, printer, VM, etc. When a host wants to send data, it does not magically put “internet data” on the wire. It follows a layered process:

1. The application produces data.
2. Layer 3 adds IP addressing so the packet has a source and destination IP.
3. Layer 2 adds MAC addressing so the frame can reach the **next hop** on the local link.
4. If the destination MAC is unknown, the host uses **ARP** to discover it.
5. The frame is sent to the NIC and transmitted.

The key lesson: **IP addresses identify the final source/destination, but MAC addresses move the data across the current local network segment.**

---

## 2. Required Terms

| Term | Meaning | Why it matters |
|---|---|---|
| Host | Any end device that sends/receives data but does not forward packets for others | Example: Host A, Host B, Host C |
| NIC | Network Interface Card | Physical/logical interface connected to the network |
| MAC address | Layer 2 address burned/assigned to an interface | Used for local delivery on the same network/link |
| IP address | Layer 3 logical address | Used to identify source and destination hosts across networks |
| Subnet mask / prefix | Defines network size, e.g. `/24` | Lets host decide if destination is local or foreign |
| Packet | Layer 3 unit with IP header | Contains source IP and destination IP |
| Frame | Layer 2 unit with Ethernet header | Contains source MAC and destination MAC |
| ARP | Address Resolution Protocol | Resolves IP address → MAC address on the local network |
| ARP cache/table | Stored IP-to-MAC mappings | Avoids repeating ARP for every packet |
| Default gateway | Router IP configured on a host | Used when destination is on a foreign network |
| Same network | Source and destination are in the same IP network | Host ARPs directly for destination host |
| Foreign network | Destination is outside local network | Host ARPs for default gateway, not final host |

---

## 3. The Two Core Scenarios

The videos explain host communication using two situations.

### Scenario A — Host talks to another host on the same network

Example:

```text
Host A ---------------- Host B
10.1.1.11/24            10.1.1.33/24
MAC: Aaaa               MAC: Bbbb
```

There may be a cable, hub, one switch, or multiple switches between them. From the host's point of view, the same basic process happens: if the destination is in the same IP network, the host sends directly to that destination's MAC address.

### Scenario B — Host talks to another host on a foreign network

Example:

```text
Host A ---- Router ---- Host C
10.1.1.11/24            10.2.2.22/24
GW: 10.1.1.1
```

Host A cannot directly deliver a Layer 2 frame to Host C because Host C is not on Host A's local network. So Host A sends the frame to the **router/default gateway**. The IP packet is still addressed to Host C, but the Ethernet frame is addressed to the router.

---

## 4. Same-Network Communication: Step-by-Step

Assume Host A wants to send data to Host B on the same network.

### Step 1 — Host A has data to send

The data may come from an application: ping, browser, SSH, file transfer, etc.

Host A somehow knows Host B's IP address. Examples:

- User typed an IP address into `ping`.
- DNS resolved a name to an IP.
- An application already has the peer IP.

Important: the user/application usually gives an **IP address**, not a MAC address.

### Step 2 — Host A builds the Layer 3 packet

Host A creates an IP packet:

```text
IP Packet
Source IP:      Host A IP
Destination IP: Host B IP
Payload:        Application/transport data
```

The source and destination IPs identify the endpoints.

### Step 3 — Host A decides: same network or foreign network?

Host A compares:

- Its own IP address
- Its subnet mask/prefix
- The destination IP address

If destination IP belongs to the same local network, Host A knows it can send directly to Host B at Layer 2.

### Step 4 — Host A needs Host B's MAC address

Layer 3 has the destination IP, but Ethernet delivery requires a destination MAC.

Host A checks its ARP cache:

```text
ARP Cache
IP Address       MAC Address
10.1.1.33        ?
```

If Host B's MAC is not known, Host A must use ARP.

### Step 5 — Host A sends an ARP Request

ARP Request means:

```text
Who has 10.1.1.33? Tell 10.1.1.11.
```

At Layer 2, this request is sent as a broadcast:

```text
Destination MAC: FF:FF:FF:FF:FF:FF
Source MAC:      Host A MAC
```

Why broadcast? Because Host A does not yet know which MAC owns the target IP.

### Step 6 — Host B replies with an ARP Reply

Host B sees the ARP Request and recognizes its own IP address. It replies:

```text
10.1.1.33 is at Host-B-MAC.
```

This ARP Reply is normally unicast back to Host A.

### Step 7 — Host A updates its ARP cache

Host A stores the mapping:

```text
10.1.1.33 -> Host-B-MAC
```

Future packets to Host B can skip ARP until the cache entry expires.

### Step 8 — Host A builds the Layer 2 frame

Now Host A can wrap the IP packet inside an Ethernet frame:

```text
Ethernet Frame
Source MAC:      Host A MAC
Destination MAC: Host B MAC
Payload:         IP packet
```

Inside the payload:

```text
IP Packet
Source IP:       Host A IP
Destination IP:  Host B IP
```

### Step 9 — The frame is transmitted

The frame goes out Host A's NIC. Switches, if present, forward the frame inside the local network. Host B receives it because the destination MAC matches Host B.

---

## 5. Same-Network Key Point

For same-network communication:

```text
Destination IP  = final host IP
Destination MAC = final host MAC
```

The host ARPs for the **destination host's IP**.

---

## 6. Foreign-Network Communication: Step-by-Step

Now assume Host A wants to send data to Host C on another network.

```text
Host A: 10.1.1.11/24
Router local interface: 10.1.1.1/24
Host C: 10.2.2.22/24
```

### Step 1 — Host A has data for Host C

Host A knows Host C's IP address through user input, DNS, or the application.

### Step 2 — Host A builds the Layer 3 packet

The IP packet is addressed end-to-end:

```text
IP Packet
Source IP:       Host A IP
Destination IP:  Host C IP
```

The destination IP is still Host C, not the router.

### Step 3 — Host A determines Host C is foreign

Host A compares Host C's IP to its own network using the subnet mask. If Host C is outside Host A's local network, Host A cannot ARP directly for Host C.

### Step 4 — Host A uses the default gateway

A host is configured with three important network settings:

1. IP address
2. Subnet mask/prefix
3. Default gateway

The **default gateway** is the router IP on the host's local network. It is the next hop for traffic going outside the local network.

You can often view it with:

```bash
ipconfig        # Windows
ip route        # Linux
netstat -rn     # Unix-like systems
```

### Step 5 — Host A needs the router's MAC address

Because Host C is foreign, Host A does not need Host C's MAC. It needs the MAC of the router's local interface.

Host A checks ARP cache:

```text
10.1.1.1 -> ?
```

If missing, Host A uses ARP for the default gateway IP.

### Step 6 — Host A ARPs for the default gateway

ARP Request:

```text
Who has 10.1.1.1? Tell 10.1.1.11.
```

Router replies:

```text
10.1.1.1 is at Router-MAC.
```

Host A stores:

```text
10.1.1.1 -> Router-MAC
```

### Step 7 — Host A builds the frame to the router

This is the most important part:

```text
Ethernet Frame
Source MAC:      Host A MAC
Destination MAC: Router MAC
Payload:         IP packet

IP Packet inside
Source IP:       Host A IP
Destination IP:  Host C IP
```

Layer 2 destination is the next hop. Layer 3 destination is the final target.

### Step 8 — Router receives and forwards

Host A's responsibility ends after handing the packet to the router. The router then checks its routing table and forwards the packet toward Host C.

---

## 7. Foreign-Network Key Point

For foreign-network communication:

```text
Destination IP  = final host IP
Destination MAC = default gateway/router MAC
```

The host ARPs for the **default gateway's IP**, not the final destination IP.

---

## 8. Same Network vs Foreign Network

| Question | Same Network | Foreign Network |
|---|---|---|
| Who is final IP destination? | Destination host | Destination host |
| Who receives first Layer 2 frame? | Destination host | Default gateway/router |
| What IP does host ARP for? | Destination host IP | Default gateway IP |
| Destination MAC in first frame | Destination host MAC | Router MAC |
| Is router required? | No | Yes |
| Does IP packet destination change? | No | No |

---

## 9. Layer Encapsulation View

### Same network

```text
Host A creates:

[ Ethernet Header ] [ IP Header ] [ Data ]
Dst MAC = Host B     Dst IP = Host B
Src MAC = Host A     Src IP = Host A
```

### Foreign network

```text
Host A creates:

[ Ethernet Header ] [ IP Header ] [ Data ]
Dst MAC = Router     Dst IP = Host C
Src MAC = Host A     Src IP = Host A
```

The MAC changes hop-by-hop. The IP source/destination remain end-to-end, unless NAT is involved.

---

## 10. Why ARP Exists

Users and applications work with IP addresses. Ethernet needs MAC addresses. ARP bridges this gap.

ARP's purpose:

```text
Resolve Layer 3 address -> Layer 2 address
Resolve IP address      -> MAC address
```

ARP only works for devices on the local network/broadcast domain. That is why a host ARPs for the router when the final destination is remote.

---

## 11. ARP Cache Behavior

ARP entries are cached so the device does not broadcast for every packet.

Typical flow:

1. ARP cache is empty.
2. First packet requires ARP.
3. ARP reply is received.
4. Mapping is stored.
5. Future traffic uses cached MAC.
6. Entry expires later and may be refreshed.

Useful commands:

```bash
arp -a              # Windows/Linux legacy command
ip neigh            # Linux neighbor/ARP table
ping <ip>           # Often triggers ARP if entry is missing
```

---

## 12. Common Misunderstandings

### Mistake 1 — Thinking hosts always ARP for the final destination

False. Hosts ARP only on the local network.

- Same network: ARP for final host.
- Foreign network: ARP for default gateway.

### Mistake 2 — Thinking the router's IP replaces the final destination IP

False. The packet's destination IP remains the final host. The router is only the next Layer 2 hop.

### Mistake 3 — Confusing packet and frame

- Packet = Layer 3 IP unit.
- Frame = Layer 2 Ethernet unit.

The packet is carried inside the frame.

### Mistake 4 — Thinking switches behave like routers

Switches move frames inside a network. Routers move packets between networks.

---

## 13. Practical Troubleshooting Logic

When a host cannot reach another host, ask:

1. Does the host have the correct IP address?
2. Does it have the correct subnet mask?
3. Is the destination same-network or foreign?
4. If foreign, is the default gateway configured?
5. Can the host ARP for the local target/default gateway?
6. Is the ARP cache showing the expected MAC?
7. Can the host ping the gateway?
8. Can the router forward to the destination network?

Useful commands:

```bash
ip addr
ip route
ip neigh
ping <gateway-ip>
ping <destination-ip>
traceroute <destination-ip>   # Linux/macOS
tracert <destination-ip>      # Windows
```

---

## 14. Mini Example

Host A:

```text
IP: 192.168.1.10/24
Gateway: 192.168.1.1
MAC: AA-AA
```

Host B:

```text
IP: 192.168.1.20/24
MAC: BB-BB
```

Host C:

```text
IP: 10.10.10.20/24
MAC: CC-CC
```

Router local interface:

```text
IP: 192.168.1.1/24
MAC: RR-RR
```

### A sends to B

```text
Dst IP  = 192.168.1.20
Dst MAC = BB-BB
ARP for = 192.168.1.20
```

### A sends to C

```text
Dst IP  = 10.10.10.20
Dst MAC = RR-RR
ARP for = 192.168.1.1
```

---

## 15. Exam / Interview Points

- A host must know IP address, subnet mask, and default gateway to communicate properly.
- Subnet mask helps the host decide whether the destination is local or remote.
- ARP resolves IP-to-MAC only within the local network.
- For same-network traffic, the first frame goes directly to the destination host.
- For remote traffic, the first frame goes to the default gateway.
- The Layer 2 header is rewritten at every hop; the Layer 3 packet remains addressed to the final destination.
- If a host has no default gateway, it can usually communicate only with local-network devices.

---

## 16. One-Line Summary

A host sends data by creating an IP packet for the final destination, deciding whether that destination is local or remote, resolving the correct next-hop MAC with ARP, wrapping the packet in an Ethernet frame, and transmitting it to either the destination host or the default gateway.
