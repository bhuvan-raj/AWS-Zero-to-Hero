# Load Balancers on AWS

![AWS](https://img.shields.io/badge/AWS-Load%20Balancing-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Type](https://img.shields.io/badge/Reference-Doc-232F3E?style=for-the-badge)

> A reference guide covering load balancing concepts, algorithms, AWS load balancer types, and a detailed ALB vs NLB comparison.

-----

## What is a Load Balancer?

A **Load Balancer** is a component that distributes incoming network traffic across multiple backend servers (targets), ensuring no single server bears too much load.

```
                        ┌─────────────┐
                        │    Client   │
                        └──────┬──────┘
                               │ Request
                               ▼
                      ┌────────────────┐
                      │  Load Balancer │
                      └───┬───┬───┬───┘
                          │   │   │
                    ┌─────┘   │   └─────┐
                    ▼         ▼         ▼
               ┌────────┐ ┌────────┐ ┌────────┐
               │Server 1│ │Server 2│ │Server 3│
               └────────┘ └────────┘ └────────┘
```

**Without a load balancer**, all traffic hits one server — leading to overload, slow responses, and downtime if that server fails.

**With a load balancer**, traffic is spread evenly, providing:

- **High Availability** — if one server fails, others continue serving requests
- **Scalability** — add/remove servers without disrupting users
- **Performance** — prevents any single server from becoming a bottleneck
- **Health Monitoring** — automatically stops routing to unhealthy targets

-----

## Load Balancing Algorithms

Load balancers use different algorithms to decide which server receives the next request.

### Round Robin

Requests are distributed to each server **in order**, cycling through the list repeatedly.

```
Request 1  →  Server A
Request 2  →  Server B
Request 3  →  Server C
Request 4  →  Server A  ← cycles back
Request 5  →  Server B
```

**Best for:** Servers with equal capacity handling stateless requests.  
**Limitation:** Ignores server load — a slow server gets the same traffic as a fast one.

-----

### Weighted Round Robin

Like Round Robin, but servers are assigned **weights** based on capacity. Higher-weight servers receive proportionally more requests.

```
Server A  (weight: 3) → gets 3 requests
Server B  (weight: 1) → gets 1 request
Server C  (weight: 2) → gets 2 requests
```

**Best for:** Heterogeneous server fleets where instances have different CPU/memory specs.

-----

### Least Connections

The next request is sent to the server with the **fewest active connections** at that moment.

```
Server A  → 10 active connections
Server B  →  2 active connections  ← next request goes here
Server C  →  7 active connections
```

**Best for:** Long-lived connections (e.g., database sessions, file transfers) where request duration varies significantly.

-----

### Least Response Time

Routes traffic to the server with the **lowest response time** AND fewest active connections — combining both metrics.

**Best for:** Latency-sensitive applications where response time differences between servers are significant.

-----

### IP Hash (Sticky Sessions)

A hash is computed from the **client’s IP address**, and the request always goes to the same server. Ensures session persistence.

```
Client 192.168.1.10  →  always → Server A
Client 192.168.1.20  →  always → Server B
```

**Best for:** Stateful applications where the server holds session data (e.g., shopping carts, login sessions).  
**Limitation:** Can cause uneven distribution if many users share an IP (e.g., behind corporate NAT).

-----

### Random

Selects a backend server **at random** for each request.

**Best for:** Simple, low-stakes scenarios where all servers are identical and stateless.

-----

## AWS Load Balancer Types

AWS offers **four types** of load balancers under the **Elastic Load Balancing (ELB)** service, each designed for different use cases.

-----

### 1. Application Load Balancer (ALB)

Operates at **Layer 7 (HTTP/HTTPS)**. Makes routing decisions based on the content of the request — URL path, hostname, headers, query strings, or HTTP method.

```
                    ┌──────────┐
                    │   ALB    │
                    └─────┬────┘
          ┌───────────────┼───────────────┐
          │               │               │
   /api/* route    /images/* route   /auth/* route
          │               │               │
   ┌─────────────┐ ┌────────────┐ ┌────────────┐
   │ API Servers │ │ CDN/Static │ │Auth Service│
   └─────────────┘ └────────────┘ └────────────┘
```

**Key features:**

- Content-based routing (path, host, headers, query params)
- Native support for HTTP/2 and WebSockets
- Sticky sessions via cookies
- Built-in authentication (Cognito, OIDC)
- WAF (Web Application Firewall) integration
- Target types: EC2, ECS, Lambda, IP addresses

**Use when:** Building microservices, REST APIs, or any HTTP-based web application.

-----

### 2. Network Load Balancer (NLB)

Operates at **Layer 4 (TCP/UDP/TLS)**. Routes connections based purely on IP protocol data — extremely fast and capable of handling millions of requests per second.

```
          ┌──────────┐
          │   NLB    │   ← Static IP per AZ
          └─────┬────┘
                │  TCP :443
        ┌───────┴────────┐
        ▼                ▼
   ┌─────────┐      ┌─────────┐
   │ EC2 AZ-a│      │ EC2 AZ-b│
   └─────────┘      └─────────┘
```

**Key features:**

- Ultra-low latency (sub-millisecond)
- Static IP addresses per Availability Zone
- Preserves client source IP natively
- TLS termination support
- Handles volatile traffic and TCP/UDP workloads
- Can be used as a VPC PrivateLink endpoint service

**Use when:** You need extreme performance, static IPs, non-HTTP protocols, or gaming/IoT/financial trading workloads.

-----

### 3. Gateway Load Balancer (GWLB)

Operates at **Layer 3 (IP)**. Designed to deploy, scale, and manage **third-party virtual network appliances** such as firewalls, intrusion detection systems (IDS), and deep packet inspection tools.

```
  Inbound Traffic
        │
        ▼
 ┌─────────────┐
 │    GWLB     │  ← Transparent bump-in-the-wire
 └──────┬──────┘
        │ GENEVE protocol (port 6081)
        ▼
 ┌──────────────────┐
 │ Firewall / IDS   │  ← 3rd party appliance
 │ (Palo Alto, etc) │
 └──────────────────┘
        │
        ▼
  Destination
```

**Key features:**

- Transparent to application traffic (no routing changes needed)
- Uses GENEVE encapsulation protocol
- Auto-scales security appliances
- Single entry/exit point for all traffic inspection

**Use when:** You need to insert security appliances (firewalls, IDS/IPS) into your network traffic flow.

-----

### 4. Classic Load Balancer (CLB)

The **legacy** AWS load balancer, operating at both Layer 4 and Layer 7. AWS recommends migrating to ALB or NLB for all new workloads.

**Key features:**

- Basic HTTP/HTTPS and TCP load balancing
- Sticky sessions
- SSL termination

> ⚠️ **Deprecated for new use cases.** Existing CLBs continue to work, but AWS no longer recommends creating new ones. Migrate to ALB or NLB.

**Use when:** Only for maintaining legacy applications already running on CLB.

-----

## ALB vs NLB — Comparison

|Feature                |Application Load Balancer (ALB)                  |Network Load Balancer (NLB)              |
|-----------------------|-------------------------------------------------|-----------------------------------------|
|**OSI Layer**          |Layer 7 (Application)                            |Layer 4 (Transport)                      |
|**Protocols**          |HTTP, HTTPS, HTTP/2, WebSocket                   |TCP, UDP, TLS                            |
|**Routing Logic**      |Content-based (path, host, headers, query params)|Connection-based (IP + port)             |
|**Performance**        |High                                             |Ultra-high (millions of req/sec)         |
|**Latency**            |Low                                              |Sub-millisecond                          |
|**Static IP**          |❌ No (DNS-based)                                 |✅ Yes (per AZ)                           |
|**Elastic IP Support** |❌ No                                             |✅ Yes                                    |
|**Preserves Client IP**|Via `X-Forwarded-For` header                     |✅ Natively                               |
|**SSL/TLS Termination**|✅ Yes                                            |✅ Yes                                    |
|**WebSocket Support**  |✅ Yes                                            |✅ Yes                                    |
|**HTTP/2 Support**     |✅ Yes                                            |❌ No                                     |
|**Sticky Sessions**    |✅ Cookie-based                                   |✅ Source IP-based                        |
|**Target Types**       |EC2, ECS, Lambda, IP                             |EC2, ECS, IP, ALB                        |
|**WAF Integration**    |✅ Yes                                            |❌ No                                     |
|**Authentication**     |✅ Cognito / OIDC                                 |❌ No                                     |
|**VPC PrivateLink**    |❌ No                                             |✅ Yes                                    |
|**Health Checks**      |HTTP, HTTPS, gRPC                                |TCP, HTTP, HTTPS                         |
|**Best For**           |Web apps, APIs, microservices                    |High-perf TCP/UDP, static IP, gaming, IoT|

-----

### When to Choose ALB

- Building a web application or REST API
- Need path-based or host-based routing (microservices)
- Routing to Lambda functions
- Need WAF or Cognito authentication
- HTTP/2 is required

### When to Choose NLB

- Need static/Elastic IP addresses (firewall whitelisting)
- Working with non-HTTP protocols (TCP, UDP)
- Ultra-low latency is critical (gaming, trading, real-time systems)
- Exposing services via AWS PrivateLink
- Client source IP must be preserved at the network layer

-----

## Quick Reference

```
HTTP web app / microservices  →  ALB
TCP/UDP / static IP / gaming  →  NLB
3rd party firewall / IDS/IPS  →  GWLB
Legacy workload (avoid new)   →  CLB
```

-----

*Part of my AWS networking and load balancing reference series.*
