# ⭐ **AWS VPC – Important Points, Definitions, Tricks & Commands**

---

# 🧠 **1. Core Definitions (Must Remember)**

### **VPC (Virtual Private Cloud)**

Your own isolated private network inside AWS.

### **CIDR (Classless Inter-Domain Routing)**

Range of IP Address block → example: `10.0.0.0/16`

### **Subnet**

A smaller segment inside VPC used for isolation.

* **Public Subnet** = Has route to **Internet Gateway (IGW)**
* **Private Subnet** = No direct Internet access

### **Route Table**

Controls traffic routing rules for subnets.

### **Internet Gateway (IGW)**

Allows **public** subnets to communicate with internet.

### **NAT Gateway**

Allows **private** subnets to access internet **OUTBOUND** (yum update, apt update).

### **Security Group**

Firewall at **instance level** — *Stateful*

### **NACL**

Firewall at **subnet level** — *Stateless*

### **VPC Endpoint**

Private connection to AWS services **without internet**

* **Gateway Endpoint** → S3 & DynamoDB
* **Interface Endpoint** → Most other services (ENI based)

### **Elastic IP (EIP)**

Static public IP address.

### **Peering**

Connect two VPCs (no transitive routing).

### **Transit Gateway (TGW)**

Hub-and-spoke multi-VPC + hybrid connections.

---

# 💡 **2. VPC CIDR Quick Tricks**

### ✔ Trick 1: CIDR Quick Math

`/16` = 65,536 IPs
`/20` = 4,096 IPs
`/24` = 256 IPs
`/28` = 16 IPs

### ✔ Trick 2: AWS Reserves 5 IPs

Example: `10.0.1.0/24`

* .0 → Network
* .1 → VPC Router
* .2 → DNS
* .3 → Future Use
* .255 → Broadcast

Usable = **256 – 5 = 251 IPs**

### ✔ Trick 3: Public vs Private subnet identification

* Has default route → IGW → **Public**
* Has default route → NAT → **Private**

### ✔ Trick 4: Subnet AZ Strictness

A subnet **cannot span AZs**.

### ✔ Trick 5: NAT Gateway is created in **Public Subnet**

Because it needs internet access.

---

# 🔐 **3. Security Group vs NACL – Easy Table**

| Feature        | Security Group | NACL                   |
| -------------- | -------------- | ---------------------- |
| Stateful       | ✅ Yes          | ❌ No                   |
| Applied on     | ENI/Instance   | Subnet                 |
| Default Rule   | Deny all       | Allow all              |
| Supports deny? | ❌ No           | ✅ Yes                  |
| Rule order     | No order       | Evaluated top → bottom |

---

# ⚙️ **4. Important AWS VPC Commands (AWS CLI)**

## **Create VPC**

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

## **Create Subnet**

```bash
aws ec2 create-subnet \
  --vpc-id vpc-123456 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone ap-south-1a
```

## **Create Internet Gateway**

```bash
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-123 --vpc-id vpc-123
```

## **Create Route Table & Routes**

```bash
aws ec2 create-route-table --vpc-id vpc-123
aws ec2 create-route \
  --route-table-id rtb-123 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-123
```

## **Associate Route Table**

```bash
aws ec2 associate-route-table \
  --subnet-id subnet-123 \
  --route-table-id rtb-123
```

## **Create NAT Gateway**

```bash
aws ec2 allocate-address
aws ec2 create-nat-gateway \
  --subnet-id subnet-public \
  --allocation-id eipalloc-123
```

## **Create Security Group**

```bash
aws ec2 create-security-group \
  --group-name web-sg \
  --description "web access" \
  --vpc-id vpc-123
```

