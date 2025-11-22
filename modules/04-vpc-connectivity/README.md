# Module 4: VPC Connectivity

## Introduction to VPC Connectivity

AWS provides multiple options for connecting VPCs to each other, to on-premises networks, and to other AWS services. This module covers the various connectivity patterns and their use cases.

## VPC Peering

### What is VPC Peering?

VPC Peering is a networking connection between two VPCs that enables routing traffic between them using private IP addresses.

### Key Characteristics

✅ **Private Connection** - Traffic stays on AWS network  
✅ **No Single Point of Failure** - Highly available  
✅ **No Bandwidth Bottleneck** - No aggregate bandwidth limit  
✅ **Cross Region** - Can peer VPCs in different regions  
✅ **Cross Account** - Can peer VPCs in different AWS accounts  

### VPC Peering Architecture

```
┌─────────────────────────┐         ┌─────────────────────────┐
│ VPC A: 10.0.0.0/16     │         │ VPC B: 10.1.0.0/16     │
│                         │         │                         │
│  ┌──────────────────┐  │  Peer   │  ┌──────────────────┐  │
│  │ App Servers      │◄─┼─────────┼─►│ Database         │  │
│  │ 10.0.1.0/24      │  │Connection│  │ 10.1.1.0/24      │  │
│  └──────────────────┘  │         │  └──────────────────┘  │
│                         │         │                         │
│ Route: 10.1.0.0/16 →   │         │ Route: 10.0.0.0/16 →   │
│        pcx-12345       │         │        pcx-12345       │
└─────────────────────────┘         └─────────────────────────┘
```

### VPC Peering Limitations

❌ **No Transitive Peering** - Cannot route through a peered VPC to reach another VPC  
❌ **No Overlapping CIDR** - VPCs must have unique CIDR blocks  
❌ **No Edge-to-Edge Routing** - Cannot access IGW, NAT, VPN through peer  
❌ **Single Peering Between VPCs** - Only one peering connection allowed between two VPCs  

### Transitive Peering Example

```
❌ This doesn't work:

VPC A ←peer→ VPC B ←peer→ VPC C

Traffic from VPC A cannot reach VPC C through VPC B
You need: VPC A ←peer→ VPC C
```

### Use Cases for VPC Peering

1. **Shared Services VPC**
   - Central VPC with shared resources (Active Directory, monitoring)
   - Peer with application VPCs

2. **Multi-Account Strategy**
   - Separate accounts for different environments
   - Peer VPCs across accounts

3. **Cross-Region Replication**
   - Data replication between regions
   - Disaster recovery setup

4. **Partner/Customer Access**
   - Secure access to partner VPCs
   - Without internet exposure

### VPC Peering Best Practices

1. **Plan CIDR Ranges** - Avoid overlapping IP ranges
2. **Document Connections** - Maintain inventory of peering connections
3. **Least Privilege Routing** - Only add specific routes needed
4. **Security Groups** - Use security group references across peers
5. **Monitor Costs** - Data transfer charges apply (especially cross-region)

### VPC Peering Pricing

| Type | Cost |
|------|------|
| Same Region | $0.01/GB (both directions) |
| Cross Region | $0.02/GB (varies by region pair) |
| Same AZ | Free (if using private IP) |

## AWS Transit Gateway

### What is Transit Gateway?

Transit Gateway is a network transit hub that connects VPCs and on-premises networks through a central hub.

### Architecture: Hub and Spoke Model

```
                    ┌─────────────────────┐
                    │  Transit Gateway    │
                    │    (Hub)           │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
    ┌──────▼──────┐     ┌──────▼──────┐    ┌──────▼──────┐
    │   VPC A     │     │   VPC B     │    │   VPC C     │
    │ 10.0.0.0/16 │     │ 10.1.0.0/16 │    │ 10.2.0.0/16 │
    └─────────────┘     └─────────────┘    └─────────────┘
           │
    ┌──────▼──────┐
    │ On-Premises │
    │ via VPN/DX  │
    └─────────────┘
```

