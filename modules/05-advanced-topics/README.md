# Module 5: Advanced VPC Topics

## Introduction

This module covers advanced VPC features that enhance functionality, security, and observability of your network infrastructure.

## VPC Endpoints

### What are VPC Endpoints?

VPC Endpoints enable private connections between your VPC and supported AWS services without requiring an Internet Gateway, NAT Gateway, VPN, or Direct Connect.

### Benefits of VPC Endpoints

✅ **Private Communication** - Traffic stays within AWS network  
✅ **No Internet Required** - No NAT Gateway costs  
✅ **Enhanced Security** - No exposure to public internet  
✅ **Lower Latency** - Direct connection to AWS services  
✅ **Cost Savings** - Eliminate NAT Gateway data processing fees  

### Types of VPC Endpoints

#### 1. Gateway Endpoints (Free)

Gateway Endpoints use route table entries to direct traffic to AWS services.

**Supported Services:**
- Amazon S3
- Amazon DynamoDB

**Architecture:**
```
┌─────────────────────────────────────────────┐
│ VPC: 10.0.0.0/16                           │
│                                             │
│  ┌─────────────────────────────────────┐  │
│  │ Private Subnet                       │  │
│  │  ┌───────────────────────────┐      │  │
│  │  │ EC2 Instance              │      │  │
│  │  │ Accesses S3 privately     │      │  │
│  │  └────────────┬──────────────┘      │  │
│  └───────────────┼──────────────────────┘  │
│                  │                          │
│         ┌────────▼────────┐                │
│         │ Route Table     │                │
│         │ s3-prefix → vpce │                │
│         └────────┬────────┘                │
│                  │                          │
│         ┌────────▼────────┐                │
│         │ Gateway Endpoint│                │
│         │ (vpce-xxx)      │                │
│         └────────┬────────┘                │
└──────────────────┼──────────────────────────┘
                   │
           ┌───────▼────────┐
           │ Amazon S3      │
           └────────────────┘
```

**Characteristics:**
- Free to use
- Specified as route target
- Scales automatically
- Supports resource policies

#### 2. Interface Endpoints (Powered by PrivateLink)

Interface Endpoints create an ENI (Elastic Network Interface) in your subnet with a private IP address.

**Supported Services (100+):**
- Amazon EC2
- Amazon SNS
- Amazon SQS
- AWS Systems Manager
- Amazon CloudWatch
- AWS Secrets Manager
- And many more...

**Architecture:**
```
┌─────────────────────────────────────────────┐
│ VPC: 10.0.0.0/16                           │
│                                             │
│  ┌─────────────────────────────────────┐  │
│  │ Private Subnet: 10.0.1.0/24         │  │
│  │                                      │  │
│  │  ┌───────────────────────────┐      │  │
│  │  │ EC2 Instance              │      │  │
│  │  │ 10.0.1.10                 │      │  │
│  │  └────────────┬──────────────┘      │  │
│  │               │                      │  │
│  │  ┌────────────▼──────────────┐      │  │
│  │  │ Interface Endpoint (ENI)  │      │  │
│  │  │ 10.0.1.50                 │      │  │
│  │  │ ec2.us-east-1.amazonaws..│      │  │
│  │  └────────────┬──────────────┘      │  │
│  └───────────────┼──────────────────────┘  │
└──────────────────┼──────────────────────────┘
                   │ AWS PrivateLink
           ┌───────▼────────┐
           │ AWS Service    │
           │ (EC2, SNS, etc)│
           └────────────────┘
```

**Characteristics:**
- $0.01/hour per AZ + $0.01/GB data processed
- Accessed via private DNS or ENI IP
- Can span multiple AZs
- Supports security groups

### VPC Endpoints Comparison

| Feature | Gateway Endpoint | Interface Endpoint |
|---------|------------------|-------------------|
| **Cost** | Free | $0.01/hour/AZ + data |
| **Services** | S3, DynamoDB | 100+ services |
| **Implementation** | Route table entry | ENI with private IP |
| **DNS** | Uses service DNS | Private DNS available |
| **Security Groups** | No | Yes |
| **Multiple AZs** | Automatic | Manual (create per AZ) |
| **Access Control** | Policy-based | SG + Policy |

