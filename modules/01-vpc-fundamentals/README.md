# Module 1: VPC Fundamentals

## Introduction to AWS VPC

Amazon Virtual Private Cloud (VPC) is a virtual network dedicated to your AWS account. It is logically isolated from other virtual networks in the AWS cloud, giving you complete control over your virtual networking environment.

### What is a VPC?

A VPC is your own isolated section of the AWS cloud where you can:
- Define your own IP address range
- Create subnets
- Configure route tables
- Set up network gateways
- Control security settings

Think of it as your own private data center in the cloud, but with the flexibility and scalability of AWS.

## Key VPC Concepts

### 1. CIDR Blocks (Classless Inter-Domain Routing)

CIDR notation is used to define IP address ranges for your VPC.

#### CIDR Format
```
IP Address/Prefix Length
Example: 10.0.0.0/16
```

#### Understanding CIDR Notation

- `/16` = 65,536 IP addresses (recommended for large VPCs)
- `/24` = 256 IP addresses (common for subnets)
- `/28` = 16 IP addresses (small subnets)

#### CIDR Calculation Examples

| CIDR Block | IP Range | Usable IPs | Use Case |
|------------|----------|------------|----------|
| 10.0.0.0/16 | 10.0.0.0 - 10.0.255.255 | 65,536 | Large VPC |
| 10.0.1.0/24 | 10.0.1.0 - 10.0.1.255 | 256 | Subnet |
| 10.0.1.0/28 | 10.0.1.0 - 10.0.1.15 | 16 | Small subnet |

#### VPC CIDR Range Restrictions

AWS VPC CIDR blocks must be:
- Between /16 (65,536 IPs) and /28 (16 IPs)
- From private IP ranges (RFC 1918):
  - 10.0.0.0/8 (10.0.0.0 to 10.255.255.255)
  - 172.16.0.0/12 (172.16.0.0 to 172.31.255.255)
  - 192.168.0.0/16 (192.168.0.0 to 192.168.255.255)

### 2. IP Addressing in VPC

#### Private IP Addresses
- Automatically assigned to instances from subnet CIDR
- Used for communication within VPC
- Retained when instance is stopped (for primary ENI)

#### Public IP Addresses
- Assigned from AWS pool
- Lost when instance stops
- Used for internet communication
- Cannot be moved between instances

#### Elastic IP Addresses
- Static public IPv4 address
- Can be associated/disassociated
- Retained even when instance stops
- Charged when not associated with running instance

#### Reserved IP Addresses in Subnets

AWS reserves 5 IP addresses in each subnet:

For subnet `10.0.1.0/24`:
1. `10.0.1.0` - Network address
2. `10.0.1.1` - VPC router
3. `10.0.1.2` - DNS server
4. `10.0.1.3` - Reserved for future use
5. `10.0.1.255` - Network broadcast

**Usable IPs: 251 out of 256**

### 3. Subnets

Subnets are subdivisions of your VPC IP address range.

#### Public Subnet
- Has a route to the Internet Gateway
- Resources can communicate with the internet
- Typically hosts:
  - Load balancers
  - Bastion hosts
  - NAT Gateways

#### Private Subnet
- No direct route to Internet Gateway
- Resources cannot directly access internet
- Can access internet through NAT Gateway/Instance
- Typically hosts:
  - Application servers
  - Databases
  - Backend services

#### Subnet Best Practices

1. **Use multiple subnets across Availability Zones** for high availability
2. **Plan CIDR ranges** to avoid overlap and allow for growth
3. **Separate public and private resources**
4. **Leave room for expansion** - don't use all address space immediately

### 4. Availability Zones

- Each AWS Region has multiple Availability Zones (AZs)
- AZs are physically separate data centers
- Connected with low-latency networking
- Deploy subnets in multiple AZs for fault tolerance

#### Example Multi-AZ Architecture