### Transit Gateway vs VPC Peering

| Feature | VPC Peering | Transit Gateway |
|---------|-------------|-----------------|
| **Architecture** | Point-to-point | Hub and spoke |
| **Transitive Routing** | No | Yes |
| **Max Connections** | Limited by peering limits | Thousands |
| **Centralized Management** | No | Yes |
| **On-Premises Connection** | No | Yes (VPN/Direct Connect) |
| **Cost** | Lower | Higher |
| **Bandwidth** | No limit | 50 Gbps per VPC attachment |

### When to Use Transit Gateway

✅ **Many VPCs** - More than 5-10 VPCs to connect  
✅ **Transitive Routing** - Need to route through central hub  
✅ **Centralized Management** - Single point of control  
✅ **Hybrid Cloud** - Connecting to on-premises networks  
✅ **Complex Routing** - Multiple route tables and segmentation  

### When to Use VPC Peering

✅ **Simple Connectivity** - Few VPCs (2-5)  
✅ **Cost Sensitive** - Lower data transfer costs  
✅ **No Transitive Routing** - Direct point-to-point is sufficient  
✅ **Maximum Bandwidth** - No bandwidth constraints  

### Transit Gateway Features

1. **Route Tables** - Multiple route tables for traffic segmentation
2. **Multi-Region Peering** - Connect Transit Gateways across regions
3. **Multicast Support** - Distribute multicast traffic
4. **Network Manager** - Global view of network
5. **VPN ECMP** - Equal-cost multi-path routing for VPN

### Transit Gateway Pricing

| Component | Cost |
|-----------|------|
| Attachment per hour | $0.05/hour |
| Data processing | $0.02/GB |

**Example Monthly Cost:**
- 5 VPC attachments: 5 × $0.05 × 730 hours = **$182.50**
- 1TB data transfer: 1024GB × $0.02 = **$20.48**
- **Total: ~$203/month**

## VPN Connections

### AWS Site-to-Site VPN

Connects your on-premises network to AWS VPC over the internet using IPsec VPN tunnels.

### VPN Architecture

```
┌─────────────────────────────────────────────┐
│ On-Premises Data Center                     │
│  ┌──────────────────────────────────┐      │
│  │ Customer Gateway Device          │      │
│  │ (Physical or Software VPN)       │      │
│  └──────────────┬───────────────────┘      │
└─────────────────┼────────────────────────────┘
                  │ IPsec VPN Tunnel
                  │ (Over Internet)
┌─────────────────▼────────────────────────────┐
│ AWS Cloud                                    │
│  ┌──────────────────────────────────┐       │
│  │ Virtual Private Gateway (VGW)    │       │
│  │ (VPN Endpoint)                   │       │
│  └──────────────┬───────────────────┘       │
│                 │                            │
│  ┌──────────────▼───────────────────┐       │
│  │ VPC: 10.0.0.0/16                │       │
│  │   - Private Subnets              │       │
│  │   - Application Servers          │       │
│  └──────────────────────────────────┘       │
└──────────────────────────────────────────────┘
```

### VPN Components

1. **Virtual Private Gateway (VGW)**
   - AWS side of VPN connection
   - Attached to VPC

2. **Customer Gateway (CGW)**
   - Physical or software device on customer side
   - Must support IPsec

3. **VPN Connection**
   - Two IPsec tunnels (for redundancy)
   - Encrypted connection

### VPN Features

✅ **Redundant Tunnels** - Two tunnels for high availability  
✅ **Encrypted** - IPsec encryption for security  
✅ **Flexible** - Works with various customer gateway devices  
✅ **Quick Setup** - Can be configured in minutes  

### VPN Limitations

❌ **Internet Dependent** - Performance depends on internet connection  
❌ **Variable Latency** - Not suitable for latency-sensitive applications  
❌ **Bandwidth Limited** - Up to 1.25 Gbps per tunnel  
❌ **Shared Infrastructure** - Uses public internet  