### Creating Gateway Endpoint

```bash
# Create S3 Gateway Endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-12345678 rtb-87654321
```

**Route table automatically updated:**
```
Destination              Target
10.0.0.0/16             local
pl-12345678 (S3 prefix) vpce-12345678
```

### Creating Interface Endpoint

```bash
# Create EC2 Interface Endpoint
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.us-east-1.ec2 \
  --subnet-ids subnet-12345678 subnet-87654321 \
  --security-group-ids sg-12345678
```

### VPC Endpoint Policies

Control what actions and resources can be accessed through the endpoint.

**Example S3 Gateway Endpoint Policy:**
```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

**Example: Restrict to Specific Bucket:**
```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::production-bucket",
        "arn:aws:s3:::production-bucket/*"
      ]
    }
  ]
}
```

### Use Cases for VPC Endpoints

1. **Private S3 Access**
   - Access S3 from private subnets without NAT Gateway
   - Save NAT Gateway costs
   - Enhanced security

2. **Secrets Management**
   - Access AWS Secrets Manager privately
   - No internet exposure for sensitive data

3. **Systems Manager**
   - Manage EC2 instances without public IPs
   - No bastion hosts needed

4. **CloudWatch Logs**
   - Send logs privately
   - Compliance requirements

## VPC Flow Logs

### What are VPC Flow Logs?

VPC Flow Logs capture information about IP traffic going to and from network interfaces in your VPC.

### Flow Logs Levels

1. **VPC Level** - Captures all ENIs in VPC
2. **Subnet Level** - Captures all ENIs in subnet
3. **ENI Level** - Captures specific network interface

### Flow Log Record Format

```
<version> <account-id> <interface-id> <srcaddr> <dstaddr> <srcport> <dstport> <protocol> <packets> <bytes> <start> <end> <action> <log-status>
```

**Example Flow Log Entry:**
```
2 123456789012 eni-abc123de 172.31.16.139 172.31.16.21 20641 22 6 20 4249 1418530010 1418530070 ACCEPT OK
```

**Parsed:**
- Version: 2
- Account: 123456789012
- Interface: eni-abc123de
- Source IP: 172.31.16.139
- Destination IP: 172.31.16.21
- Source Port: 20641
- Destination Port: 22 (SSH)
- Protocol: 6 (TCP)
- Packets: 20
- Bytes: 4249
- Action: ACCEPT
- Status: OK

### Flow Logs Destinations

1. **CloudWatch Logs**
   - Real-time monitoring
   - CloudWatch Insights queries
   - Alarms and notifications

2. **S3**
   - Long-term storage
   - Cost-effective
   - Integration with Athena

3. **Kinesis Data Firehose**
   - Real-time streaming
   - Integration with analytics tools

### Creating Flow Logs

#### To CloudWatch Logs
```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole
```

#### To S3
```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::my-flow-logs-bucket/
```

### Traffic Types

- **ACCEPT** - Accepted traffic only
- **REJECT** - Rejected traffic only
- **ALL** - Both accepted and rejected (recommended)

### Use Cases for Flow Logs

1. **Security Analysis**
   - Identify unauthorized access attempts
   - Detect suspicious traffic patterns
   - Investigate security incidents

2. **Troubleshooting**
   - Diagnose connectivity issues
   - Identify why traffic is blocked
   - Verify security group rules

3. **Compliance**
   - Network traffic auditing
   - Regulatory requirements
   - Forensic analysis

4. **Cost Optimization**
   - Identify high-traffic instances
   - Optimize data transfer
   - Reduce unnecessary traffic

### Analyzing Flow Logs with CloudWatch Insights

**Query: Top Talkers (by bytes)**
```
fields @timestamp, srcaddr, dstaddr, bytes
| sort bytes desc
| limit 20
```

**Query: Rejected SSH Attempts**
```
fields @timestamp, srcaddr, dstaddr, srcport, dstport, action
| filter dstport = 22 and action = "REJECT"
| stats count() by srcaddr
| sort count desc
```

**Query: Traffic by Protocol**
```
fields @timestamp, protocol, bytes
| stats sum(bytes) by protocol
```

### Analyzing Flow Logs with Athena

**Create Athena Table:**
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS vpc_flow_logs (
  version int,
  account string,
  interfaceid string,
  sourceaddress string,
  destinationaddress string,
  sourceport int,
  destinationport int,
  protocol int,
  numpackets int,
  numbytes bigint,
  starttime int,
  endtime int,
  action string,
  logstatus string
)
PARTITIONED BY (dt string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ' '
LOCATION 's3://my-flow-logs-bucket/AWSLogs/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

**Query: Find Rejected Traffic**
```sql
SELECT sourceaddress, destinationaddress, sourceport, destinationport, protocol, action
FROM vpc_flow_logs
WHERE action = 'REJECT' AND dt = '2024-01-15'
LIMIT 100;
```

## DNS in VPC

### VPC DNS Components

1. **Amazon DNS Server**
   - Located at VPC base + 2 (e.g., 10.0.0.2 for 10.0.0.0/16)
   - Automatically provided to instances

2. **DNS Hostnames**
   - Public DNS: `ec2-54-123-45-67.compute-1.amazonaws.com`
   - Private DNS: `ip-10-0-1-23.ec2.internal`

### DNS Settings

**enableDnsSupport** (default: true)
- Enables DNS resolution in VPC
- If disabled, DNS queries to Amazon DNS server fail

**enableDnsHostnames** (default: false for custom VPC)
- Assigns public DNS hostnames to instances with public IPs
- Requires `enableDnsSupport` to be true

### Enabling DNS Settings

```bash
# Enable DNS Support
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-12345678 \
  --enable-dns-support

