# peering-connections

# AWS VPC Peering Connection & ICMP Connectivity Setup Guide

This repository contains a comprehensive guide for setting up **VPC Peering** between two AWS Virtual Private Clouds (VPCs) and troubleshooting ICMP (`ping`) connectivity issues between EC2 instances.

---

## 🏗️ Architecture Overview

* **VPC A (`first`):** CIDR `10.0.0.0/16` (Subnet: `10.0.1.0/24`)
* **VPC B (`second`):** CIDR `172.31.0.0/16` (Subnet: `172.31.3.128/25`)
* **Peering Connection:** `pcx-0947412334ffceb23`

---

## 📋 Prerequisites

1. Two AWS VPCs with **non-overlapping CIDR blocks**.
2. At least one running EC2 instance in each VPC.
3. VPC Peering, Route Tables, and Security Groups.

---

## 🚀 Step-by-Step Configuration

### 1. Create and Accept VPC Peering Connection
1. Open the **AWS VPC Console** > **Peering connections**.
2. Click **Create peering connection**.
3. Select **VPC A (`first`)** as Requester and **VPC B (`second`)** as Accepter.
4. Select the created Peering Connection and navigate to **Actions** > **Accept request**.
5. Verify that the Peering Connection status is **Active**.

---

### 2. Update Route Tables (Bidirectional Routing)
For traffic to flow between both VPCs, route tables in both VPCs must point to the Peering Connection ID (`pcx-xxxxxxxx`).

#### VPC A Route Table (`rtb-0d4c709bbf8f94d81`)  
| Destination | Target | Status |
| :--- | :--- | :--- |
| `172.31.0.0/16` (VPC B CIDR) | `pcx-0947412334ffceb23` | Active |

#### VPC B Route Table (`rtb-0373a5fb1b3d17618`)
| Destination | Target | Status |
| :--- | :--- | :--- |
| `10.0.1.0/24` (VPC A Subnet CIDR) | `pcx-0947412334ffceb23` | Active |

> ⚠️ **Important:** Do NOT add Public IP addresses in VPC Route Tables. Always use the peered VPC's **Private CIDR block**.

---

### 3. Configure Security Group Rules
Ensure inbound traffic is allowed on both EC2 instances.

* **Inbound Rules (Target Instance):**
  * **Type:** `Custom ICMP Rule - IPv4` (Echo Request) or `All traffic`
  * **Protocol:** ICMP / All
  * **Source:** Peered VPC CIDR (e.g., `10.0.0.0/16` or `172.31.0.0/16`)

---

## 🧪 Testing Connectivity

1. SSH / Session Manager into the source EC2 instance.
2. Ping the **Private IPv4 Address** of the destination EC2 instance:
   ```bash
   ping -c 4 <PRIVATE_IP_OF_TARGET_EC2>

