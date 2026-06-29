# OSI Model

> Detailed notes from Practical Networking's OSI Model lessons:
> 1. `LkolbURrtTs` — practical OSI layers 1, 2, and 3
> 2. `0aGqGKrRE0g` — practical OSI layer 4, encapsulation, and device/protocol layers
> 3. `2iFFRqzX3yE` — practical OSI layers 5, 6, and 7

---

## 1. Big Picture

The goal of networking is simple: **allow two hosts to share data**.

Before networks, moving data from one computer to another might mean copying it to removable media and physically carrying it over. Networking automates that exchange, but only because devices agree on common rules.

The **OSI model** organizes those rules into seven layers. Each layer has a specific responsibility. If all layers do their jobs, data can move from an application on one host to an application on another host.

```text
User/Application data
        ↓
Layer 7 Application
Layer 6 Presentation
Layer 5 Session
Layer 4 Transport
Layer 3 Network
Layer 2 Data Link
Layer 1 Physical
        ↓
Bits on the medium
```

Important mindset: **the OSI model is a model, not a rigid law.** It is a way to understand the responsibilities involved in network communication. Real devices and protocols sometimes cross layer boundaries.

---

## 2. The Seven Layers at a Glance

| Layer | Name | Main responsibility | Common examples |
|---|---|---|---|
| 7 | Application | Defines application-level commands and network services | HTTP, FTP, DNS, SMTP, SSH |
| 6 | Presentation | Defines how data is represented, encoded, compressed, or encrypted | ASCII/UTF-8, JPEG, JSON, TLS concepts, compression |
| 5 | Session | Maintains or distinguishes user/application sessions | HTTP cookies, login sessions, RPC sessions |
| 4 | Transport | Service-to-service delivery using ports | TCP, UDP, source/destination ports |
| 3 | Network | End-to-end delivery using logical addresses | IP, routers, IPv4/IPv6 addresses |
| 2 | Data Link | Hop-to-hop delivery on a local link | Ethernet, Wi-Fi NICs, MAC addresses, switches |
| 1 | Physical | Transports raw bits | Cables, fiber, radio waves, repeaters, hubs |

A useful memory aid from bottom to top:

```text
Please Do Not Throw Sausage Pizza Away
Physical, Data Link, Network, Transport, Session, Presentation, Application
```

But memorizing names is not enough. The useful understanding is what each layer contributes to moving data.

---

## 3. Core Delivery Scopes

The lower and middle OSI layers can be understood by the scope of delivery they provide.

| Scope | Layer | Question answered |
|---|---|---|
| Bits across a medium | Layer 1 | How do 1s and 0s physically move? |
| Hop-to-hop | Layer 2 | How does data reach the next device on this local link? |
| End-to-end | Layer 3 | How does data reach the final host across networks? |
| Service-to-service | Layer 4 | Which application/process should receive this data? |
| Session/user continuity | Layer 5 | Which user/session does this data belong to? |
| Data interpretation | Layer 6 | How should these bits be interpreted? |
| Application behavior | Layer 7 | What command or network service action is being requested? |

---

## 4. Layer 1 — Physical Layer

### Purpose

Layer 1 is responsible for **transporting bits**: 1s and 0s.

It does not care about IP addresses, MAC addresses, websites, files, or applications. It only provides a way for signals representing bits to move from one device to another.

### Examples

Layer 1 technologies include:

- Copper Ethernet cables
- Fiber optic cables
- Electrical signaling
- Light pulses
- Wi-Fi radio waves
- Repeaters
- Hubs / multi-port repeaters

Do not get stuck on the word “physical.” Wi-Fi is wireless, but it is still partly a Layer 1 technology because radio waves carry bits through the air.

### Devices

| Device | Why it fits Layer 1 |
|---|---|
| Repeater | Regenerates or amplifies a signal so bits can travel farther |
| Hub | Multi-port repeater; repeats bits out ports without understanding frames |

### Key point

```text
Layer 1 = move bits across the medium
```

---