# Enable DNS Hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-12345678 \
  --enable-dns-hostnames
```

### Route 53 Private Hosted Zones

Associate custom domain names with your VPC.

**Use Cases:**
- Custom internal domain names
- Service discovery
- Multi-tier application DNS

**Example:**
```
Database: db.internal.mycompany.com → 10.0.2.100
API: api.internal.mycompany.com → 10.0.1.50
```

**Creating Private Hosted Zone:**
```bash
aws route53 create-hosted-zone \
  --name internal.mycompany.com \
  --vpc VPCRegion=us-east-1,VPCId=vpc-12345678 \
  --caller-reference 2024-01-15-001
```

## DHCP Options Sets

### What are DHCP Options Sets?

DHCP Options Sets define network configuration parameters for instances in your VPC.

### Configurable Options

1. **domain-name** - Domain name for instances
2. **domain-name-servers** - DNS servers (up to 4)
3. **ntp-servers** - NTP servers for time sync
4. **netbios-name-servers** - NetBIOS name servers
5. **netbios-node-type** - NetBIOS node type

### Default DHCP Options Set

- Domain name: region-specific (e.g., `ec2.internal` for us-east-1)
- Domain name servers: AmazonProvidedDNS

### Creating Custom DHCP Options Set

```bash
aws ec2 create-dhcp-options \
  --dhcp-configurations \
    Key=domain-name,Values=mycompany.local \
    Key=domain-name-servers,Values=10.0.0.2,8.8.8.8
```

**Associate with VPC:**
```bash
aws ec2 associate-dhcp-options \
  --dhcp-options-id dopt-12345678 \
  --vpc-id vpc-12345678
```

### Use Cases

1. **Custom Domain Names**
   - Internal company domain
   - Service-specific domains

2. **Custom DNS Servers**
   - On-premises DNS servers
   - Hybrid cloud DNS resolution

3. **Compliance**
   - Specific NTP servers
   - Corporate standards

## IPv6 in VPC

### IPv6 Support in AWS VPC

AWS supports dual-stack mode (IPv4 and IPv6 simultaneously).

### IPv6 CIDR Blocks

- AWS assigns /56 CIDR block to VPC
- Subnets receive /64 CIDR blocks
- All IPv6 addresses are public and globally unique
- No private IPv6 addresses

### IPv6 Architecture

```
VPC: 2600:1f13:1234:5600::/56

