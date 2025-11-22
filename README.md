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

