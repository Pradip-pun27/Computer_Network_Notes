# Docker Networking

> Detailed practical notes from a Docker networking walkthrough (`OU6xOM0SE4o`) covering default bridge behavior, published ports, container IPs, Docker DNS, custom networks, routing between networks, and troubleshooting commands.

---

## 1. Big Picture

Docker containers are isolated processes, but each container can also have its own network namespace. That means a container can have:

- Its own network interfaces
- Its own IP addresses
- Its own routing table
- Its own DNS resolver configuration
- Membership in one or more Docker networks

The main idea from the walkthrough is:

```text
Container networking is still normal networking:
IP addresses identify endpoints.
Subnets decide local vs remote.
Gateways move traffic out of a network.
DNS maps names to IP addresses.
Routes decide where packets go next.
```

Docker adds automation around these ideas, but it does not remove them.

---

## 2. Required Terms

| Term | Meaning | Why it matters |
|---|---|---|
| Image | Template used to create containers | Example: `httpd`, `nginx`, custom debug image |
| Container | Running instance of an image | Has its own process and network namespace |
| Docker host | Machine/VM where Docker Engine runs | On Linux this is usually the host; on Docker Desktop it is often a hidden VM |
| Docker network | Virtual network managed by Docker | Containers attach to networks to communicate |
| Bridge network | Docker's common virtual L2-style network type | Default driver for many local containers |
| Default `bridge` | Built-in Docker network named `bridge` | Containers join it unless told otherwise |
| Custom bridge network | User-created Docker bridge network | Provides better isolation and automatic DNS names |
| Published port | Host port forwarded to a container port | Allows host/outside clients to reach container services |
| Gateway | Router address for a Docker network | Usually `.1` in the network's subnet |
| Docker DNS | Docker-provided name resolution for containers | Lets containers use service/container names on custom networks |
| `--internal` network | Network without external connectivity | Useful when containers should not reach outside networks |
| `NET_ADMIN` capability | Linux capability for network administration | Needed to add routes inside a container |

---

## 3. Port Publishing: Host Port vs Container Port

A web server inside a container may listen on port `80`, but that does not automatically make it reachable from the host or the internet.

Example:

```bash
docker run -p 80:80 -d httpd
```

Meaning:

```text
Host port 80  --->  Container port 80
```

If someone connects to port `80` on the Docker host, Docker forwards that connection to port `80` inside the container.

Important distinction:

| Concept | Example | Meaning |
|---|---|---|
| Container port | `80` inside `httpd` | The application listens here inside the container |
| Host port | `80` on the host | The host accepts traffic here |
| Published mapping | `-p 80:80` | Forward host `80` to container `80` |

If no port is published, the service can still exist inside the container network, but the host cannot simply access it through `localhost:80`.

Useful examples:

```bash
# Host port 8080 forwards to container port 80
docker run --name web -p 8080:80 -d httpd

# Test from the host
curl http://localhost:8080
```

---

## 4. What Happens by Default?

When you run a container without specifying a network, Docker usually attaches it to the default network named `bridge`.

```bash
docker run --name s1 -d httpd
```

Inspect the container:

```bash
docker inspect s1
```

Or inspect the network:

```bash
docker network inspect bridge
```

Typical default bridge layout:

```text
Docker bridge network: 172.17.0.0/16
Gateway:               172.17.0.1
Container s1:           172.17.0.2
Container s2:           172.17.0.3
```

The exact subnet and IPs can differ by system.

---

## 5. Host IPs, Container IPs, and Docker Desktop

A common confusion is whether the host can directly access a container's internal IP address.

Example container IP:

```text
172.17.0.2
```

On native Linux Docker, the host often can reach container bridge IPs directly because Docker networking is integrated into the Linux host.

On Docker Desktop for macOS/Windows, Docker usually runs inside a hidden Linux VM. In that case:

```text
Your laptop OS  --->  Docker Desktop VM  --->  Container bridge network
```

So from macOS/Windows, curling the container IP directly may fail:

```bash
curl http://172.17.0.2   # may fail on Docker Desktop
```

But publishing a port works because Docker Desktop sets up forwarding:

```bash
docker run -p 8080:80 -d httpd
curl http://localhost:8080
```

Key rule:

```text
Published ports are for host/outside-to-container access.
Container IPs are mainly for container-network access.
```