### VPN Pricing

| Component | Cost |
|-----------|------|
| VPN Connection | $0.05/hour per connection |
| Data Transfer Out | Standard AWS data transfer rates |

**Example:** 1 VPN connection running 24/7 = $0.05 × 730 hours = **$36.50/month**

### AWS Client VPN

Client VPN enables remote users to connect to AWS or on-premises networks.

**Use Cases:**
- Remote employee access
- Mobile workforce
- Secure access without VPN hardware

## AWS Direct Connect

### What is Direct Connect?

Direct Connect is a dedicated network connection from your premises to AWS, bypassing the public internet.

### Direct Connect Architecture

```
┌─────────────────────────────────────────────┐
│ On-Premises Data Center                     │
│  ┌──────────────────────────────────┐      │
│  │ Customer Router                  │      │
│  └──────────────┬───────────────────┘      │
└─────────────────┼────────────────────────────┘
                  │ Dedicated Fiber
                  │ (Private Connection)
┌─────────────────▼────────────────────────────┐
│ AWS Direct Connect Location                  │
│  ┌──────────────────────────────────┐       │
│  │ AWS Direct Connect Endpoint      │       │
│  └──────────────┬───────────────────┘       │
└─────────────────┼────────────────────────────┘
                  │
┌─────────────────▼────────────────────────────┐
│ AWS Cloud                                    │
│  ┌──────────────────────────────────┐       │
│  │ Virtual Private Gateway (VGW)    │       │
│  └──────────────┬───────────────────┘       │
│                 │                            │
│  ┌──────────────▼───────────────────┐       │
│  │ VPC: 10.0.0.0/16                │       │
│  └──────────────────────────────────┘       │
└──────────────────────────────────────────────┘
```

### Direct Connect Benefits

✅ **Consistent Performance** - Dedicated bandwidth (1 Gbps, 10 Gbps, 100 Gbps)  
✅ **Lower Latency** - Direct connection to AWS  
✅ **Cost Effective** - Lower data transfer costs for high volume  
✅ **Private Connection** - Doesn't traverse public internet  
✅ **Hybrid Cloud** - Ideal for hybrid architectures  

### Direct Connect vs VPN

| Feature | Direct Connect | VPN |
|---------|---------------|-----|
| **Connection** | Dedicated fiber | Internet-based |
| **Bandwidth** | 1-100 Gbps | Up to 1.25 Gbps/tunnel |
| **Latency** | Low, consistent | Variable |
| **Setup Time** | Weeks to months | Minutes |
| **Cost** | Higher (port + data transfer) | Lower |
| **Use Case** | High throughput, consistent | Quick setup, lower volume |

### Direct Connect with VPN (Maximum Security)

Combine Direct Connect with VPN for encrypted connection over dedicated line:

```
On-Premises → Direct Connect → VPN over Direct Connect → VPC
```

**Benefits:**
- Dedicated bandwidth of Direct Connect
- Encryption security of VPN
- Best of both worlds

### Direct Connect Pricing

| Component | Cost (varies by location) |
|-----------|--------------------------|
| Port Hours (1 Gbps) | ~$0.30/hour ($219/month) |
| Port Hours (10 Gbps) | ~$2.25/hour ($1,642/month) |
| Data Transfer Out (Reduced) | Lower than standard rates |

## AWS PrivateLink

### What is PrivateLink?

PrivateLink provides private connectivity between VPCs and AWS services or customer-hosted services without using public IPs.

### PrivateLink Architecture