## **Add SG Rules**

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-123 --protocol tcp --port 22 --cidr 0.0.0.0/0
```

---

# 🎯 **5. Exam & Interview Winning Points**

### 🔸 **Public Subnet = Route to IGW**

Most common Viva + Exam question.

### 🔸 **Private Subnet = Route to NAT**

Instance inside private subnet cannot ping internet without NAT.

### 🔸 **SG is Stateful**

If inbound is allowed, outbound automatically allowed.

### 🔸 **VPC Peering is NOT transitive**

A→B works
B→C works
A→C does **NOT** work.

### 🔸 **You cannot overlap CIDRs**

Peering will fail if CIDR overlaps.

### 🔸 **VPC Flow Logs show:**

* ACCEPT / REJECT
* Source IP
* Destination IP
* Protocol
* Action

### 🔸 **Instance in Public Subnet needs 3 things:**

1. Public IP
2. IGW attached
3. Route to IGW

### 🔸 **NAT Gateway is per-AZ**

High availability = place NAT in **each AZ**.

---

# 🧭 **6. VPC Components Summary (One-Liners)**

* **VPC** → Full network
* **Subnet** → Partition of VPC
* **IGW** → Internet access
* **NAT GW** → Private → Internet
* **SG** → Instance firewall
* **NACL** → Subnet firewall
* **Route Table** → Routing rules
* **VPN Gateway** → On-prem ↔ AWS
* **Transit GW** → Large-scale multi-VPC
* **VPC Endpoint** → Private AWS service access
* **DHCP Option Set** → DNS, NTP settings
* **Elastic IP** → Static Public IP

---

# 🧩 **7. Quick VPC Diagram (Logical)**

```
VPC (10.0.0.0/16)
 ├── Public Subnet (10.0.1.0/24)
 │     ├── EC2 (Public IP)
 │     └── NAT Gateway
 │
 └── Private Subnet (10.0.2.0/24)
       └── EC2 (No Public IP)
```

---

# 🔢 **VPC IP Formula (CIDR → Number of Hosts)**

### ✅ **Formula**

```
Number of IPs = 2^(32 – CIDR)
```

### 🧮 **Usable IPs**

```
Usable IPs = Total IPs – 5
```

AWS reserves **5 IPs per subnet**:

* Network address
* VPC router
* DNS
* AWS Reserved
* Broadcast (not used but reserved)

---

# 📝 **Examples**

### ⭐ **1. CIDR: /16**

```
Total IPs = 2^(32–16)
          = 2^16
          = 65,536
Usable = 65,536 – 5 = 65,531
```

---

### ⭐ **2. CIDR: /20**

```
Total IPs = 2^(32–20)
          = 2^12
          = 4096
Usable = 4096 – 5 = 4091
```

---

### ⭐ **3. CIDR: /24**

```
Total IPs = 2^(32–24)
          = 2^8
          = 256