---

## 6. Containers Can Usually Reach the Internet

A container on a normal bridge network usually has outbound internet access.

Inside the container, traffic to the internet follows the usual routing logic:

1. Container checks whether destination IP is in its local subnet.
2. If not local, it sends traffic to its default gateway.
3. Docker's bridge/gateway forwards/NATs traffic through the Docker host.
4. The host sends traffic to the real network/router/ISP.

Example commands inside a debug-capable container:

```bash
hostname -I
ip route
ping google.com
nslookup google.com
traceroute google.com
```

This is the same basic idea as any host using a default gateway.

---

## 7. Building a Debug-Friendly Image

Minimal images often do not include tools like `ping`, `traceroute`, `ip`, `curl`, or `nslookup`. For learning and troubleshooting, it is useful to build a temporary debug image.

Example `Dockerfile`:

```Dockerfile
FROM httpd

RUN apt-get update
RUN apt-get install -y \
    iputils-ping \
    traceroute \
    iproute2 \
    curl \
    telnet \
    dnsutils \
    vim
```

Build it:

```bash
docker build -t nhttpd .
```

Then run containers from it:

```bash
docker run --name s1 -d nhttpd
docker run --name s2 -d nhttpd
```

Image vs container:

```text
Image     = template/class
Container = running instance/object
```

One image can create many containers.

---

## 8. Container-to-Container Communication on the Default Bridge

Start two containers:

```bash
docker run --name s1 -d nhttpd
docker run --name s2 -d nhttpd
```

Inspect the default bridge:

```bash
docker network inspect bridge
```

You may see something like:

```text
s1 -> 172.17.0.2
s2 -> 172.17.0.3
```

From `s1`, you can usually reach `s2` by IP:

```bash
docker exec -it s1 bash
curl http://172.17.0.3
ping 172.17.0.3
```

But on the default `bridge`, container names are not reliably available through Docker DNS:

```bash
ping s2       # often fails on the default bridge
nslookup s2   # often fails on the default bridge
```

The lesson:

```text
Default bridge: containers can often talk by IP, but name-based discovery is limited.
Custom networks: containers get Docker DNS and can use container/service names.
```

---

## 9. Why the Default Bridge Can Be Too Broad

If many unrelated containers are all placed on the default `bridge`, they may be able to reach each other by IP.

That can be undesirable:

```text
web app container
monitoring container
random test container
old database container
```

If all are on the same default network, isolation is weak. A compromised or misconfigured container may have access to services it should not even know about.

Better practice:

```text
Create separate custom networks for separate application tiers or projects.
```

Examples:

- `frontend` network for reverse proxies/load balancers
- `backend` network for application servers
- `database` network for databases

---

## 10. Custom Bridge Networks

Create a custom bridge network:

```bash
docker network create backend
```

Docker can choose a subnet automatically, or you can specify one:

```bash
docker network create --subnet 10.0.0.0/24 backend
```

Inspect it:

```bash
docker network inspect backend
```

Typical result:

```text
Network: 10.0.0.0/24
Gateway: 10.0.0.1
Containers: none yet
```

Attach existing containers:

```bash
docker network connect backend s1
docker network connect backend s2
```

A container can be attached to multiple networks at the same time. That means after connecting `s1` to `backend`, it may still also be on the default `bridge`.

Check with:

```bash
docker inspect s1
```

If you want it only on `backend`, disconnect it from `bridge`:

```bash
docker network disconnect bridge s1
docker network disconnect bridge s2
```

---

## 11. Custom Networks Enable DNS Names

On a user-defined bridge network, Docker provides automatic DNS resolution for containers on the same network.

Example:

```bash
docker network create --subnet 10.0.0.0/24 backend

docker run --name s1 --network backend -d nhttpd
docker run --name s2 --network backend -d nhttpd
```

Now from `s1`:

```bash
docker exec -it s1 bash
nslookup s2
curl http://s2
```

And from `s2`:

```bash
nslookup s1
curl http://s1
```

This is extremely useful for multi-container apps. For example, an Nginx container can proxy to an application container by name:

```nginx
proxy_pass http://api:8080;
```

As long as `nginx` and `api` are on the same custom Docker network, Docker DNS can resolve `api`.

Important:

```text
Docker DNS is network-scoped.
A container name is resolvable to containers that share the relevant custom network.
```

---

## 12. Creating Containers Directly on a Network

Instead of creating containers first and connecting later, specify the network at `docker run` time:

```bash
docker run --name s1 --network backend -d nhttpd
```

This avoids accidentally leaving a container attached to the default `bridge`.

Recommended pattern:

```bash
docker network create --subnet 10.0.0.0/24 backend

docker run --name s1 --network backend -d nhttpd
docker run --name s2 --network backend -d nhttpd
```

---

## 13. Internal Networks

A custom network can be created as internal:

```bash
docker network create --internal --subnet 10.0.2.0/24 private_net
```

An internal network is for containers that should communicate with each other but should not have normal outbound access through Docker's gateway to external networks.

Use cases:

- Database-only network
- Sensitive internal services
- Lab environments where internet access should be blocked

Tradeoff:

```text
Internal networks increase isolation, but they also remove easy outbound connectivity.
```

---

## 14. Multi-Tier Network Design

A safer container layout is often tiered:

```text
[Client]
   |
[published port]
   |
[reverse proxy / nginx]
   |
frontend network
   |
[app servers]
   |
backend network
   |
[databases]
```

Why not put everything on one network?

If the reverse proxy is compromised and every container is on the same network, the attacker may reach the database directly.

Better isolation:

```text
Reverse proxy: connected to frontend + app network
App servers:   connected to app network + database network
Database:      connected only to database network
```

This limits which containers can see each other.

---

## 15. Multiple Networks and Container Interfaces

A container can have multiple network attachments, similar to a machine with multiple network cards.

Example:

```bash
docker network create --subnet 10.0.0.0/24 backend
docker network create --subnet 10.0.1.0/24 frontend

# s1 only on backend
docker run --name s1 --network backend -d nhttpd

# s2 only on frontend
docker run --name s2 --network frontend -d nhttpd

# gateway container connected to both
docker run --name gw --network backend -d nhttpd
docker network connect frontend gw
```

Conceptually:

```text
backend:  10.0.0.0/24    s1 ----- gw
frontend: 10.0.1.0/24    s2 ----- gw
```

The `gw` container has one IP on each network. It can see both sides because it is attached to both networks.

---

## 16. Why Two Separate Docker Networks Do Not Automatically Communicate

Suppose:

```text
s1: 10.0.0.2/24 on backend
s2: 10.0.1.2/24 on frontend
```

From `s1`, trying to reach `s2` by IP may fail:

```bash
ping 10.0.1.2
```

Reasons:

1. `10.0.1.2` is not in `s1`'s local subnet.
2. `s1` sends non-local traffic to its default gateway, usually `10.0.0.1`.
3. Docker's bridge gateway may not route between your custom networks in the way you expect.
4. Even if one direction is routed, the reply path must also exist.

This is normal routing behavior. Different subnets need a router and correct routes.

---

## 17. Manually Routing Between Two Docker Networks

The walkthrough demonstrates using a container as a gateway/router between two Docker networks.

Create networks:

```bash
docker network create --subnet 10.0.0.0/24 backend
docker network create --subnet 10.0.1.0/24 frontend
```

Run containers with network-admin capability so routes can be added inside them:

```bash
docker run --name s1 --network backend --cap-add=NET_ADMIN -d nhttpd
docker run --name s2 --network frontend --cap-add=NET_ADMIN -d nhttpd
```

Create a gateway container connected to both networks:

```bash
docker run --name gw --network backend -d nhttpd
docker network connect frontend gw
```

Find the gateway's IPs:

```bash
docker inspect gw
# or from containers:
nslookup gw
hostname -I
```

Example addresses:

```text
s1 on backend: 10.0.0.2
s2 on frontend: 10.0.1.2
gw on backend: 10.0.0.3
gw on frontend: 10.0.1.3
```

Add route on `s1` to reach the frontend subnet through `gw`'s backend IP:

```bash
docker exec -it s1 bash
ip route add 10.0.1.0/24 via 10.0.0.3
```

Add route on `s2` to reach the backend subnet through `gw`'s frontend IP:

```bash
docker exec -it s2 bash
ip route add 10.0.0.0/24 via 10.0.1.3
```

Now test by IP:

```bash
# from s1
ping 10.0.1.2
curl http://10.0.1.2
traceroute 10.0.1.2

# from s2
ping 10.0.0.2
curl http://10.0.0.2
traceroute 10.0.0.2
```

The key networking lesson:

```text
Forward path and return path both matter.
If s2 can receive a packet but has no route back to s1, communication still fails.
```

---

## 18. DNS Limitation Across Separate Networks

After manual routes are added, IP connectivity may work between networks:

```bash
curl http://10.0.0.2
```

But name-based connectivity may still fail:

```bash
curl http://s1
nslookup s1
```

Why? Docker DNS is scoped to Docker networks. A container on `frontend` does not necessarily learn names from `backend` just because there is an IP route between the two subnets.

Practical options:

- Put containers that need name-based discovery on a shared custom network.
- Attach a proxy/service-discovery container to both networks.
- Use Docker Compose service networks intentionally.
- Use an external DNS system for more advanced lab/production setups.
- Use static IPs only for labs, not as the main production discovery mechanism.

---

## 19. Important Commands

### Network inventory

```bash
docker network ls
docker network inspect bridge
docker network inspect backend
```

### Container inventory

```bash
docker ps
docker inspect s1
docker inspect s2
```

### Create networks

```bash
docker network create backend
docker network create --subnet 10.0.0.0/24 backend
docker network create --internal --subnet 10.0.2.0/24 private_net
```

### Connect and disconnect containers

```bash
docker network connect backend s1
docker network disconnect bridge s1
```

### Run containers on networks

```bash
docker run --name s1 --network backend -d nhttpd
docker run --name web -p 8080:80 -d httpd
docker run --name s1 --network backend --cap-add=NET_ADMIN -d nhttpd
```

### Execute commands inside containers

```bash
docker exec -it s1 bash
docker exec s1 hostname -I
docker exec s1 ip route
docker exec s1 nslookup s2
docker exec s1 curl http://s2
```

### Add routes inside containers

```bash
ip route add 10.0.1.0/24 via 10.0.0.3
ip route add 10.0.0.0/24 via 10.0.1.3
```

---

## 20. Troubleshooting Checklist

When Docker networking does not work, check in this order.

### 1. Is the service running?

```bash
docker ps
curl http://localhost:8080
```

If testing a web server, make sure the application inside the container is actually listening.

### 2. Was the port published?

```bash
docker ps
```

Look for output like:

```text
0.0.0.0:8080->80/tcp
```

If there is no published port, host-to-container access through `localhost:<port>` will not work.

### 3. Which network is the container on?

```bash
docker inspect s1
docker network inspect backend
```

Check whether containers are on the same network or separate networks.

### 4. What IP address does the container have?

```bash
docker exec s1 hostname -I
```

Or inspect from Docker:

```bash
docker inspect s1
```

### 5. Does IP connectivity work?

```bash
docker exec -it s1 bash
ping <other-container-ip>
curl http://<other-container-ip>
```

If IP works but names fail, the problem is probably DNS/name discovery.

### 6. Does DNS work?

```bash
nslookup s2
curl http://s2
```

If names fail on the default bridge, move containers to a custom network.

### 7. What does the routing table say?

```bash
ip route
```

Look for:

- The local connected subnet
- The default gateway
- Any manual static routes

### 8. Is there a return route?

If routing between two Docker networks manually, both sides need routes:

```text
s1 needs route to s2's network.
s2 needs route back to s1's network.
```

One-way routing is not enough for ping, curl, or TCP connections.

### 9. Is the network internal?

```bash
docker network inspect private_net
```

If the network was created with `--internal`, containers may not have normal outbound internet access.

---

## 21. Common Mistakes

### Mistake 1 — Confusing `-p 8080:80`

`8080` is the host port. `80` is the container port.

```bash
-p HOST_PORT:CONTAINER_PORT
```

### Mistake 2 — Assuming container IPs are stable

Container IPs can change when containers are recreated. Prefer DNS names on custom networks.

### Mistake 3 — Using the default bridge for everything

The default bridge is convenient but poor for clear app isolation and name-based service discovery.

### Mistake 4 — Forgetting a container can be on multiple networks

After `docker network connect`, the container may still be on its old network too. Inspect and disconnect if necessary.

### Mistake 5 — Expecting DNS to cross networks automatically