```
VPC: 10.0.0.0/16

Availability Zone 1 (us-east-1a):
  - Public Subnet: 10.0.1.0/24
  - Private Subnet: 10.0.2.0/24

Availability Zone 2 (us-east-1b):
  - Public Subnet: 10.0.3.0/24
  - Private Subnet: 10.0.4.0/24
```

## Default VPC vs Custom VPC

### Default VPC

Every AWS account comes with a default VPC in each region:

**Characteristics:**
- CIDR block: 172.31.0.0/16
- Default subnet in each AZ (/20 subnets)
- Internet Gateway attached
- Default security group
- Default NACL (allows all traffic)
- Public DNS hostnames enabled

**Use Cases:**
- Quick testing
- Learning AWS
- Simple applications

**Limitations:**
- Fixed CIDR range
- Less control over networking
- Not suitable for production

### Custom VPC

A VPC you create with your own specifications:

**Advantages:**
- Choose your own CIDR range
- Complete control over subnets
- Custom routing
- Enhanced security
- Better for production workloads
- Can create multiple VPCs for isolation

**When to Use:**
- Production environments
- Complex networking requirements
- Compliance requirements
- Multi-tier applications

## VPC Components Overview

### Core Components

1. **VPC** - Virtual network
2. **Subnets** - IP address ranges within VPC
3. **Route Tables** - Rules for routing traffic
4. **Internet Gateway** - Gateway to internet
5. **NAT Gateway/Instance** - Allows private subnet internet access
6. **Network ACLs** - Stateless firewall at subnet level
7. **Security Groups** - Stateful firewall at instance level

### Optional Components

1. **VPC Peering** - Connect VPCs together
2. **VPN Gateway** - Connect to on-premises network
3. **Transit Gateway** - Hub for connecting VPCs
4. **VPC Endpoints** - Private connection to AWS services
5. **Elastic IPs** - Static public IP addresses
6. **NAT Instance** - Alternative to NAT Gateway

## VPC Limits and Quotas

| Resource | Default Limit | Adjustable |
|----------|--------------|------------|
| VPCs per Region | 5 | Yes |
| Subnets per VPC | 200 | Yes |
| IPv4 CIDR blocks per VPC | 5 | Yes (up to 50) |
| Route tables per VPC | 200 | Yes |
| Routes per route table | 50 | Yes |
| Security groups per VPC | 2,500 | Yes |
| Rules per security group | 60 | Yes |
| Network ACLs per VPC | 200 | Yes |

## VPC Architecture Patterns

### Pattern 1: Single-Tier (Simple)
```
┌─────────────────────────────────┐
│ VPC: 10.0.0.0/16               │
│  ┌───────────────────────────┐ │
│  │ Public Subnet: 10.0.1.0/24│ │
│  │  - EC2 Instances          │ │
│  │  - Has Internet Access    │ │
│  └───────────────────────────┘ │
│           │                     │
│    ┌──────▼──────┐             │
│    │   IGW       │             │
│    └─────────────┘             │
└─────────────────────────────────┘
```

### Pattern 2: Two-Tier (Web + Database)
```
┌──────────────────────────────────────────┐
│ VPC: 10.0.0.0/16                        │
│  ┌────────────────────────────────────┐ │
│  │ Public Subnet: 10.0.1.0/24         │ │
│  │  - Web Servers                     │ │
│  └────────────────────────────────────┘ │
│              │                           │
│  ┌───────────▼──────────────────────┐  │
│  │ Private Subnet: 10.0.2.0/24      │  │
│  │  - Database Servers              │  │
│  │  - No direct internet access     │  │
│  └──────────────────────────────────┘  │
│           │                              │
│    ┌──────▼──────┐                      │
│    │   IGW       │                      │
│    └─────────────┘                      │
└──────────────────────────────────────────┘
```