Subnet A: 2600:1f13:1234:5600::/64
Subnet B: 2600:1f13:1234:5601::/64
Subnet C: 2600:1f13:1234:5602::/64
```

### Enabling IPv6

#### 1. Associate IPv6 CIDR with VPC
```bash
aws ec2 associate-vpc-cidr-block \
  --vpc-id vpc-12345678 \
  --amazon-provided-ipv6-cidr-block
```

#### 2. Add IPv6 CIDR to Subnets
```bash
aws ec2 associate-subnet-cidr-block \
  --subnet-id subnet-12345678 \
  --ipv6-cidr-block 2600:1f13:1234:5600::/64
```

#### 3. Update Route Tables
```bash
# For public subnet (Internet access)
aws ec2 create-route \
  --route-table-id rtb-12345678 \
  --destination-ipv6-cidr-block ::/0 \
  --gateway-id igw-12345678

# For private subnet (Egress-only)
aws ec2 create-route \
  --route-table-id rtb-87654321 \
  --destination-ipv6-cidr-block ::/0 \
  --egress-only-internet-gateway-id eigw-12345678
```

#### 4. Update Security Groups
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --ip-permissions IpProtocol=tcp,FromPort=80,ToPort=80,Ipv6Ranges='[{CidrIpv6=::/0}]'
```

### Egress-Only Internet Gateway (IPv6)

Similar to NAT Gateway but for IPv6:
- Allows outbound IPv6 traffic
- Blocks inbound IPv6 connections
- Free to use

```bash
aws ec2 create-egress-only-internet-gateway \
  --vpc-id vpc-12345678
```

### IPv6 Best Practices

1. **Plan for Dual-Stack** - Keep IPv4 for compatibility
2. **Update Security Groups** - Add IPv6 rules
3. **Update NACLs** - Include IPv6 rules
4. **Test Applications** - Ensure IPv6 compatibility
5. **Monitor** - Track IPv6 traffic patterns

## VPC Network Firewall

### What is Network Firewall?

AWS Network Firewall is a managed stateful firewall service for VPC-level traffic filtering.

### Features

✅ **Stateful Inspection** - Track connection state  
✅ **Intrusion Prevention** - Block malicious traffic  
✅ **Web Filtering** - Domain-based filtering  
✅ **Custom Rules** - Suricata-compatible rules  
✅ **Managed Rules** - AWS Managed Threat Signatures  

### Architecture

```
┌─────────────────────────────────────────────┐
│ Internet                                    │
└─────────────────┬───────────────────────────┘
                  │
          ┌───────▼────────┐
          │ Internet GW    │
          └───────┬────────┘
                  │
     ┌────────────▼────────────┐
     │ Firewall Subnet         │
     │  ┌──────────────────┐  │
     │  │ Network Firewall │  │
     │  └──────────────────┘  │
     └────────────┬────────────┘
                  │
     ┌────────────▼────────────┐
     │ Application Subnets     │
     │  - Web Servers          │
     │  - App Servers          │
     └─────────────────────────┘
```

### Use Cases

1. **Advanced Threat Protection**
   - IPS/IDS capabilities
   - Block known malware signatures

2. **Domain Filtering**
   - Block access to malicious domains
   - Allowlist/denylist domains

3. **Compliance**
   - Centralized inspection
   - Audit logging

4. **Outbound Traffic Control**
   - Prevent data exfiltration
   - Control egress traffic

### Cost

- **Endpoint**: ~$0.395/hour
- **Data Processing**: $0.065/GB

## Hands-On Lab 1: Setup VPC Endpoint for S3

### Objective
Create a Gateway Endpoint for S3 to enable private access.

### Steps

#### 1. Create Gateway Endpoint
```bash
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345678 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-12345678
```

#### 2. Verify Route Table
```bash
aws ec2 describe-route-tables --route-table-ids rtb-12345678
```

