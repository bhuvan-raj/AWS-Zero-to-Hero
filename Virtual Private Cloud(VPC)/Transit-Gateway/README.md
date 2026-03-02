<div align="center">

![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![VPC](https://img.shields.io/badge/VPC-Networking-blue?style=for-the-badge)
![Transit Gateway](https://img.shields.io/badge/Transit%20Gateway-Advanced-red?style=for-the-badge)

# AWS Transit Gateway — In-Depth Guide

### 🌐 Connect dozens of VPCs and on-premises networks through a single, scalable hub.

</div>

---

## 📋 Table of Contents

- [What is Transit Gateway?](#-what-is-transit-gateway)
- [Why Transit Gateway? — The Problem it Solves](#-why-transit-gateway--the-problem-it-solves)
- [Core Concepts](#-core-concepts)
  - [Transit Gateway](#transit-gateway-tgw)
  - [Attachments](#attachments)
  - [Route Tables](#route-tables)
  - [Associations and Propagations](#associations-and-propagations)
- [Transit Gateway Architecture](#-transit-gateway-architecture)
- [Key Features](#-key-features)
- [Transit Gateway vs VPC Peering](#-transit-gateway-vs-vpc-peering)
- [Lab 1 — Connect VPCs in the Same Region](#-lab-1--connect-vpcs-in-the-same-region)
- [Lab 2 — Transit Gateway Peering Across Regions](#-lab-2--transit-gateway-peering-across-regions)
- [Pricing Overview](#-pricing-overview)
- [Best Practices](#-best-practices)
- [Summary](#-summary)

---

## 💡 What is Transit Gateway?

**AWS Transit Gateway (TGW)** is a **network transit hub** that allows you to interconnect multiple VPCs, AWS accounts, and on-premises networks through a single, centrally managed gateway.

Instead of creating point-to-point connections between every pair of VPCs, all networks connect to the Transit Gateway — and the TGW routes traffic between them. This is the **hub-and-spoke model**.

```
                         ┌─────────────────────┐
                         │   TRANSIT GATEWAY   │
                         │   (Central Hub)     │
                         └──────────┬──────────┘
              ┌───────────┬─────────┼──────────┬───────────┐
              │           │         │          │           │
           ┌──▼──┐     ┌──▼──┐   ┌──▼──┐   ┌──▼──┐   ┌────▼────┐
           │VPC A│     │VPC B│   │VPC C│   │VPC D│   │On-Prem  │
           │Dev  │     │Prod │   │Data │   │Mgmt │   │(VPN/DX) │
           └─────┘     └─────┘   └─────┘   └─────┘   └─────────┘
```

Every VPC attaches to the TGW once — and can communicate with every other attached network based on the routing rules you define.

---

## 🔥 Why Transit Gateway? — The Problem it Solves

### The VPC Peering Scaling Problem

Before Transit Gateway, organizations connected VPCs using **VPC Peering** — direct, one-to-one connections. This worked fine for a small number of VPCs, but it does not scale.

**VPC Peering is non-transitive.** If VPC A is peered with VPC B, and VPC B is peered with VPC C — traffic from VPC A **cannot** reach VPC C through VPC B. You must create a separate direct peering between A and C.

```
                   ❌ WITHOUT TRANSIT GATEWAY (VPC Peering Mesh)

     VPC A ◀────▶ VPC B
       │    ╲    ╱  │
       │      ╲╱   │
       │      ╱╲   │
       │    ╱    ╲  │
     VPC D ◀────▶ VPC C

   4 VPCs require 6 peering connections
   10 VPCs require 45 peering connections
   n VPCs require n(n-1)/2 connections   ← impossible to manage
```

```
                   ✅ WITH TRANSIT GATEWAY (Hub and Spoke)

     VPC A ─────┐
     VPC B ─────┤
     VPC C ─────┼──▶  TRANSIT GATEWAY  ──▶ routes traffic
     VPC D ─────┤
     On-Prem ───┘

   100 VPCs require 100 attachments — 1 per VPC
   Simple, scalable, centrally managed
```

### Transit Gateway solves:

- **Scalability** — connect hundreds of VPCs without a mesh of peering connections
- **Transitivity** — traffic can flow between any two attached networks
- **Centralized routing** — manage all routing from one place
- **Cross-account connectivity** — share a TGW across AWS accounts via Resource Access Manager (RAM)
- **On-premises integration** — connect VPN and Direct Connect to all VPCs at once

---

## 🧩 Core Concepts

### Transit Gateway (TGW)

The TGW itself is a **regional resource** — it lives in a specific AWS region. It acts as the central router and maintains its own route tables. You can have multiple TGWs per region, and peer them across regions.

Key TGW attributes you configure at creation:

| Attribute | Description | Default |
|-----------|-------------|---------|
| Amazon ASN | BGP Autonomous System Number for routing | `64512` |
| DNS Support | Enable DNS resolution across attachments | Enabled |
| VPN ECMP Support | Equal-cost multi-path routing for VPN | Enabled |
| Default Route Table Association | Auto-associate new attachments to default RT | Enabled |
| Default Route Table Propagation | Auto-propagate routes to default RT | Enabled |

---

### Attachments

An **attachment** is how a network connects to the Transit Gateway. Each attachment type brings that network into the TGW routing domain.

| Attachment Type | What It Connects |
|----------------|-----------------|
| **VPC Attachment** | An AWS VPC (you select subnets per AZ) |
| **VPN Attachment** | A Site-to-Site VPN connection to on-premises |
| **Direct Connect Gateway Attachment** | AWS Direct Connect to on-premises |
| **Transit Gateway Peering Attachment** | Another Transit Gateway (same or different region) |
| **Connect Attachment** | SD-WAN appliances using GRE tunnels |

When you attach a VPC, you specify **one subnet per Availability Zone**. TGW places an **Elastic Network Interface (ENI)** in each selected subnet — this is how traffic enters and exits the TGW for that VPC.

---

### Route Tables

A **TGW Route Table** is what determines where traffic goes. It contains routes like any router:

```
Destination CIDR    →    Next Hop (Attachment)
──────────────────────────────────────────────
10.0.0.0/16         →    VPC-A attachment
10.1.0.0/16         →    VPC-B attachment
10.2.0.0/16         →    VPC-C attachment
172.16.0.0/12       →    VPN attachment (on-prem)
0.0.0.0/0           →    Inspection VPC attachment
```

You can have **multiple route tables** on a single TGW to create isolated routing domains — for example, a "Dev" route table that only lets Dev VPCs communicate, and a "Prod" route table that isolates production traffic.

---

### Associations and Propagations

Every attachment has two routing relationships with TGW route tables:

**Association** — which TGW route table does this attachment use to *look up* where to send its outbound traffic? Each attachment can be associated with **one** route table.

**Propagation** — which TGW route tables should this attachment *advertise* its CIDR into? An attachment can propagate into **multiple** route tables.

```
VPC A (10.0.0.0/16)
    Associated with:  Route Table "Production"
    Propagates into:  Route Table "Production", Route Table "Shared-Services"

This means:
  - VPC A uses the "Production" table to find where to send traffic
  - VPC A's CIDR (10.0.0.0/16) appears as a route in both tables
```

---

## 🏗️ Transit Gateway Architecture

### Full Architecture with Route Table Segmentation

```
┌─────────────────────────────────────────────────────────────────────┐
│                         AWS REGION (us-east-1)                      │
│                                                                     │
│   VPC-Dev          VPC-Prod         VPC-Shared        On-Prem       │
│  10.0.0.0/16      10.1.0.0/16      10.2.0.0/16      192.168.0.0/16 │
│      │                │                │                │           │
│      │ attachment      │ attachment     │ attachment     │ VPN       │
│      └────────┬────────┘               │                │           │
│               │                        │                │           │
│    ╔══════════▼════════════════════════▼════════════════▼═════════╗ │
│    ║               TRANSIT GATEWAY                                ║ │
│    ╠════════════════════════════════════════════════════════════╗ ║ │
│    ║  RT: Dev          RT: Prod          RT: Shared-Services    ║ ║ │
│    ║  ──────────        ───────────       ──────────────────    ║ ║ │
│    ║  10.0.0.0/16       10.1.0.0/16       10.0.0.0/16          ║ ║ │
│    ║  10.2.0.0/16       10.2.0.0/16       10.1.0.0/16          ║ ║ │
│    ║                    192.168.0.0/16     10.2.0.0/16          ║ ║ │
│    ╚════════════════════════════════════════════════════════════╝ ║ │
│    ╚══════════════════════════════════════════════════════════════╝ │
└─────────────────────────────────────────────────────────────────────┘
```

In this setup:
- Dev VPCs can talk to Shared Services but **not** to Prod
- Prod VPCs can talk to Shared Services and On-Premises
- Each routing domain is enforced by which route table each attachment is associated with

---

## ⚡ Key Features

**Multicast Support** — TGW supports IP multicast, allowing one source to send traffic to multiple destinations simultaneously. Useful for media streaming and financial data distribution.

**Network Manager** — A global view of your AWS and on-premises network topology, monitoring, and route analysis in a single dashboard.

**Flow Logs** — Enable TGW Flow Logs to capture information about IP traffic going through your transit gateway for security and troubleshooting.

**Appliance Mode** — When enabled on a VPC attachment, TGW maintains flow stickiness to the same Availability Zone for the lifetime of a connection — required for stateful appliances like firewalls and NAT.

**Equal-Cost Multi-Path (ECMP)** — Distribute traffic across multiple VPN tunnels to a single destination for increased bandwidth and redundancy.

---

## ⚖️ Transit Gateway vs VPC Peering

| Feature | Transit Gateway | VPC Peering |
|---------|----------------|-------------|
| Transitivity | ✅ Transitive routing | ❌ Non-transitive |
| Scalability | ✅ Hundreds of VPCs | ❌ Complex mesh beyond ~10 VPCs |
| Centralized Management | ✅ Single place for all routing | ❌ Manage every peering individually |
| Cross-Region | ✅ Via TGW Peering | ✅ Native cross-region peering |
| Cross-Account | ✅ Via AWS RAM | ✅ Supported |
| On-Premises Connectivity | ✅ VPN / Direct Connect attachment | ❌ Not supported |
| Bandwidth | Up to 50 Gbps per attachment | Up to 10 Gbps burst |
| Latency | Slightly higher (extra hop) | Lower (direct connection) |
| Cost | Per attachment + per GB | Per GB only (no hourly charge) |
| Best For | Large-scale, hub-and-spoke networks | Simple, few VPC connections |

> 💡 **Rule of thumb:** Use VPC Peering for simple, low-cost, two-VPC connections. Use Transit Gateway when you need centralized routing, more than 3-4 VPCs, or on-premises connectivity.

---

## 🧪 Lab 1 — Connect VPCs in the Same Region

### Objective

Connect **three VPCs** in the same AWS region using a Transit Gateway so that all three can communicate with each other.

```
VPC-A (10.0.0.0/16)  ──┐
                        ├──▶  Transit Gateway  ──▶  Full mesh routing
VPC-B (10.1.0.0/16)  ──┤
                        │
VPC-C (10.2.0.0/16)  ──┘
```

### Architecture

```
Region: us-east-1

┌────────────────┐   ┌────────────────┐   ┌────────────────┐
│     VPC-A      │   │     VPC-B      │   │     VPC-C      │
│  10.0.0.0/16  │   │  10.1.0.0/16  │   │  10.2.0.0/16  │
│               │   │               │   │               │
│ ┌───────────┐ │   │ ┌───────────┐ │   │ ┌───────────┐ │
│ │  Subnet A │ │   │ │  Subnet B │ │   │ │  Subnet C │ │
│ │10.0.1.0/24│ │   │ │10.1.1.0/24│ │   │ │10.2.1.0/24│ │
│ │  EC2: A   │ │   │ │  EC2: B   │ │   │ │  EC2: C   │ │
│ └─────┬─────┘ │   │ └─────┬─────┘ │   │ └─────┬─────┘ │
└───────┼────────┘   └───────┼────────┘   └───────┼────────┘
        │                    │                    │
        └──────────┬─────────┘                    │
                   │           ┌───────────────────┘
                   ▼           ▼
            ┌─────────────────────┐
            │   TRANSIT GATEWAY   │
            │   TGW Route Table   │
            │  10.0.0.0/16 → A   │
            │  10.1.0.0/16 → B   │
            │  10.2.0.0/16 → C   │
            └─────────────────────┘
```

### Prerequisites

- AWS account with sufficient permissions (EC2, VPC, Transit Gateway)
- AWS CLI configured or access to AWS Console
- Basic understanding of VPCs and subnets

---

### Step 1 — Create Three VPCs

Create each VPC with a unique, non-overlapping CIDR block. **CIDR ranges must not overlap across VPCs** — this is a hard requirement for Transit Gateway routing.

**Create VPC-A:**
1. Go to **VPC Console → Your VPCs → Create VPC**
2. Configure:

| Field | Value |
|-------|-------|
| Name | `VPC-A` |
| IPv4 CIDR | `10.0.0.0/16` |
| Tenancy | Default |

**Create VPC-B:**

| Field | Value |
|-------|-------|
| Name | `VPC-B` |
| IPv4 CIDR | `10.1.0.0/16` |

**Create VPC-C:**

| Field | Value |
|-------|-------|
| Name | `VPC-C` |
| IPv4 CIDR | `10.2.0.0/16` |

---

### Step 2 — Create Subnets

Create one subnet in each VPC. These subnets will host the EC2 instances used for testing.

| VPC | Subnet Name | CIDR | Availability Zone |
|-----|-------------|------|------------------|
| VPC-A | `Subnet-A` | `10.0.1.0/24` | `us-east-1a` |
| VPC-B | `Subnet-B` | `10.1.1.0/24` | `us-east-1a` |
| VPC-C | `Subnet-C` | `10.2.1.0/24` | `us-east-1a` |

Also create one **TGW attachment subnet** per VPC (best practice — keep TGW ENIs in a dedicated subnet):

| VPC | Subnet Name | CIDR |
|-----|-------------|------|
| VPC-A | `TGW-Subnet-A` | `10.0.2.0/28` |
| VPC-B | `TGW-Subnet-B` | `10.1.2.0/28` |
| VPC-C | `TGW-Subnet-C` | `10.2.2.0/28` |

---

### Step 3 — Create the Transit Gateway

1. Go to **VPC Console → Transit Gateways → Create Transit Gateway**
2. Configure:

| Field | Value |
|-------|-------|
| Name | `Lab-TGW` |
| Description | `Transit Gateway for same-region lab` |
| Amazon side ASN | `64512` |
| DNS Support | ✅ Enable |
| VPN ECMP Support | ✅ Enable |
| Default Route Table Association | ✅ Enable |
| Default Route Table Propagation | ✅ Enable |

3. Click **Create Transit Gateway**

> ⏳ Wait for the TGW state to change from `pending` to `available` — this takes about 5 minutes.

---

### Step 4 — Create VPC Attachments

Attach each VPC to the Transit Gateway.

**For VPC-A:**
1. Go to **VPC Console → Transit Gateway Attachments → Create Transit Gateway Attachment**
2. Configure:

| Field | Value |
|-------|-------|
| Transit Gateway ID | Select `Lab-TGW` |
| Attachment Type | `VPC` |
| Attachment Name | `Attach-VPC-A` |
| VPC ID | Select `VPC-A` |
| Subnet IDs | Select `TGW-Subnet-A` |

3. Click **Create**

Repeat for **VPC-B** (`Attach-VPC-B` → `TGW-Subnet-B`) and **VPC-C** (`Attach-VPC-C` → `TGW-Subnet-C`).

> ⏳ Wait for all three attachments to show state `available` before proceeding.

---

### Step 5 — Verify the TGW Route Table

Because **Default Route Table Association** and **Default Route Table Propagation** were enabled, the TGW automatically:
- Associated all three attachments to the default route table
- Propagated each VPC's CIDR into the route table

1. Go to **Transit Gateway Route Tables**
2. Select the default route table → click **Routes** tab

You should see:

| Destination | Attachment | Type |
|-------------|-----------|------|
| `10.0.0.0/16` | Attach-VPC-A | propagated |
| `10.1.0.0/16` | Attach-VPC-B | propagated |
| `10.2.0.0/16` | Attach-VPC-C | propagated |

---

### Step 6 — Update VPC Route Tables

The TGW knows how to route between VPCs — but the **VPC route tables** need to know to send traffic destined for other VPCs to the TGW.

**For VPC-A's route table:**
1. Go to **VPC Console → Route Tables** → select the route table associated with `Subnet-A`
2. Click **Edit Routes → Add Route**

| Destination | Target |
|-------------|--------|
| `10.1.0.0/16` | Transit Gateway → `Lab-TGW` |
| `10.2.0.0/16` | Transit Gateway → `Lab-TGW` |

Repeat for **VPC-B** (add routes for `10.0.0.0/16` and `10.2.0.0/16`) and **VPC-C** (add routes for `10.0.0.0/16` and `10.1.0.0/16`).

> 💡 **Tip:** If you have many VPCs, use a summarized supernet route (e.g., `10.0.0.0/8`) pointing to the TGW instead of individual routes per VPC.

---

### Step 7 — Launch EC2 Instances for Testing

Launch one EC2 instance in each VPC's workload subnet.

| Instance | VPC | Subnet | Security Group Rule |
|----------|-----|--------|---------------------|
| `EC2-A` | VPC-A | Subnet-A | Allow ICMP from `10.0.0.0/8` |
| `EC2-B` | VPC-B | Subnet-B | Allow ICMP from `10.0.0.0/8` |
| `EC2-C` | VPC-C | Subnet-C | Allow ICMP from `10.0.0.0/8` |

> ⚠️ Security Groups are stateful but they must explicitly allow ICMP (ping) inbound from the other VPC CIDR ranges. Do not use `0.0.0.0/0` in production.

---

### Step 8 — Test Connectivity

SSH into `EC2-A` and ping the private IPs of `EC2-B` and `EC2-C`:

```bash
# From EC2-A (in VPC-A)
ping 10.1.1.x    # EC2-B private IP — should succeed ✅
ping 10.2.1.x    # EC2-C private IP — should succeed ✅
```

```bash
# From EC2-B (in VPC-B)
ping 10.0.1.x    # EC2-A private IP — should succeed ✅
ping 10.2.1.x    # EC2-C private IP — should succeed ✅
```

If pings succeed, your Transit Gateway is routing traffic correctly between all three VPCs. 🎉

### Lab 1 — Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Ping times out | VPC route table missing TGW route | Add route to the VPC RT pointing to TGW |
| Ping times out | Security Group blocking ICMP | Add inbound ICMP allow rule for source CIDR |
| Attachment stuck in `pending` | TGW still initializing | Wait for TGW state to become `available` |
| No routes in TGW RT | Propagation not enabled | Enable propagation for each attachment on the TGW RT |
| Wrong subnet selected for attachment | TGW ENI in wrong AZ | Recreate attachment selecting the correct TGW subnet |

---

## 🌍 Lab 2 — Transit Gateway Peering Across Regions

### Objective

Connect VPCs in **two different AWS regions** (us-east-1 and ap-southeast-1) using **Transit Gateway Peering** so that workloads in both regions can communicate privately.

```
us-east-1                              ap-southeast-1
──────────────────────────────────     ──────────────────────────────────
VPC-US (10.0.0.0/16)                   VPC-AP (10.3.0.0/16)
     │                                       │
     └──▶ TGW-US ◀── TGW Peering ──▶ TGW-AP ◀──┘
```

### Architecture

```
┌───────────────────────────────────┐      ┌───────────────────────────────────┐
│         Region: us-east-1         │      │        Region: ap-southeast-1     │
│                                   │      │                                   │
│  ┌─────────────────────────────┐  │      │  ┌─────────────────────────────┐  │
│  │         VPC-US              │  │      │  │         VPC-AP              │  │
│  │       10.0.0.0/16           │  │      │  │       10.3.0.0/16           │  │
│  │  ┌──────────────────────┐   │  │      │  │  ┌──────────────────────┐   │  │
│  │  │  Subnet  10.0.1.0/24 │   │  │      │  │  │  Subnet  10.3.1.0/24 │   │  │
│  │  │  EC2-US              │   │  │      │  │  │  EC2-AP              │   │  │
│  │  └──────────┬───────────┘   │  │      │  │  └──────────┬───────────┘   │  │
│  └─────────────┼───────────────┘  │      │  └─────────────┼───────────────┘  │
│                │                   │      │                │                   │
│         ┌──────▼──────┐            │      │         ┌──────▼──────┐            │
│         │   TGW-US    │            │      │         │   TGW-AP    │            │
│         │  (us-east-1)│◀───────────┼──────┼────────▶│(ap-southeast│            │
│         └─────────────┘  Peering   │      │         └─────────────┘            │
│                           Attachment│      │                                   │
└───────────────────────────────────┘      └───────────────────────────────────┘
```

### Prerequisites

- Two AWS regions available: `us-east-1` and `ap-southeast-1`
- Same AWS account (cross-account TGW peering is also supported via RAM but not covered here)
- IAM permissions for Transit Gateway in both regions

---

### Step 1 — Create VPCs in Both Regions

**In us-east-1 — Create VPC-US:**

| Field | Value |
|-------|-------|
| Name | `VPC-US` |
| CIDR | `10.0.0.0/16` |
| Subnet Name | `Subnet-US` |
| Subnet CIDR | `10.0.1.0/24` |
| TGW Subnet | `TGW-Subnet-US` — `10.0.2.0/28` |

**In ap-southeast-1 — Create VPC-AP:**

> ⚠️ Switch your AWS Console region to **Asia Pacific (Singapore)** before creating these resources.

| Field | Value |
|-------|-------|
| Name | `VPC-AP` |
| CIDR | `10.3.0.0/16` |
| Subnet Name | `Subnet-AP` |
| Subnet CIDR | `10.3.1.0/24` |
| TGW Subnet | `TGW-Subnet-AP` — `10.3.2.0/28` |

---

### Step 2 — Create a Transit Gateway in Each Region

**In us-east-1:**

| Field | Value |
|-------|-------|
| Name | `TGW-US` |
| Amazon side ASN | `64512` |
| Default RT Association | ✅ Enable |
| Default RT Propagation | ✅ Enable |

**In ap-southeast-1:**

> ⚠️ Switch console region to ap-southeast-1.

| Field | Value |
|-------|-------|
| Name | `TGW-AP` |
| Amazon side ASN | `64513` |
| Default RT Association | ✅ Enable |
| Default RT Propagation | ✅ Enable |

> ⚠️ **Each TGW must have a unique ASN.** `TGW-US` uses `64512`, `TGW-AP` uses `64513`. They must be different.

Wait for both TGWs to reach `available` state.

---

### Step 3 — Attach VPCs to Their Local TGW

**In us-east-1 — Attach VPC-US to TGW-US:**

| Field | Value |
|-------|-------|
| Attachment Name | `Attach-VPC-US` |
| Transit Gateway | `TGW-US` |
| Attachment Type | VPC |
| VPC | `VPC-US` |
| Subnet | `TGW-Subnet-US` |

**In ap-southeast-1 — Attach VPC-AP to TGW-AP:**

| Field | Value |
|-------|-------|
| Attachment Name | `Attach-VPC-AP` |
| Transit Gateway | `TGW-AP` |
| Attachment Type | VPC |
| VPC | `VPC-AP` |
| Subnet | `TGW-Subnet-AP` |

Wait for both attachments to reach `available`.

---

### Step 4 — Create the Transit Gateway Peering Attachment

This is the cross-region connection between the two TGWs. The **requester** side creates the peering attachment and specifies the **accepter** TGW in the other region.

**In us-east-1 (Requester side):**

1. Go to **VPC Console → Transit Gateway Attachments → Create Transit Gateway Attachment**
2. Configure:

| Field | Value |
|-------|-------|
| Transit Gateway ID | `TGW-US` |
| Attachment Type | `Peering Connection` |
| Attachment Name | `TGW-Peering-US-to-AP` |
| Account | My Account |
| Region | `ap-southeast-1` |
| Transit Gateway ID (peer) | ID of `TGW-AP` |

3. Click **Create**

The attachment state will show as `pending acceptance`.

---

### Step 5 — Accept the Peering Attachment

**Switch to ap-southeast-1 (Accepter side):**

1. Go to **VPC Console → Transit Gateway Attachments**
2. Find the peering attachment with state `pendingAcceptance`
3. Select it → click **Actions → Accept Transit Gateway Attachment**
4. Confirm

The attachment state will transition to `available` (takes a few minutes).

---

### Step 6 — Configure TGW Route Tables — Static Routes

Unlike VPC attachments (which auto-propagate their CIDRs), **peering attachments do not support route propagation**. You must add static routes manually on both sides.

**In us-east-1 — TGW-US Route Table:**

1. Go to **Transit Gateway Route Tables** → select TGW-US default route table
2. Click **Routes → Create Static Route**

| Destination | Attachment |
|-------------|-----------|
| `10.3.0.0/16` | `TGW-Peering-US-to-AP` |

This tells TGW-US: "Send traffic destined for the AP region's VPC through the peering attachment."

**In ap-southeast-1 — TGW-AP Route Table:**

Switch console to ap-southeast-1.

1. Go to **Transit Gateway Route Tables** → select TGW-AP default route table
2. Click **Routes → Create Static Route**

| Destination | Attachment |
|-------------|-----------|
| `10.0.0.0/16` | `TGW-Peering-US-to-AP` (peering attachment in this region) |

---

### Step 7 — Update VPC Route Tables in Both Regions

The VPCs also need to know to send cross-region traffic to their local TGW.

**VPC-US route table (in us-east-1):**

| Destination | Target |
|-------------|--------|
| `10.3.0.0/16` | Transit Gateway → `TGW-US` |

**VPC-AP route table (in ap-southeast-1):**

| Destination | Target |
|-------------|--------|
| `10.0.0.0/16` | Transit Gateway → `TGW-AP` |

---

### Step 8 — Launch EC2 Instances and Configure Security Groups

**In us-east-1:**
- Launch `EC2-US` in `Subnet-US` (10.0.1.0/24)
- Security Group: Allow ICMP inbound from `10.3.0.0/16`

**In ap-southeast-1:**
- Launch `EC2-AP` in `Subnet-AP` (10.3.1.0/24)
- Security Group: Allow ICMP inbound from `10.0.0.0/16`

---

### Step 9 — Test Cross-Region Connectivity

SSH into `EC2-US` and ping the **private IP** of `EC2-AP`:

```bash
# From EC2-US (us-east-1)
ping 10.3.1.x    # EC2-AP private IP — should succeed ✅
```

```bash
# From EC2-AP (ap-southeast-1)
ping 10.0.1.x    # EC2-US private IP — should succeed ✅
```

Traffic path:
```
EC2-US → VPC-US subnet → TGW-US → Peering Attachment → TGW-AP → VPC-AP subnet → EC2-AP
```

All traffic remains **within the AWS backbone network** — it never traverses the public internet. 🎉

---

### Lab 2 — Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Peering attachment stuck in `pending acceptance` | Not accepted in peer region | Switch to the peer region and accept the attachment |
| Routes not working after peering | Static routes not added | Peering does not auto-propagate — add static routes manually on both TGW route tables |
| Ping fails despite routes existing | Security Groups not updated | Add inbound ICMP rule allowing the remote VPC CIDR |
| Attachment shows `failed` | Duplicate ASN on both TGWs | Each TGW must have a unique ASN |
| VPC route table not pointing to TGW | Missed the VPC RT update step | Add route to VPC RT with TGW as target for the remote CIDR |
| High latency on pings | Expected for cross-region | Cross-region traffic adds ~150-200ms depending on regions |

---

## 💰 Pricing Overview

Transit Gateway pricing has two components:

| Component | Cost |
|-----------|------|
| **Attachment fee** | ~$0.05 per attachment-hour (each VPC/VPN/peering attachment) |
| **Data processing** | ~$0.02 per GB of data processed through the TGW |

**Cross-region peering** adds standard **inter-region data transfer** costs on top of TGW data processing fees.

> 💡 Always **delete TGW attachments and the TGW itself** when done with labs — attachments are billed hourly even when no traffic is flowing.

---

## ✅ Best Practices

| Practice | Why It Matters |
|----------|---------------|
| Use dedicated TGW subnets per AZ | Isolates TGW ENIs from workload traffic and simplifies troubleshooting |
| Plan non-overlapping CIDRs upfront | Overlapping CIDRs cannot be routed — plan your IP space before scaling |
| Use multiple route tables for segmentation | Isolate Dev/Prod/Shared without separate TGWs |
| Enable TGW Flow Logs | Essential for security auditing and network troubleshooting |
| Use Appliance Mode for stateful inspection | Prevents asymmetric routing through firewalls |
| Set `Instance Cap` carefully for ECMP | Too many paths can cause uneven load distribution |
| Share TGW via AWS RAM in multi-account setups | Avoid deploying a TGW per account — share one centrally |
| Tag all TGW resources | Cost allocation and filtering in AWS Cost Explorer |

---

## 📌 Summary

| Concept | Key Point |
|---------|-----------|
| **Transit Gateway** | Regional hub-and-spoke router — replaces complex VPC peering meshes |
| **Attachment** | How a VPC, VPN, or peer TGW connects to the TGW |
| **Route Table** | Controls where the TGW forwards traffic — supports multiple tables for segmentation |
| **Association** | Which route table an attachment uses to look up outbound routes |
| **Propagation** | Which route tables an attachment advertises its CIDR into |
| **Same-Region Lab** | VPC attachments auto-propagate routes — update VPC RTs to point to TGW |
| **Cross-Region Lab** | Peering attachments require manual static routes on both TGW route tables |
| **Peering vs Mesh** | TGW scales to 100s of VPCs; VPC peering mesh breaks down beyond ~10 VPCs |

---

<div align="center">

*Part of the AWS Networking series*

</div>