### Pattern 3: Three-Tier (Web + App + Database)
```
┌─────────────────────────────────────────────────┐
│ VPC: 10.0.0.0/16                               │
│  ┌──────────────────────────────────────────┐  │
│  │ Public Subnet: 10.0.1.0/24               │  │
│  │  - Load Balancer                         │  │
│  │  - Bastion Host                          │  │
│  └──────────────────────────────────────────┘  │
│              │                                  │
│  ┌───────────▼────────────────────────────┐   │
│  │ Private Subnet 1: 10.0.2.0/24          │   │
│  │  - Application Servers                 │   │
│  └────────────────────────────────────────┘   │
│              │                                  │
│  ┌───────────▼────────────────────────────┐   │
│  │ Private Subnet 2: 10.0.3.0/24          │   │
│  │  - Database Servers                    │   │
│  └────────────────────────────────────────┘   │
│           │                                     │
│    ┌──────▼──────┐                            │
│    │   IGW/NAT   │                            │
│    └─────────────┘                            │
└─────────────────────────────────────────────────┘
```

## Hands-On Exercise 1: Explore Default VPC

### Objectives
- View default VPC components
- Understand default configuration
- Identify resources

### Steps

1. **Open AWS Console**
   - Navigate to VPC Dashboard

2. **View Default VPC**
   - Look for VPC marked as "default"
   - Note the CIDR block (172.31.0.0/16)

3. **Examine Subnets**
   - Click on "Subnets"
   - See one subnet per Availability Zone
   - Note each subnet has /20 CIDR

4. **Check Route Table**
   - View main route table
   - Observe route to Internet Gateway

5. **View Internet Gateway**
   - See IGW attached to default VPC

## Hands-On Exercise 2: Plan a Custom VPC

### Scenario
Design a VPC for a web application with the following requirements:
- Web tier (public)
- Application tier (private)
- Database tier (private)
- High availability (2 AZs)

### Task: Plan Your CIDR Blocks

```
VPC CIDR: __________________ (/16 recommended)

Availability Zone 1:
  Public Subnet: __________________
  App Private Subnet: __________________
  DB Private Subnet: __________________

Availability Zone 2:
  Public Subnet: __________________
  App Private Subnet: __________________
  DB Private Subnet: __________________

Reserved for future growth: __________________
```

### Sample Solution

```
VPC CIDR: 10.0.0.0/16 (65,536 IPs)

Availability Zone 1 (us-east-1a):
  Public Subnet: 10.0.1.0/24 (256 IPs)
  App Private Subnet: 10.0.2.0/24 (256 IPs)
  DB Private Subnet: 10.0.3.0/24 (256 IPs)

Availability Zone 2 (us-east-1b):
  Public Subnet: 10.0.11.0/24 (256 IPs)
  App Private Subnet: 10.0.12.0/24 (256 IPs)
  DB Private Subnet: 10.0.13.0/24 (256 IPs)

Reserved: 10.0.20.0/22 and beyond for future use
```

## Key Takeaways

✅ VPC provides isolated networking in AWS  
✅ CIDR blocks define IP address ranges  
✅ Subnets divide VPC into smaller segments  
✅ Public subnets have internet access via IGW  
✅ Private subnets provide isolation for backend resources  
✅ Use multiple AZs for high availability  
✅ Plan CIDR ranges carefully to avoid overlap  
✅ Default VPC is good for learning, custom VPC for production  

## Quiz

1. What is the maximum and minimum size of a VPC CIDR block?
2. How many IP addresses are reserved in a /24 subnet?
3. What is the difference between a public and private subnet?
4. Why should you deploy subnets across multiple Availability Zones?
5. What CIDR range would you use for a VPC needing 10,000 IP addresses?

## Next Steps

Continue to [Module 2: Routing and Internet Connectivity](../02-routing-internet-gateway/README.md) to learn about:
- Route tables
- Internet Gateways
- NAT Gateways
- Routing strategies

## Additional Resources

- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [VPC CIDR Calculator](https://www.ipaddressguide.com/cidr)
- [AWS VPC Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