## 5. Layer 2 — Data Link Layer

### Purpose

Layer 2 is responsible for **hop-to-hop delivery**.

A **hop** is one local Layer 2 movement, such as:

```text
Host NIC -> Switch
Switch -> Router interface
Router interface -> Next router interface
Router interface -> Destination host NIC
```

Layer 2 interacts directly with Layer 1. It puts bits onto the medium and receives bits from the medium.

### Addressing: MAC addresses

Layer 2 commonly uses **MAC addresses**.

A MAC address is:

- 48 bits long
- Usually shown as 12 hexadecimal digits
- Assigned to a network interface

The same MAC may be displayed in different formats:

```text
AA-BB-CC-DD-EE-FF     Windows-style
AA:BB:CC:DD:EE:FF     Linux-style
AABB.CCDD.EEFF        Cisco-style
```

### Layer 2 devices and components

| Item | Layer 2 role |
|---|---|
| NIC | Sends and receives frames on a network medium |
| Wi-Fi adapter | Sends and receives frames over wireless Layer 1 signals |
| Switch | Forwards frames within a local network based on MAC addresses |

Switches are Layer 2 devices because they make forwarding decisions using Layer 2 information, especially destination MAC addresses.

### Data unit: frame

Layer 2 wraps the Layer 3 packet in a **frame**.

```text
[ Ethernet Header ][ IP Packet ][ Ethernet Trailer ]
  Src MAC
  Dst MAC
```

### Key point

```text
Layer 2 = hop-to-hop delivery using MAC addresses
```

The Layer 2 header is only useful for the current hop. At the next router, the old Layer 2 header is removed and a new one is created for the next hop.

---

## 6. Layer 3 — Network Layer

### Purpose

Layer 3 is responsible for **end-to-end delivery**.

It answers:

```text
Which final host should receive this data?
```

### Addressing: IP addresses

Layer 3 commonly uses IP addresses.

For IPv4:

- 32 bits long
- Written as four decimal octets
- Each octet ranges from 0 to 255

Example:

```text
192.168.1.50
```

Layer 3 addressing identifies the final source and final destination hosts across multiple networks.

### Layer 3 devices and protocols

| Item | Layer 3 role |
|---|---|
| Router | Moves packets between networks |
| Host with IP address | Participates in Layer 3 communication |
| IP protocol | Provides logical addressing and packet delivery |

### Data unit: packet

Layer 3 wraps upper-layer data in a **packet**.

```text
[ IP Header ][ Transport Segment ]
  Src IP
  Dst IP
```

### Why both IP and MAC addresses are needed

A common question:

> If we have IP addresses, why do we need MAC addresses?

Because they solve different problems.

| Address | Layer | Scope | Purpose |
|---|---|---|---|
| MAC address | Layer 2 | One local hop | Deliver frame to the next interface |
| IP address | Layer 3 | End-to-end path | Deliver packet to the final host |

Example path:

```text
Host A -> Router 1 -> Router 2 -> Host B
```

The IP addresses remain focused on the endpoints:

```text
Source IP      = Host A
Destination IP = Host B
```

But MAC addresses change at every hop:

```text
Hop 1: Host A MAC  -> Router 1 MAC
Hop 2: Router 1 MAC -> Router 2 MAC
Hop 3: Router 2 MAC -> Host B MAC
```

### ARP connects Layer 3 and Layer 2

**ARP**, the Address Resolution Protocol, maps an IP address to a MAC address on a local network.

```text
IP address -> MAC address
Layer 3    -> Layer 2
```

ARP is a good example of why the OSI model is a model rather than a perfect box system. ARP connects Layer 3 information to Layer 2 information.

### Key point

```text
Layer 3 = end-to-end delivery using IP addresses
```

---

## 7. Layer 4 — Transport Layer

### Purpose

Layer 4 is responsible for **service-to-service delivery**.

A host may run many network applications at the same time:

- Web browser
- Chat client
- Online game
- File transfer
- Video call

