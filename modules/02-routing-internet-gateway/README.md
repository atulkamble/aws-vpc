# Module 2: Routing and Internet Connectivity

## Introduction to VPC Routing

Routing determines how network traffic flows within your VPC and to external networks. Understanding routing is crucial for designing effective VPC architectures.

## Route Tables

### What is a Route Table?

A route table contains a set of rules (routes) that determine where network traffic from your subnet or gateway is directed.

### Components of a Route

Each route consists of:
- **Destination**: The IP address range (CIDR block)
- **Target**: Where traffic should be sent

Example route:
```
Destination: 10.0.0.0/16    Target: local
Destination: 0.0.0.0/0      Target: igw-123456
```

### Types of Route Tables

#### 1. Main Route Table
- Automatically created with VPC
- Default route table for subnets not explicitly associated
- Can be modified but not deleted

#### 2. Custom Route Tables
- Created by users
- Can be associated with specific subnets
- Can be deleted if not associated with subnets

### Route Table Association

- Each subnet must be associated with exactly one route table
- Multiple subnets can share the same route table
- If not explicitly associated, subnet uses main route table

### Route Priority

When multiple routes match, AWS uses the **most specific route** (longest prefix match):

```
Routes in table:
1. 10.0.0.0/16    → local
2. 10.0.1.0/24    → vgw-123
3. 0.0.0.0/0      → igw-123

Traffic to 10.0.1.5:
→ Uses route #2 (most specific)

Traffic to 10.0.2.5:
→ Uses route #1

Traffic to 8.8.8.8:
→ Uses route #3
```

## Internet Gateway (IGW)

### What is an Internet Gateway?

An Internet Gateway is a horizontally scaled, redundant, and highly available VPC component that:
- Allows communication between VPC and the internet
- Provides a target for internet-routable traffic
- Performs NAT for instances with public IP addresses

### IGW Characteristics

✅ **Highly Available** - No availability risk or bandwidth constraints  
✅ **Scalable** - Automatically scales based on traffic  
✅ **No Cost** - Free to use (only pay for data transfer)  
✅ **One per VPC** - Only one IGW can be attached to a VPC  

### How Internet Gateway Works

```
┌─────────────────────────────────────────────┐
│ Internet                                    │
└─────────────────┬───────────────────────────┘
                  │
            ┌─────▼─────┐
            │    IGW    │
            └─────┬─────┘
                  │
┌─────────────────▼───────────────────────────┐
│ VPC: 10.0.0.0/16                           │
│  ┌───────────────────────────────────────┐ │
│  │ Public Subnet: 10.0.1.0/24            │ │
│  │  ┌──────────────────────────────┐    │ │
│  │  │ EC2 Instance                 │    │ │
│  │  │ Private IP: 10.0.1.10        │    │ │
│  │  │ Public IP: 54.123.45.67      │    │ │
│  │  └──────────────────────────────┘    │ │
│  └───────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

### Creating and Attaching IGW

**Steps:**
1. Create Internet Gateway
2. Attach to VPC (one IGW per VPC)
3. Add route to route table: `0.0.0.0/0 → igw-id`
4. Ensure instances have public IP or Elastic IP
5. Configure security groups to allow traffic

### Requirements for Internet Access

For an instance in a public subnet to access the internet:

1. ✅ Internet Gateway attached to VPC
2. ✅ Route to IGW in subnet's route table (`0.0.0.0/0 → igw-id`)
3. ✅ Public IP address or Elastic IP
4. ✅ Security group allows outbound traffic
5. ✅ Network ACL allows traffic

## NAT Gateway

### What is NAT Gateway?

NAT (Network Address Translation) Gateway allows instances in private subnets to access the internet while preventing inbound connections from the internet.

### Use Cases

- Software updates for private instances
- External API calls from private instances
- Outbound internet access without exposing instances
- Downloading packages, patches, and updates

### NAT Gateway Characteristics

✅ **Managed Service** - AWS manages the infrastructure  
✅ **High Availability** - Redundant within AZ  
✅ **Scalable** - Auto-scales up to 45 Gbps  
✅ **High Performance** - Better than NAT instances  
💰 **Cost** - $0.045/hour + $0.045/GB data processed  

### NAT Gateway Architecture

```
┌─────────────────────────────────────────────────────┐
│ Internet                                            │
└────────────────────┬────────────────────────────────┘
                     │
               ┌─────▼─────┐
               │    IGW    │
               └─────┬─────┘
                     │