```
┌─────────────────────────────────────────────┐
│ Service Provider VPC                         │
│  ┌──────────────────────────────────┐       │
│  │ Application Load Balancer        │       │
│  └──────────────┬───────────────────┘       │
│                 │                            │
│  ┌──────────────▼───────────────────┐       │
│  │ VPC Endpoint Service             │       │
│  │ (Powered by PrivateLink)         │       │
│  └──────────────┬───────────────────┘       │
└─────────────────┼────────────────────────────┘
                  │ AWS PrivateLink
┌─────────────────▼────────────────────────────┐
│ Consumer VPC                                 │
│  ┌──────────────────────────────────┐       │
│  │ Interface VPC Endpoint           │       │
│  │ (ENI with Private IP)            │       │
│  └──────────────┬───────────────────┘       │
│                 │                            │
│  ┌──────────────▼───────────────────┐       │
│  │ Application Instances            │       │
│  └──────────────────────────────────┘       │
└──────────────────────────────────────────────┘
```

### PrivateLink Use Cases

1. **SaaS Applications**
   - Expose your SaaS to customers privately
   - Customers connect via VPC Endpoint

2. **Shared Services**
   - Centralized services across accounts
   - Private access without VPC peering

3. **AWS Services**
   - Private access to AWS services (S3, DynamoDB, etc.)
   - No internet gateway needed

4. **Third-Party Services**
   - Access partner services privately
   - Available in AWS Marketplace

### PrivateLink vs VPC Peering

| Feature | PrivateLink | VPC Peering |
|---------|------------|-------------|
| **Access Pattern** | Service-specific | Full network access |
| **Overlapping IPs** | Supported | Not supported |
| **Scalability** | Highly scalable | Limited by peering connections |
| **Security** | Service-level | Network-level |

## Connectivity Decision Tree

```
Need to connect to AWS?
│
├─ Few VPCs (2-5), simple connectivity?
│  └─ Use: VPC Peering ✓
│
├─ Many VPCs (10+), complex routing?
│  └─ Use: Transit Gateway ✓
│
├─ On-premises to AWS?
│  ├─ Quick setup, moderate bandwidth?
│  │  └─ Use: Site-to-Site VPN ✓
│  │
│  └─ High bandwidth, consistent performance?
│     └─ Use: Direct Connect ✓
│
├─ Remote users need access?
│  └─ Use: Client VPN ✓
│
└─ Exposing service to many consumers?
   └─ Use: PrivateLink ✓
```

## Hybrid Cloud Architecture Patterns

### Pattern 1: Simple VPN

```
On-Premises ←VPN→ VPC
```

**Use Case:** Small-scale, dev/test, backup

### Pattern 2: Direct Connect with VPN Backup

```
On-Premises ←Direct Connect→ VPC
            ←VPN (Backup)→
```

**Use Case:** Production, need reliability

### Pattern 3: Hub and Spoke with Transit Gateway

```
                Transit Gateway
                       │
        ┌──────────────┼──────────────┐
        │              │              │
    VPC Prod      VPC Dev      On-Premises
                                (via VPN/DX)
```

**Use Case:** Multiple VPCs, centralized management

### Pattern 4: Multi-Region

```
Region 1                    Region 2
   │                           │
Transit GW 1 ←──Peering──→ Transit GW 2
   │                           │
VPCs + On-Prem            VPCs + On-Prem
```

**Use Case:** Disaster recovery, global applications

## Hands-On Lab 1: Create VPC Peering

### Objective
Peer two VPCs and enable communication between them.

### Steps

#### 1. Create VPC Peering Connection
```bash
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-1a2b3c4d \
  --peer-vpc-id vpc-5e6f7g8h \
  --peer-region us-west-2
```

#### 2. Accept Peering Connection
```bash
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-12345678
```

#### 3. Add Routes to Route Tables

**VPC A Route Table:**
```bash
aws ec2 create-route \
  --route-table-id rtb-1a2b3c4d \
  --destination-cidr-block 10.1.0.0/16 \
  --vpc-peering-connection-id pcx-12345678
```

**VPC B Route Table:**
```bash
aws ec2 create-route \
  --route-table-id rtb-5e6f7g8h \
  --destination-cidr-block 10.0.0.0/16 \
  --vpc-peering-connection-id pcx-12345678
```