Layer 3 can deliver data to the correct host, but the host still needs to know which application/process should receive it. Layer 4 solves that problem.

### Addressing: ports

Layer 4 uses **port numbers**.

There are port number spaces for both TCP and UDP:

```text
TCP ports: 0-65535
UDP ports: 0-65535
```

A port identifies a service or application endpoint on a host.

### TCP vs UDP

| Protocol | Main tendency | Typical use idea |
|---|---|---|
| TCP | Reliability, ordered delivery, connection-oriented behavior | Web, SSH, file transfers |
| UDP | Efficiency, low overhead, no built-in delivery guarantee | DNS, streaming, games, voice/video |

TCP and UDP are both Layer 4 protocols because both help distinguish data streams and deliver data to the right service.

### Source and destination ports

Every transport conversation uses:

- Source IP
- Destination IP
- Source port
- Destination port
- Transport protocol, TCP or UDP

Example HTTP request:

```text
Client: 192.168.1.10:49152  ->  Web Server: 203.0.113.20:80
Protocol: TCP
```

The destination port `80` identifies HTTP on the server. The client source port `49152` is an ephemeral/random port chosen so the client knows which local application should receive the response.

Server response:

```text
Web Server: 203.0.113.20:80  ->  Client: 192.168.1.10:49152
Protocol: TCP
```

The source and destination reverse on the way back.

### Well-known examples

| Application/protocol | Common port |
|---|---|
| HTTP | TCP 80 |
| HTTPS | TCP 443 |
| DNS | UDP/TCP 53 |
| SSH | TCP 22 |
| SMTP | TCP 25 |
| DHCP | UDP 67/68 |

### Data unit: segment or datagram

Layer 4 data is often called:

- TCP **segment**
- UDP **datagram**

In many beginner explanations, “segment” is used generally for the Layer 4 unit.

### Key point

```text
Layer 4 = service-to-service delivery using ports
```

---

## 8. Layers 5, 6, and 7 — The Upper Layers

Many TCP/IP explanations combine OSI Layers 5, 6, and 7 into a single **Application** layer. That is because modern applications often implement these responsibilities in their own ways.

Still, the OSI distinctions are useful.

---

## 9. Layer 5 — Session Layer

### Purpose

Layer 5 distinguishes and maintains **sessions**.

It answers:

```text
Which user session or application conversation does this data belong to?
```

### Practical example: HTTP cookies

Suppose a user logs into a website from home Wi-Fi, then moves to a coffee shop and gets a different IP address.

If the website identified the user only by Layer 3 and Layer 4 information, the changed IP address might force the user to log in again. Instead, web applications commonly use **HTTP cookies**.

A cookie is an application-created text value stored by the client and sent back to the server. It lets the website identify the same user/session even if lower-layer details change.

```text
Old network: user has IP A, same cookie
New network: user has IP B, same cookie
Website: recognizes session using the cookie
```

### Legacy context

The OSI session layer also makes sense historically. Early computing used large mainframes accessed by multiple users through terminals. Multiple users might share the same large systems and similar lower-layer addresses, so the network/application needed a way to keep user sessions separate.

### Key point

```text
Layer 5 = distinguish user/application sessions
```

---

## 10. Layer 6 — Presentation Layer

### Purpose

Layer 6 defines **how data should be represented and interpreted**.

After data reaches the correct host, service, and session, the receiver still needs to know what the bits mean.

The same bits can be interpreted in many ways:

- ASCII or UTF-8 text
- Base64 characters
- Hexadecimal values
- 32-bit integers
- 64-bit integers
- Image/audio/video formats
- Compressed data
- Encrypted data

### Example: text encoding

An HTTP request is transmitted as bits. The receiver needs to know that groups of bits represent characters.

For example, with ASCII-style text encoding:

```text
01000111 01000101 01010100
   G        E        T
```

Layer 6 responsibility is the “how should these bits be interpreted?” part.

### Common examples