┌────────────────────▼─────────────────────────────────┐
│ VPC: 10.0.0.0/16                                    │
│  ┌────────────────────────────────────────────────┐ │
│  │ Public Subnet: 10.0.1.0/24                     │ │
│  │        ┌──────────────┐                        │ │
│  │        │ NAT Gateway  │                        │ │
│  │        │ EIP: 1.2.3.4 │                        │ │
│  │        └──────┬───────┘                        │ │
│  └───────────────┼────────────────────────────────┘ │
│                  │                                   │
│  ┌───────────────▼────────────────────────────────┐ │
│  │ Private Subnet: 10.0.2.0/24                    │ │
│  │  ┌──────────────────────────────────┐         │ │
│  │  │ EC2 Instance                     │         │ │
│  │  │ Private IP: 10.0.2.10            │         │ │
│  │  │ No Public IP                     │         │ │
│  │  └──────────────────────────────────┘         │ │
│  └────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘

Route Table for Private Subnet:
  0.0.0.0/0 → nat-gateway-id
```

### NAT Gateway Best Practices

1. **Deploy in Public Subnet** - NAT Gateway needs internet access via IGW
2. **Associate Elastic IP** - NAT Gateway requires a static public IP
3. **One per AZ** - For high availability, create NAT Gateway in each AZ
4. **Update Route Tables** - Private subnet route table should point to NAT Gateway

### High Availability NAT Gateway Design

```
┌──────────────────────────────────────────────────────────┐
│ VPC: 10.0.0.0/16                                        │
│                                                          │
│  ┌─────────────────────┐    ┌─────────────────────┐   │
│  │ Availability Zone A │    │ Availability Zone B │   │
│  │                     │    │                     │   │
│  │  ┌──────────────┐  │    │  ┌──────────────┐  │   │
│  │  │Public Subnet │  │    │  │Public Subnet │  │   │
│  │  │ NAT-GW-A     │  │    │  │ NAT-GW-B     │  │   │
│  │  └──────┬───────┘  │    │  └──────┬───────┘  │   │
│  │         │          │    │         │          │   │
│  │  ┌──────▼───────┐  │    │  ┌──────▼───────┐  │   │
│  │  │Private Sub A │  │    │  │Private Sub B │  │   │
│  │  │Route:0.0.0.0/0│  │    │  │Route:0.0.0.0/0│  │   │
│  │  │→ NAT-GW-A    │  │    │  │→ NAT-GW-B    │  │   │
│  │  └──────────────┘  │    │  └──────────────┘  │   │
│  └─────────────────────┘    └─────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

### NAT Gateway Limitations

- Cannot send traffic over VPC peering or VPN connections
- Cannot be used by instances in same subnet
- Doesn't support port forwarding
- Doesn't support bastion server functionality
- Maximum of 55,000 simultaneous connections per unique destination

## NAT Instance

### What is a NAT Instance?

A NAT Instance is an EC2 instance configured to perform NAT, allowing private subnet instances to access the internet.

### NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|---------|-------------|--------------|
| Availability | Highly available within AZ | Manual failover required |
| Bandwidth | Up to 45 Gbps | Depends on instance type |
| Maintenance | Managed by AWS | Managed by you |
| Performance | Optimized for NAT | Variable |
| Cost | $0.045/hour + data | EC2 cost |
| Security Groups | Not supported | Supported |
| Bastion Server | No | Yes (can be used) |
| Port Forwarding | No | Yes |

### When to Use NAT Instance

- **Cost optimization** for very low traffic
- **Port forwarding** required
- **Bastion host** functionality needed
- **Security group** filtering required
- **Custom NAT** configuration needed

### NAT Instance Setup

**Requirements:**
1. Use NAT AMI (Amazon Linux NAT AMI)
2. Disable source/destination check
3. Launch in public subnet
4. Assign Elastic IP
5. Configure security group
6. Update private subnet route table