Should show route to S3 prefix list through endpoint.

#### 3. Test from EC2 Instance
```bash
# From private instance (no internet access)
aws s3 ls s3://my-bucket/

# Should work via VPC Endpoint
```

#### 4. Check VPC Endpoint Policy (Optional)
Restrict to specific bucket:
```bash
aws ec2 modify-vpc-endpoint \
  --vpc-endpoint-id vpce-12345678 \
  --policy-document file://endpoint-policy.json
```

## Hands-On Lab 2: Enable and Analyze Flow Logs

### Objective
Enable VPC Flow Logs and analyze traffic patterns.

### Steps

#### 1. Create S3 Bucket
```bash
aws s3 mb s3://my-vpc-flow-logs-bucket
```

#### 2. Enable Flow Logs
```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::my-vpc-flow-logs-bucket/
```

#### 3. Wait for Logs (5-15 minutes)

#### 4. Query with Athena
Create table and run queries (see Athena section above).

## Hands-On Lab 3: Setup Private Hosted Zone

### Objective
Create custom internal DNS for VPC resources.

### Steps

#### 1. Create Private Hosted Zone
```bash
aws route53 create-hosted-zone \
  --name internal.mycompany.com \
  --vpc VPCRegion=us-east-1,VPCId=vpc-12345678 \
  --caller-reference $(date +%s)
```

#### 2. Create DNS Record
```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "db.internal.mycompany.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "10.0.2.100"}]
      }
    }]
  }'
```

#### 3. Test Resolution
```bash
# From EC2 instance in VPC
nslookup db.internal.mycompany.com

# Should resolve to 10.0.2.100
```

## Best Practices Summary

### VPC Endpoints
1. ✅ Use Gateway Endpoints for S3 and DynamoDB (free)
2. ✅ Use Interface Endpoints for private AWS service access
3. ✅ Apply endpoint policies for least privilege
4. ✅ Deploy in multiple AZs for high availability

### VPC Flow Logs
1. ✅ Enable on production VPCs for security
2. ✅ Use S3 for cost-effective long-term storage
3. ✅ Set up automated analysis with Athena
4. ✅ Create CloudWatch alarms for suspicious traffic

### DNS
1. ✅ Enable DNS hostnames for production VPCs
2. ✅ Use Route 53 Private Hosted Zones for custom DNS
3. ✅ Document custom DHCP options
4. ✅ Plan for hybrid DNS resolution

### IPv6
1. ✅ Test thoroughly before production deployment
2. ✅ Keep dual-stack (IPv4 + IPv6)
3. ✅ Update all security controls
4. ✅ Use Egress-Only IGW for private subnets

## Key Takeaways

✅ VPC Endpoints enable private AWS service access  
✅ Gateway Endpoints are free (S3, DynamoDB)  
✅ Interface Endpoints cost per hour + data  
✅ VPC Flow Logs essential for security and troubleshooting  
✅ Analyze Flow Logs with CloudWatch Insights or Athena  
✅ DNS settings control name resolution in VPC  
✅ IPv6 provides global addressing, all addresses are public  
✅ Network Firewall provides advanced threat protection  

## Quiz

1. What's the difference between Gateway and Interface Endpoints?
2. Which AWS services support Gateway Endpoints?
3. What information is captured in VPC Flow Logs?
4. How much do VPC Flow Logs cost?
5. What does the enableDnsHostnames setting do?
6. Are IPv6 addresses in AWS VPC public or private?
7. What is an Egress-Only Internet Gateway used for?

## Next Steps

Continue to [Module 6: Hands-On Labs](../06-hands-on-labs/README.md) for:
- Complete VPC implementation labs
- Multi-tier architecture projects
- Real-world scenarios
- CloudFormation templates

## Additional Resources

- [VPC Endpoints Documentation](https://docs.aws.amazon.com/vpc/latest/privatelink/)
- [VPC Flow Logs Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [IPv6 in VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ipv6.html)
- [Network Firewall Documentation](https://docs.aws.amazon.com/network-firewall/)