| Presentation function | Examples |
|---|---|
| Character encoding | ASCII, UTF-8 |
| Serialization | JSON, XML, protocol-specific formats |
| Compression | gzip, Brotli |
| Encryption/formatting concepts | TLS-related data representation, certificates |
| Media formats | JPEG, PNG, MP3, MP4 |

### Key point

```text
Layer 6 = data representation, encoding, compression, encryption concepts
```

---

## 11. Layer 7 — Application Layer

### Purpose

Layer 7 defines **application-level network behavior**.

It answers:

```text
What does the application want to do?
```

Layer 7 protocols define commands, request/response formats, and service behavior.

### Example: HTTP GET

If Layer 6 interprets the bits as the characters `GET`, Layer 7 defines what `GET` means.

In HTTP, `GET` is a command requesting a resource from a web server.

Example HTTP request:

```text
GET /simple.html HTTP/1.1
Host: example.com
```

Layer 7 defines that this means: fetch `/simple.html` from the named host.

### Protocol examples

| Protocol | Layer 7 purpose |
|---|---|
| HTTP/HTTPS | Web requests and responses |
| DNS | Name-to-IP lookups |
| SMTP | Sending email |
| IMAP/POP3 | Retrieving email |
| FTP | File transfer commands |
| SSH | Secure remote shell service |

### Key point

```text
Layer 7 = application commands and network services
```

---

## 12. Encapsulation

**Encapsulation** is the process of adding headers as data moves down the stack on the sending host.

Example: a browser sends an HTTP request.

### Step-by-step

1. **Layer 7-5:** Application creates data, handles commands, representation, and session logic.
2. **Layer 4:** Transport adds TCP/UDP header with source and destination ports.
3. **Layer 3:** Network adds IP header with source and destination IP addresses.
4. **Layer 2:** Data Link adds frame header/trailer with source and destination MAC addresses.
5. **Layer 1:** Physical transmits the bits.

```text
Application Data

Layer 4:
[ TCP Header ][ Application Data ]
= Segment

Layer 3:
[ IP Header ][ TCP Header ][ Application Data ]
= Packet

Layer 2:
[ Ethernet Header ][ IP Header ][ TCP Header ][ Application Data ][ Trailer ]
= Frame

Layer 1:
010101010101... bits on the medium
```

Each layer adds information needed to accomplish that layer's goal.

---

## 13. Decapsulation

**Decapsulation** is the reverse process on the receiving host.

1. **Layer 1:** Bits are received from the medium.
2. **Layer 2:** NIC checks the frame's destination MAC. If it matches, remove Layer 2 header/trailer.
3. **Layer 3:** Host checks destination IP. If it matches, remove IP header.
4. **Layer 4:** Transport checks destination port and delivers data to the correct application.
5. **Layers 5-7:** Application/session/presentation logic interprets and processes the data.

```text
Bits
 ↓
Frame: check destination MAC
 ↓
Packet: check destination IP
 ↓
Segment: check destination port
 ↓
Application data: interpret and act
```

---

## 14. Hop-by-Hop vs End-to-End Example

Topology:

```text
Host A ---- Router 1 ---- Router 2 ---- Host B
```

Host A sends data to Host B.

### Layer 3 stays end-to-end

```text
Source IP      = Host A IP
Destination IP = Host B IP
```

Usually, those IP addresses stay the same across the routed path. NAT is a common exception.

### Layer 2 changes each hop

| Hop | Source MAC | Destination MAC |
|---|---|---|
| Host A -> Router 1 | Host A MAC | Router 1 local MAC |
| Router 1 -> Router 2 | Router 1 outgoing MAC | Router 2 local MAC |
| Router 2 -> Host B | Router 2 outgoing MAC | Host B MAC |

Routers remove the old Layer 2 frame and build a new Layer 2 frame for the next hop.

---

## 15. Devices by OSI Layer