A route between subnets does not automatically make Docker DNS names visible across those subnets.

### Mistake 6 — Forgetting `NET_ADMIN`

Adding routes inside a container requires extra privilege:

```bash
--cap-add=NET_ADMIN
```

Without it, `ip route add ...` fails with a permission error.

### Mistake 7 — Forgetting routes are ephemeral

Routes added manually inside a running container disappear when the container is recreated. For repeatable setups, use startup scripts, Docker Compose, or a proper network design.

---

## 22. Practical Lab: Two Web Containers on One Custom Network

```bash
# Build debug image first if needed
# docker build -t nhttpd .

docker network create --subnet 10.10.0.0/24 labnet

docker run --name s1 --network labnet -d nhttpd
docker run --name s2 --network labnet -d nhttpd

# Test name resolution and HTTP from s1 to s2
docker exec s1 nslookup s2
docker exec s1 curl http://s2

# Test the reverse direction
docker exec s2 nslookup s1
docker exec s2 curl http://s1
```

Expected lesson:

```text
Same custom network -> IP connectivity + Docker DNS names work.
```

---

## 23. Practical Lab: Published Web Server

```bash
docker run --name public-web -p 8080:80 -d httpd
curl http://localhost:8080
```

Expected lesson:

```text
Published port -> host can reach container service.
No published port -> host cannot use localhost to reach it directly.
```

Clean up:

```bash
docker stop public-web
docker rm public-web
```

---

## 24. Practical Lab: Two Networks with a Gateway Container

```bash
docker network create --subnet 10.20.0.0/24 backend
docker network create --subnet 10.20.1.0/24 frontend

docker run --name s1 --network backend --cap-add=NET_ADMIN -d nhttpd
docker run --name s2 --network frontend --cap-add=NET_ADMIN -d nhttpd

docker run --name gw --network backend -d nhttpd
docker network connect frontend gw
```

Find IPs:

```bash
docker exec s1 hostname -I
docker exec s2 hostname -I
docker exec gw hostname -I
```

Assume:

```text
s1 = 10.20.0.2
gw = 10.20.0.3 and 10.20.1.3
s2 = 10.20.1.2
```

Add routes:

```bash
docker exec s1 ip route add 10.20.1.0/24 via 10.20.0.3
docker exec s2 ip route add 10.20.0.0/24 via 10.20.1.3
```

Test:

```bash
docker exec s1 ping -c 3 10.20.1.2
docker exec s2 ping -c 3 10.20.0.2
docker exec s1 traceroute 10.20.1.2
docker exec s2 traceroute 10.20.0.2
```

Expected lesson:

```text
Separate subnets require routing.
The gateway container must be reachable from both sides.
Both directions need routes.
```

---

## 25. Docker Compose Note

Most real projects use Docker Compose instead of manually running every command.

Compose can create networks, attach services, and provide service-name DNS automatically. But doing it manually is useful because it shows what Compose is building for you.

A simple Compose mental model:

```text
docker network create ...
docker run --network ...
Docker DNS service names
```

Compose just makes that repeatable in YAML.

---

## 26. Exam / Interview Points

- `docker run -p HOST:CONTAINER` publishes a container port on the Docker host.
- Containers join the default `bridge` network unless another network is specified.
- The default `bridge` allows IP-level communication but is limited for automatic container-name DNS.
- User-defined bridge networks provide Docker DNS for container/service names.
- A container can be connected to multiple Docker networks.
- Inspect networks with `docker network inspect <network>`.
- Inspect a container's network attachments with `docker inspect <container>`.
- Docker Desktop on macOS/Windows uses a VM, so container bridge IP reachability differs from native Linux.
- Custom networks are better for application isolation than putting everything on the default bridge.
- `--internal` networks block normal external connectivity.
- Separate Docker networks are separate IP subnets; routing between them requires a router/gateway and return routes.
- Manual routes inside containers require `--cap-add=NET_ADMIN`.
- Manual `ip route add` changes are not persistent across container recreation.
- DNS reachability and IP reachability are related but separate problems.

---

## 27. One-Line Summary

Docker networking is ordinary IP networking wrapped in Docker automation: use published ports for host access, custom bridge networks for container-to-container DNS and isolation, inspect IPs/routes when troubleshooting, and remember that separate subnets need gateways, routes, and return paths to communicate.