## Egress-Only Internet Gateway

### What is Egress-Only Internet Gateway?

An Egress-Only Internet Gateway is specifically for IPv6 traffic, allowing outbound IPv6 connections while preventing inbound connections.

### Use Case

- IPv6 instances in private subnets
- Need outbound internet access
- Block inbound connections

### Egress-Only IGW vs NAT Gateway

| Feature | NAT Gateway | Egress-Only IGW |
|---------|-------------|-----------------|
| IP Version | IPv4 | IPv6 |
| Cost | Hourly + data | Free |
| Address Translation | Yes | No (IPv6 doesn't need NAT) |

## Routing Scenarios

### Scenario 1: Public Subnet Routing

```
Route Table: rtb-public

Destination       Target          Purpose
10.0.0.0/16      local           VPC local traffic
0.0.0.0/0        igw-12345       Internet access
```

### Scenario 2: Private Subnet Routing

```
Route Table: rtb-private

Destination       Target          Purpose
10.0.0.0/16      local           VPC local traffic
0.0.0.0/0        nat-12345       Internet via NAT
```

### Scenario 3: VPN Connected Subnet

```
Route Table: rtb-hybrid

Destination       Target          Purpose
10.0.0.0/16      local           VPC local traffic
192.168.0.0/16   vgw-12345       On-premises network
0.0.0.0/0        nat-12345       Internet via NAT
```

### Scenario 4: Multiple Routes

```
Route Table: rtb-complex

Destination       Target          Purpose
10.0.0.0/16      local           VPC local traffic
172.16.0.0/12    pcx-12345       Peered VPC
192.168.0.0/16   vgw-12345       On-premises
0.0.0.0/0        igw-12345       Internet
```

## Route Table Best Practices

1. **Use descriptive names** - Tag route tables clearly
2. **Explicit associations** - Don't rely on main route table
3. **Minimal routes** - Only add necessary routes
4. **Document routes** - Use tags to explain purpose
5. **Separate public/private** - Different route tables for different subnet types
6. **Review regularly** - Audit routes periodically

## Troubleshooting Routing Issues

### Instance Cannot Access Internet

**Checklist:**
- ✅ Route to IGW or NAT exists (`0.0.0.0/0`)
- ✅ Subnet association correct
- ✅ IGW attached to VPC
- ✅ Instance has public IP (for IGW) or NAT Gateway exists
- ✅ Security group allows outbound traffic
- ✅ NACL allows traffic
- ✅ No overlapping routes

### Private Instance Cannot Reach Internet via NAT

**Checklist:**
- ✅ NAT Gateway in public subnet
- ✅ NAT Gateway has Elastic IP
- ✅ Public subnet has route to IGW
- ✅ Private subnet has route to NAT Gateway
- ✅ Security groups allow traffic
- ✅ NAT Gateway state is "available"

### Testing Connectivity

```bash
# Test internet connectivity
ping 8.8.8.8

# Test DNS resolution
nslookup google.com

# Trace route
traceroute google.com

# Check routes from instance
ip route show

# Test specific port
telnet google.com 443
```

## Hands-On Lab 1: Create VPC with Internet Gateway

### Objective
Create a VPC with a public subnet and Internet Gateway.

### Steps

#### 1. Create VPC
```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=MyVPC}]'
```

#### 2. Create Internet Gateway
```bash
aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=MyIGW}]'
```

#### 3. Attach IGW to VPC
```bash
aws ec2 attach-internet-gateway --vpc-id <vpc-id> --internet-gateway-id <igw-id>
```

#### 4. Create Public Subnet
```bash
aws ec2 create-subnet --vpc-id <vpc-id> --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
```

#### 5. Create Route Table
```bash
aws ec2 create-route-table --vpc-id <vpc-id>
```

#### 6. Add Route to IGW
```bash
aws ec2 create-route --route-table-id <rtb-id> --destination-cidr-block 0.0.0.0/0 --gateway-id <igw-id>
```

#### 7. Associate Route Table with Subnet
```bash
aws ec2 associate-route-table --subnet-id <subnet-id> --route-table-id <rtb-id>
```

#### 8. Enable Auto-Assign Public IP
```bash
aws ec2 modify-subnet-attribute --subnet-id <subnet-id> --map-public-ip-on-launch
```

## Hands-On Lab 2: Setup NAT Gateway

### Objective
Enable internet access for private subnet using NAT Gateway.

### Prerequisites
- Existing VPC with public and private subnets
- Internet Gateway attached

### Steps

#### 1. Allocate Elastic IP
```bash
aws ec2 allocate-address --domain vpc
```

#### 2. Create NAT Gateway in Public Subnet
```bash
aws ec2 create-nat-gateway --subnet-id <public-subnet-id> --allocation-id <eip-allocation-id>
```

Wait for NAT Gateway to become available (5-10 minutes):
```bash
aws ec2 describe-nat-gateways --nat-gateway-id <nat-gateway-id>
```

#### 3. Update Private Subnet Route Table
```bash
aws ec2 create-route --route-table-id <private-rtb-id> --destination-cidr-block 0.0.0.0/0 --nat-gateway-id <nat-gateway-id>
```

#### 4. Verify Connectivity
Launch an instance in private subnet and test internet access:
```bash
# From private instance
ping 8.8.8.8
curl http://checkip.amazonaws.com
```

## Hands-On Lab 3: Multi-AZ Architecture

### Objective
Create a highly available VPC with NAT Gateways in multiple AZs.

### Architecture
```
VPC: 10.0.0.0/16
- AZ1: Public (10.0.1.0/24), Private (10.0.2.0/24), NAT-GW-1
- AZ2: Public (10.0.3.0/24), Private (10.0.4.0/24), NAT-GW-2
```

### Implementation
See [lab3-multi-az-vpc.yaml](./cloudformation/lab3-multi-az-vpc.yaml) for CloudFormation template.

## Cost Optimization Tips

### NAT Gateway Costs

**Typical Costs:**
- NAT Gateway: $0.045/hour × 730 hours = **$32.85/month**
- Data Processing: $0.045/GB
- Example: 100GB/month = **$4.50**
- **Total: ~$37/month per NAT Gateway**

### Ways to Reduce Costs:

1. **Use NAT Instance for dev/test** (lower traffic environments)
2. **Consolidate traffic** through single NAT Gateway (non-production)
3. **Use VPC Endpoints** for AWS services (S3, DynamoDB)
4. **Implement caching** to reduce data transfer
5. **Schedule NAT Gateway** (delete during non-business hours for dev)

### Cost Comparison Example

| Solution | Monthly Cost | Availability | Use Case |
|----------|-------------|--------------|----------|
| NAT Gateway (1 AZ) | ~$37 | Medium | Dev/Test |
| NAT Gateway (2 AZs) | ~$74 | High | Production |
| NAT Instance (t3.micro) | ~$7 | Low | Low traffic |
| VPC Endpoint | $0.01/GB | High | AWS services only |

## Key Takeaways

✅ Route tables control traffic flow in VPC  
✅ Internet Gateway enables internet access for public subnets  
✅ NAT Gateway allows private instances to access internet  
✅ Most specific route wins (longest prefix match)  
✅ Each subnet needs explicit route table association  
✅ Deploy NAT Gateway per AZ for high availability  
✅ NAT Gateway incurs hourly and data processing charges  
✅ Egress-Only IGW is for IPv6 outbound traffic  

## Quiz

1. What is the difference between an Internet Gateway and a NAT Gateway?
2. How does route priority work when multiple routes match?
3. What components are required for an instance to access the internet?
4. Why should you create a NAT Gateway in each Availability Zone?
5. What is the purpose of the "local" route in a route table?
6. Can a NAT Gateway be in a private subnet?
7. What happens if you delete an Internet Gateway while instances are using it?

## Next Steps

Continue to [Module 3: VPC Security](../03-vpc-security/README.md) to learn about:
- Security Groups
- Network ACLs
- VPC security best practices
- Defense in depth strategies

## Additional Resources

- [VPC Route Tables Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [NAT Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Internet Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [AWS VPC Pricing](https://aws.amazon.com/vpc/pricing/)