Usable = 256 – 5 = 251
```

---

# 📘 **Subnet Formula (General)**

### **Network Size**

```
Block Size = 256 – Subnet Mask Value
```

Example:
Subnet Mask = 255.255.255.0 → Last octet = 0
Block Size = 256 – 0 = 256

For /26 → mask 255.255.255.192
192 in last octet → Block Size = 256 – 192 = 64

---

# 📦 Quick Table

| CIDR | Total IPs | Usable IPs |
| ---- | --------- | ---------- |
| /16  | 65,536    | 65,531     |
| /20  | 4,096     | 4,091      |
| /22  | 1,024     | 1,019      |
| /24  | 256       | 251        |
| /26  | 64        | 59         |
| /28  | 16        | 11         |

---


# AWS VPC Complete Course

Welcome to the comprehensive AWS Virtual Private Cloud (VPC) course! This course will take you from beginner to advanced levels in understanding and implementing AWS VPC networking.

## 📚 Course Overview

Amazon Virtual Private Cloud (VPC) is a fundamental service that allows you to provision a logically isolated section of the AWS cloud where you can launch AWS resources in a virtual network that you define. This course covers everything from basic concepts to advanced networking scenarios.

## 🎯 Learning Objectives

By completing this course, you will be able to:
- Understand VPC architecture and core components
- Design and implement custom VPCs from scratch
- Configure routing, security, and connectivity
- Implement hybrid cloud networking solutions
- Apply AWS VPC best practices and security standards
- Troubleshoot common VPC networking issues

## 👥 Target Audience

- Cloud Engineers and Architects
- DevOps Engineers
- Network Engineers transitioning to cloud
- AWS Solutions Architects preparing for certification
- Anyone wanting to master AWS networking

## 📋 Prerequisites

- Basic understanding of networking concepts (IP addressing, subnets, routing)
- AWS account (Free tier is sufficient for most labs)
- AWS CLI installed and configured (optional but recommended)
- Basic familiarity with AWS Console

## 🗂️ Course Structure

### Module 1: VPC Fundamentals
- Introduction to AWS VPC
- CIDR Blocks and IP Addressing
- Subnets (Public and Private)
- Availability Zones
- Default VPC vs Custom VPC

### Module 2: Routing and Internet Connectivity
- Route Tables
- Internet Gateway (IGW)
- NAT Gateway and NAT Instance
- Egress-Only Internet Gateway

### Module 3: VPC Security
- Security Groups
- Network Access Control Lists (NACLs)
- Security Group vs NACL Comparison
- Best Practices for VPC Security

### Module 4: VPC Connectivity
- VPC Peering
- AWS Transit Gateway
- VPN Connections
- AWS Direct Connect
- AWS PrivateLink

### Module 5: Advanced VPC Topics
- VPC Endpoints (Gateway and Interface)
- VPC Flow Logs
- DNS in VPC
- DHCP Options Sets
- IPv6 in VPC
- Network Firewall

### Module 6: Hands-On Labs
- Lab 1: Create a Basic VPC
- Lab 2: Build a Multi-Tier Architecture
- Lab 3: Implement VPC Peering
- Lab 4: Configure VPN Connection
- Lab 5: Setup VPC Endpoints
- Lab 6: Implement Network Security

## 🛠️ Tools and Resources

- AWS Management Console
- AWS CLI
- CloudFormation Templates (included)
- Terraform Scripts (included)
- Network Diagram Tools

## 📖 How to Use This Course

1. Start with Module 1 and progress sequentially
2. Read the theory in each module
3. Follow along with the hands-on labs
4. Review the diagrams and architecture patterns
5. Practice with the provided CloudFormation templates
6. Complete the exercises at the end of each module

## 💰 Cost Considerations

Most labs in this course can be completed within the AWS Free Tier. However:
- NAT Gateway incurs charges (~$0.045/hour)
- VPN Connections incur charges (~$0.05/hour)
- Data transfer charges may apply
- Always clean up resources after labs

## 📝 Certification Alignment

This course content aligns with:
- AWS Certified Solutions Architect - Associate
- AWS Certified Advanced Networking - Specialty
- AWS Certified SysOps Administrator - Associate

---

## 🌐 **AWS VPC Project — Secure Multi-Tier Architecture**

This project builds a **production-grade AWS VPC** with:

* Public & Private Subnets
* Internet Gateway
* NAT Gateway
* Route Tables
* Bastion Host
* Private EC2 Instance
* Security Groups
* Terraform & CloudFormation Automation
* AWS CLI Scripts

---

## 📌 **Architecture Overview**

### **VPC CIDR:** `10.0.0.0/16`

### **Subnets**

| Subnet           | CIDR        | Type    | AZ          |
| ---------------- | ----------- | ------- | ----------- |
| public-subnet-a  | 10.0.1.0/24 | Public  | ap-south-1a |
| public-subnet-b  | 10.0.2.0/24 | Public  | ap-south-1b |
| private-subnet-a | 10.0.3.0/24 | Private | ap-south-1a |
| private-subnet-b | 10.0.4.0/24 | Private | ap-south-1b |

---

# 📐 Mermaid Architecture Diagram

```
flowchart TD
    IGW --> RT_Public
    NAT --> RT_Private

    RT_Public --> PublicA
    RT_Public --> PublicB

    RT_Private --> PrivateA
    RT_Private --> PrivateB

    Bastion --> PublicA
    AppServer --> PrivateA

    Internet --> IGW
```

---

# 🔧 **Features**

* Highly available 2-AZ architecture
* NAT for private instances
* Locked-down security groups
* Works with Terraform & CloudFormation
* CLI scripts for one-click deployment

---

# 🚀 **PART 2 — Full Terraform Code**

(Includes providers, modules, main root code)

👉 I will send **all files** next:

* `provider.tf`
* `variables.tf`
* `main.tf`
* `outputs.tf`
* Modules: `vpc`, `subnets`, `nat-gw`, `igw`, `ec2`

---

# 📦 **PART 3 — Full CloudFormation Stack (YAML)**

Complete `vpc-stack.yaml` + `parameters.json`

---

# 🖥️ **PART 4 — AWS CLI Shell Scripts**

* Create VPC automatically
* Launch Bastion & Private EC2
* Test NAT, SSH, Security Groups

---

# 📁 **PART 5 — ZIP Project Download**

I will generate a full zip file using Python (python_user_visible tool).

---

# 🗺️ **PART 6 — Architecture Diagrams**

* ASCII
* Mermaid
* PNG (via vector)

---


## 🤝 Contributing

Found an error or want to improve the course? Contributions are welcome!

## 📄 License

This course material is provided for educational purposes.

## 🚀 Let's Get Started!

Begin your journey with [Module 1: VPC Fundamentals](./modules/01-vpc-fundamentals/README.md)

---

