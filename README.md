## 🎬 Watch Me Build This Lab!

# Lab 02: Building a Secure 2-Tier Web Application

**Author:** Jason Roland
**Estimated Time:** 60 Minutes
**Difficulty:** Beginner

---

## 📌 Objective

In this lab, you build a classic **Infrastructure-as-a-Service (IaaS) 2-tier architecture** in Microsoft Azure. The goal is to demonstrate how to securely separate a **public-facing web server** from a **private backend database server** using **Virtual Networks, Subnets, and Network Security Groups (NSGs)**.

By the end of this lab, only the web server will be accessible from the internet, while the database server remains fully isolated and reachable **only** from within the virtual network.

---

## 🏗️ Architecture Overview

**High-level design:**

* A single Azure Virtual Network
* Two subnets:

  * **Public Subnet** → Web Server (Internet-facing)
  * **Private Subnet** → Database Server (No public access)
* Controlled access using NSGs

```
Internet
   |
   v
[ Public Subnet ]
   vm-web-01
        |
        v
[ Private Subnet ]
   vm-db-01
```

---

## ✅ Prerequisites

* Active **Azure Subscription**
* Completed **Week 2 Video Modules**
* Terminal installed (macOS, Linux, or Windows with WSL)
* Basic familiarity with SSH

---

## 🏷️ Lab Variables (Naming Convention)

| Resource        | Name                     |
| --------------- | ------------------------ |
| Resource Group  | `rg-lab02-jason`         |
| Virtual Network | `vnet-lab02`             |
| Public Subnet   | `snet-web` (10.0.1.0/24) |
| Private Subnet  | `snet-db` (10.0.2.0/24)  |
| Web VM          | `vm-web-01`              |
| Database VM     | `vm-db-01`               |

---

## 🚀 Step-by-Step Instructions

### Phase 1: Network Foundation

1. Navigate to **Virtual Networks → Create**
2. Configure:

   * **Resource Group:** Create new → `rg-lab02-jason`
   * **Name:** `vnet-lab02`
   * **Region:** East US
3. IP Addressing:

   * Address space: `10.0.0.0/16`
   * Subnet 1: `snet-web` → `10.0.1.0/24`
   * Subnet 2: `snet-db` → `10.0.2.0/24`
4. Review + Create

---

### Phase 2: Deploy the Web Server (Front End)

1. Navigate to **Virtual Machines → Create**
2. Configure:

   * **Name:** `vm-web-01`
   * **Image:** Ubuntu Server 20.04 LTS
   * **Size:** Standard_B1s
   * **Key Pair:** `key-lab02`
   * **Inbound Ports:** SSH (22), HTTP (80)
3. Networking:

   * Subnet: `snet-web`
   * Public IP: Create new (Standard)
4. Create VM and download the `.pem` key if prompted

---

### Phase 3: Deploy the Database Server (Back End)

1. Create another Virtual Machine
2. Configure:

   * **Name:** `vm-db-01`
   * **Image:** Ubuntu Server 20.04 LTS
   * **Size:** Standard_B1s
   * **Key Pair:** Use existing → `key-lab02`
   * **Inbound Ports:** SSH (22)
3. **Critical Networking Configuration:**

   * Virtual Network: `vnet-lab02`
   * Subnet: `snet-db`
   * **Public IP:** None
4. Review + Create

---

### Phase 4: Validate Internal Connectivity (Jump Host Pattern)

Since the database VM has no public IP, access must occur through the web VM.

1. Retrieve the **Private IP** of `vm-db-01` (example: `10.0.2.4`)
2. SSH into the web server:

```bash
ssh -i key-lab02.pem azureuser@<web-public-ip>
```

3. From inside the web VM, test connectivity:

```bash
ping 10.0.2.4
```

4. Successful replies confirm internal VNet communication

---

### Phase 5: Secure the Database with NSGs

1. Navigate to **vm-db-01 → Networking**
2. Open the attached **Network Security Group**
3. Add an inbound rule:

   * **Source:** IP Addresses
   * **Source CIDR:** `10.0.1.0/24`
   * **Destination Port:** `*` (or 3306 / 5432)
   * **Action:** Allow
   * **Priority:** 100
   * **Name:** `Allow-Web-Subnet`

This ensures only the web subnet can access the database server.

---

## 🛠️ Troubleshooting

**Issue:** Ping fails

* Ensure both VMs are in the same VNet
* Confirm `vm-db-01` is in `snet-db`

**Issue:** Cannot SSH into database VM

* This is expected behavior
* Database VM has no public IP
* Access must be performed via the web VM

---

## 🧹 Clean Up

To avoid unnecessary charges:

* Delete the resource group:

```
rg-lab02-jason
```

---

## 🎯 Skills Demonstrated

* Azure Virtual Networking
* Subnet segmentation
* Network Security Groups (NSGs)
* Secure IaaS architecture
* Jump host access pattern

---

## 📎 Notes

This lab serves as a foundational security pattern used in real-world cloud environments and can be extended with:

* Azure Bastion
* Load Balancers
* Managed Databases
* Monitoring and Logging