| Device/component | Typical layer | Why |
|---|---|---|
| Cable/fiber/radio | Layer 1 | Carries signals/bits |
| Repeater | Layer 1 | Regenerates signal |
| Hub | Layer 1 | Repeats bits out ports |
| NIC / Wi-Fi adapter | Layer 2 | Sends/receives frames using MAC addressing |
| Switch | Layer 2 | Forwards frames based on MAC addresses |
| Router | Layer 3 | Forwards packets based on IP/routing table |
| Firewall | Layers 3-7 depending on type | May inspect IPs, ports, sessions, or application data |
| Load balancer | Layers 4 or 7 commonly | May distribute by ports/connections or HTTP/application data |

### Important exception idea

A router is usually called a Layer 3 device, but if it uses an access control list that checks TCP/UDP ports, it is looking into Layer 4 information.

So “device at layer X” usually means:

```text
The device primarily makes decisions using information from that layer.
```

It does not always mean the device never looks at other layers.

---

## 16. Protocols by OSI Layer

| Protocol/technology | Typical layer | Notes |
|---|---|---|
| Ethernet | Layer 2 | Framing and MAC addressing on LANs |
| Wi-Fi / 802.11 | Layers 1-2 | Radio transmission plus wireless framing/MAC behavior |
| ARP | Between Layers 2 and 3 | Maps IP addresses to MAC addresses |
| IP | Layer 3 | Logical addressing and routing |
| ICMP | Layer 3-ish | Used by ping/traceroute; carried by IP |
| TCP | Layer 4 | Reliable transport with ports |
| UDP | Layer 4 | Lightweight transport with ports |
| HTTP | Layer 7 | Web application protocol |
| DNS | Layer 7 | Name resolution service |
| FTP | Layer 7 | File transfer commands/data behavior |

Again, do not force every protocol perfectly into one box. The model is for understanding.

---

## 17. OSI Model vs TCP/IP Model

The TCP/IP model is another way to describe networking. It is often more practical for the internet.

| OSI model | TCP/IP model idea |
|---|---|
| Layers 7, 6, 5 | Application |
| Layer 4 | Transport |
| Layer 3 | Internet |
| Layers 2, 1 | Network Access / Link |

TCP/IP combines OSI Layers 5-7 because applications implement sessions, presentation, and commands differently.

Example:

- HTTP may use cookies for sessions.
- FTP may not preserve a user session independently if the user's IP changes.
- HTTP and FTP may both use text encoding, but they define different application commands.

The combined TCP/IP Application layer reflects that these upper-layer behaviors are often application-specific.

---

## 18. Practical Troubleshooting with OSI

The OSI model is useful for structured troubleshooting. Start at the likely layer and work logically.

### Layer 1 checks

Symptoms:

- Link light off
- Interface down
- No signal
- Bad cable or weak wireless signal

Checks:

```bash
ip link
ethtool <interface>
ping <local-gateway>
```

Ask:

- Is the cable connected?
- Is Wi-Fi connected?
- Is the interface enabled?
- Is the physical port working?

### Layer 2 checks

Symptoms:

- Cannot reach local gateway
- ARP fails
- Wrong VLAN
- Switch forwarding issue

Checks:

```bash
ip neigh
arp -a
```

Ask:

- Can the host learn the gateway MAC address?
- Is the device in the correct VLAN/broadcast domain?
- Is there a MAC address conflict or switching issue?

### Layer 3 checks

Symptoms:

- Local network works, remote networks fail
- Incorrect IP/subnet/gateway
- Routing failure

Checks:

```bash
ip addr
ip route
ping <gateway-ip>
ping <remote-ip>
traceroute <remote-ip>
```

Ask:

- Is the IP address correct?
- Is the subnet mask/prefix correct?
- Is the default gateway configured?
- Do routers have routes both directions?

### Layer 4 checks

Symptoms:

- Host is reachable, but service is not
- Port blocked
- TCP connection refused or timed out

Checks:

```bash
ss -tulpen
nc -vz <host> <port>
curl -v http://<host>
```

Ask:

- Is the service listening on the expected port?
- Is TCP or UDP being used?
- Is a firewall blocking the port?

### Layers 5-7 checks

Symptoms:

- Login/session problems
- TLS/certificate problems
- Encoding/format issues
- Application errors
- HTTP 4xx/5xx responses

Checks:

```bash
curl -v https://example.com
openssl s_client -connect example.com:443
```

Ask:

- Is the user session/cookie valid?
- Is the certificate valid?
- Does the application understand the request format?
- Is the correct hostname/path/API command being used?

---

## 19. Common Misunderstandings

### Mistake 1 — Treating OSI as only a memorization chart

The value is not just remembering seven names. The value is understanding what each layer contributes to network communication.

### Mistake 2 — Thinking MAC addresses deliver data across the internet

MAC addresses are hop-to-hop. They are rewritten at each routed hop.

### Mistake 3 — Thinking IP addresses identify applications

IP addresses identify hosts/interfaces at Layer 3. Ports identify services/applications at Layer 4.

### Mistake 4 — Thinking switches and routers do the same job

Switches forward frames within a local network. Routers forward packets between networks.

### Mistake 5 — Forcing every protocol into exactly one layer

Some protocols span responsibilities. ARP links IP and MAC. TLS often gets discussed around presentation/security but interacts with applications and transport behavior. Real networking is messier than the model.

### Mistake 6 — Ignoring Layers 5-7

Many practical explanations combine them, but the responsibilities still exist:

- Session: keep conversations/users distinct
- Presentation: interpret/format data
- Application: define commands and service behavior

---

## 20. Mini End-to-End Example: Opening a Website

A laptop opens `https://example.com`.

1. **Layer 7:** Browser wants a web resource using HTTP semantics.
2. **Layer 6:** Data is represented/encoded; encryption-related formatting may apply for HTTPS.
3. **Layer 5:** Browser/server may use cookies or tokens to maintain a user session.
4. **Layer 4:** TCP connection uses a client ephemeral source port and destination port `443`.
5. **Layer 3:** IP packet uses laptop source IP and server destination IP.
6. **Layer 2:** Ethernet/Wi-Fi frame uses laptop MAC to gateway MAC for the first hop.
7. **Layer 1:** Bits are transmitted as electrical, light, or radio signals.

At each router:

- The router reads Layer 3 destination IP to choose the next hop.
- The router removes the old Layer 2 header.
- The router creates a new Layer 2 header for the next hop.
- The Layer 3 destination remains the web server IP unless NAT or similar translation occurs.

At the server:

- Layer 2 confirms the frame is for the server NIC.
- Layer 3 confirms the IP packet is for the server IP.
- Layer 4 delivers the data to the service listening on TCP 443.
- Upper layers interpret the request and generate an application response.

---

## 21. Exam / Interview Points

- The OSI model has seven layers: Physical, Data Link, Network, Transport, Session, Presentation, Application.
- Layer 1 moves bits across a medium.
- Layer 2 provides hop-to-hop delivery using MAC addresses and frames.
- Layer 3 provides end-to-end delivery using IP addresses and packets.
- Layer 4 provides service-to-service delivery using ports; TCP and UDP live here.
- Layer 5 handles session identification/maintenance.
- Layer 6 handles data representation, encoding, compression, and encryption concepts.
- Layer 7 handles application commands and network services.
- Switches are typically Layer 2 devices; routers are typically Layer 3 devices.
- MAC addresses change hop-by-hop; IP addresses are end-to-end in normal routing.
- Encapsulation adds headers as data moves down the stack; decapsulation removes them as data moves up the stack.
- ARP maps Layer 3 IP addresses to Layer 2 MAC addresses on a local network.
- Devices and protocols can be exceptions; the OSI model is a teaching and troubleshooting abstraction.
- The TCP/IP model commonly combines OSI Layers 5-7 into one Application layer.

---

## 22. One-Line Summary

The OSI model explains network communication as layered responsibilities: bits move at Layer 1, frames move hop-to-hop at Layer 2, packets move end-to-end at Layer 3, ports deliver data service-to-service at Layer 4, and Layers 5-7 manage sessions, data representation, and application commands so two hosts can meaningfully exchange data.