#### 4. Update Security Groups

Allow traffic from peer VPC CIDR in security groups.

#### 5. Test Connectivity
```bash
# From instance in VPC A
ping <private-ip-in-vpc-b>
```

## Hands-On Lab 2: Setup Transit Gateway

### Objective
Create Transit Gateway and attach multiple VPCs.

### CloudFormation Template
See [lab2-transit-gateway.yaml](./cloudformation/lab2-transit-gateway.yaml)

## Monitoring and Troubleshooting

### VPC Peering Issues

**Connection Not Working:**
1. ✅ Check peering connection status (should be "active")
2. ✅ Verify routes exist in both VPCs
3. ✅ Check security groups allow traffic
4. ✅ Verify NACLs allow traffic
5. ✅ Ensure no CIDR overlap

### VPN Connection Issues

**Tunnel Down:**
1. ✅ Check customer gateway configuration
2. ✅ Verify BGP/static routes
3. ✅ Check firewall rules on-premises
4. ✅ Validate IPsec parameters match
5. ✅ Review CloudWatch metrics

**Poor Performance:**
1. ✅ Check internet connection quality
2. ✅ Enable VPN acceleration (if available)
3. ✅ Consider Direct Connect
4. ✅ Review MTU settings

### Transit Gateway Issues

**High Latency:**
1. ✅ Check attachment bandwidth limits
2. ✅ Review route propagation
3. ✅ Verify route table associations
4. ✅ Check for routing loops

## Cost Optimization

### VPC Connectivity Costs Comparison

**Scenario: 1TB/month data transfer**

| Solution | Monthly Cost | Notes |
|----------|-------------|-------|
| VPC Peering (same region) | $10 | Most cost-effective |
| VPC Peering (cross-region) | $20 | Regional pricing varies |
| Transit Gateway | $203+ | Attachment + data fees |
| VPN | $37+ | Connection + data transfer |
| Direct Connect (1Gbps) | $219+ | Port + reduced data rates |

### Tips to Reduce Costs

1. **Use VPC Peering** when possible (fewer VPCs)
2. **Leverage VPC Endpoints** for AWS services (avoids NAT Gateway costs)
3. **Consolidate traffic** through fewer connections
4. **Use same-region** connectivity when possible
5. **Right-size Direct Connect** ports (consider hosted connections)

## Key Takeaways

✅ VPC Peering for simple point-to-point connectivity  
✅ Transit Gateway for complex hub-and-spoke architectures  
✅ Site-to-Site VPN for quick, encrypted on-premises connectivity  
✅ Direct Connect for high-bandwidth, consistent performance  
✅ PrivateLink for exposing services privately  
✅ No transitive routing with VPC Peering  
✅ Plan CIDR ranges to avoid overlap  
✅ Consider costs when choosing connectivity option  

## Quiz

1. What is transitive peering and why doesn't VPC Peering support it?
2. When would you choose Transit Gateway over VPC Peering?
3. What are the two tunnels in a VPN connection for?
4. What's the main difference between Direct Connect and VPN?
5. How does PrivateLink differ from VPC Peering?
6. Can you peer VPCs across AWS accounts? Across regions?

## Next Steps

Continue to [Module 5: Advanced VPC Topics](../05-advanced-topics/README.md) to learn about:
- VPC Endpoints
- VPC Flow Logs
- DNS and DHCP Options
- IPv6 in VPC

## Additional Resources

- [VPC Peering Documentation](https://docs.aws.amazon.com/vpc/latest/peering/)
- [Transit Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/tgw/)
- [VPN Documentation](https://docs.aws.amazon.com/vpn/)
- [Direct Connect Documentation](https://docs.aws.amazon.com/directconnect/)
- [PrivateLink Documentation](https://docs.aws.amazon.com/vpc/latest/privatelink/)
