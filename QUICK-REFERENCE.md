# AWS VPC Quick Reference Guide

## CIDR Block Quick Reference

### Common CIDR Blocks and IP Counts

| CIDR | IP Addresses | Netmask | Use Case |
|------|--------------|---------|----------|
| /32 | 1 | 255.255.255.255 | Single host |
| /31 | 2 | 255.255.255.254 | Point-to-point links |
| /30 | 4 | 255.255.255.252 | Point-to-point links |
| /29 | 8 | 255.255.255.248 | Very small subnets |
| /28 | 16 | 255.255.255.240 | Small subnets |
| /27 | 32 | 255.255.255.224 | Small subnets |
| /26 | 64 | 255.255.255.192 | Small subnets |
| /25 | 128 | 255.255.255.128 | Medium subnets |
| /24 | 256 | 255.255.255.0 | Standard subnet |
| /23 | 512 | 255.255.254.0 | Medium subnets |
| /22 | 1,024 | 255.255.252.0 | Large subnets |
| /21 | 2,048 | 255.255.248.0 | Large subnets |
| /20 | 4,096 | 255.255.240.0 | Very large subnets |
| /19 | 8,192 | 255.255.224.0 | Very large subnets |
| /18 | 16,384 | 255.255.192.0 | Extra large subnets |
| /17 | 32,768 | 255.255.128.0 | Extra large subnets |
| /16 | 65,536 | 255.255.0.0 | Standard VPC |

### AWS Reserved IPs (per subnet)
- **5 IPs reserved** in each subnet
- Example for 10.0.1.0/24:
  - 10.0.1.0 - Network address
  - 10.0.1.1 - VPC router
  - 10.0.1.2 - DNS server
  - 10.0.1.3 - Reserved for future use
  - 10.0.1.255 - Network broadcast

**Usable IPs = Total - 5**

## AWS CLI Quick Commands

### VPC Commands

```bash
# List VPCs
aws ec2 describe-vpcs

# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Delete VPC
aws ec2 delete-vpc --vpc-id vpc-12345678

# Enable DNS hostnames
aws ec2 modify-vpc-attribute --vpc-id vpc-12345678 --enable-dns-hostnames

# Describe VPC attributes
aws ec2 describe-vpc-attribute --vpc-id vpc-12345678 --attribute enableDnsHostnames
```

### Subnet Commands

```bash
# List subnets
aws ec2 describe-subnets

# Create subnet
aws ec2 create-subnet --vpc-id vpc-12345678 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a

# Delete subnet
aws ec2 delete-subnet --subnet-id subnet-12345678

# Enable auto-assign public IP
aws ec2 modify-subnet-attribute --subnet-id subnet-12345678 --map-public-ip-on-launch
```

### Internet Gateway Commands

```bash
# Create IGW
aws ec2 create-internet-gateway

# Attach IGW to VPC
aws ec2 attach-internet-gateway --vpc-id vpc-12345678 --internet-gateway-id igw-12345678

# Detach IGW
aws ec2 detach-internet-gateway --vpc-id vpc-12345678 --internet-gateway-id igw-12345678

# Delete IGW
aws ec2 delete-internet-gateway --internet-gateway-id igw-12345678
```

### NAT Gateway Commands

```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Create NAT Gateway
aws ec2 create-nat-gateway --subnet-id subnet-12345678 --allocation-id eipalloc-12345678

# Describe NAT Gateway
aws ec2 describe-nat-gateways --nat-gateway-id nat-12345678

# Delete NAT Gateway
aws ec2 delete-nat-gateway --nat-gateway-id nat-12345678
```

### Route Table Commands

```bash
# Create route table
aws ec2 create-route-table --vpc-id vpc-12345678

# Create route
aws ec2 create-route --route-table-id rtb-12345678 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-12345678

# Associate route table with subnet
aws ec2 associate-route-table --subnet-id subnet-12345678 --route-table-id rtb-12345678

# Describe route tables
aws ec2 describe-route-tables --route-table-ids rtb-12345678
```

### Security Group Commands

```bash
# Create security group
aws ec2 create-security-group --group-name my-sg --description "My SG" --vpc-id vpc-12345678

# Add ingress rule
aws ec2 authorize-security-group-ingress --group-id sg-12345678 --protocol tcp --port 80 --cidr 0.0.0.0/0

# Add ingress rule (reference another SG)
aws ec2 authorize-security-group-ingress --group-id sg-12345678 --protocol tcp --port 3306 --source-group sg-87654321

# Remove rule
aws ec2 revoke-security-group-ingress --group-id sg-12345678 --protocol tcp --port 22 --cidr 0.0.0.0/0

# Describe security groups
aws ec2 describe-security-groups --group-ids sg-12345678
```

### VPC Flow Logs Commands

```bash
# Create flow logs (to S3)
aws ec2 create-flow-logs --resource-type VPC --resource-ids vpc-12345678 --traffic-type ALL \
  --log-destination-type s3 --log-destination arn:aws:s3:::my-bucket/

# Create flow logs (to CloudWatch)
aws ec2 create-flow-logs --resource-type VPC --resource-ids vpc-12345678 --traffic-type ALL \
  --log-destination-type cloud-watch-logs --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole

# Describe flow logs
aws ec2 describe-flow-logs

# Delete flow logs
aws ec2 delete-flow-logs --flow-log-ids fl-12345678
```

### VPC Endpoint Commands

```bash
# Create S3 Gateway Endpoint
aws ec2 create-vpc-endpoint --vpc-id vpc-12345678 --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-12345678

# Create Interface Endpoint
aws ec2 create-vpc-endpoint --vpc-id vpc-12345678 --vpc-endpoint-type Interface \
  --service-name com.amazonaws.us-east-1.ec2 --subnet-ids subnet-12345678 \
  --security-group-ids sg-12345678

# Describe endpoints
aws ec2 describe-vpc-endpoints

# Delete endpoint
aws ec2 delete-vpc-endpoints --vpc-endpoint-ids vpce-12345678
```

## Port Numbers Reference

### Common Ports

| Port | Protocol | Service |
|------|----------|---------|
| 20 | TCP | FTP Data |
| 21 | TCP | FTP Control |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 143 | TCP | IMAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 5432 | TCP | PostgreSQL |
| 5439 | TCP | Redshift |
| 6379 | TCP | Redis |
| 8080 | TCP | HTTP Alt |
| 27017 | TCP | MongoDB |

### AWS Service Endpoints

| Service | Port | Protocol |
|---------|------|----------|
| S3 | 443 | HTTPS |
| DynamoDB | 443 | HTTPS |
| EC2 API | 443 | HTTPS |
| RDS MySQL | 3306 | TCP |
| RDS PostgreSQL | 5432 | TCP |
| ElastiCache Redis | 6379 | TCP |
| ElastiCache Memcached | 11211 | TCP |

## Security Group vs NACL Quick Comparison

| Feature | Security Group | NACL |
|---------|---------------|------|
| **Scope** | Instance (ENI) level | Subnet level |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow and Deny |
| **Rule Order** | All evaluated | Numbered order |
| **Applies To** | Specified instances | All instances in subnet |
| **Default** | Deny all inbound | Allow all |
| **Return Traffic** | Auto allowed | Must be explicitly allowed |
| **Max Rules** | 60 inbound, 60 outbound | 20 inbound, 20 outbound |

## VPC Limits

| Resource | Default Limit | Hard Limit |
|----------|--------------|------------|
| VPCs per Region | 5 | Adjustable |
| Subnets per VPC | 200 | Adjustable |
| Internet Gateways per Region | 5 | = VPCs limit |
| NAT Gateways per AZ | 5 | Adjustable |
| Elastic IPs per Region | 5 | Adjustable |
| Security Groups per VPC | 2,500 | 10,000 |
| Rules per Security Group | 60 | 100 |
| Security Groups per ENI | 5 | 16 |
| VPC Peering Connections per VPC | 50 | 125 |
| Active VPN Connections per VPC | 10 | Adjustable |
| Route Tables per VPC | 200 | Adjustable |
| Routes per Route Table | 50 | 100 |

## Troubleshooting Checklist

### Cannot SSH to Instance
- [ ] Security group allows port 22 from your IP
- [ ] NACL allows port 22 inbound
- [ ] NACL allows ephemeral ports outbound
- [ ] Instance has public IP (if accessing from internet)
- [ ] Route table has route to IGW (public subnet)
- [ ] Correct SSH key
- [ ] SSH service running on instance

### Instance Cannot Access Internet
- [ ] Internet Gateway attached to VPC
- [ ] Route to IGW in route table (0.0.0.0/0 → igw-xxx)
- [ ] Instance has public IP or Elastic IP
- [ ] Security group allows outbound traffic
- [ ] NACL allows outbound traffic
- [ ] NACL allows ephemeral ports inbound (for return traffic)

### Private Instance Cannot Reach Internet via NAT
- [ ] NAT Gateway in public subnet
- [ ] NAT Gateway has Elastic IP
- [ ] Public subnet has route to IGW
- [ ] Private subnet route table has route to NAT Gateway (0.0.0.0/0 → nat-xxx)
- [ ] NAT Gateway state is "available"
- [ ] Security groups allow traffic
- [ ] NACLs allow traffic

### VPC Peering Not Working
- [ ] Peering connection status is "active"
- [ ] Routes exist in both VPCs pointing to peering connection
- [ ] Security groups allow traffic from peer VPC
- [ ] NACLs allow traffic
- [ ] No CIDR overlap
- [ ] Not trying to route transitively

## Cost Calculator

### Monthly Cost Estimates

**NAT Gateway (per AZ)**
- Hours: 730 × $0.045 = **$32.85**
- Data: 100GB × $0.045 = **$4.50**
- **Total: ~$37/month**

**VPN Connection**
- Connection: 730 × $0.05 = **$36.50**
- Data transfer: Standard rates
- **Total: ~$37/month+**

**Transit Gateway (5 attachments)**
- Attachments: 5 × 730 × $0.05 = **$182.50**
- Data: 1TB × $0.02 = **$20.48**
- **Total: ~$203/month**

**VPC Endpoints (Interface)**
- Per AZ: 730 × $0.01 = **$7.30**
- Data: 100GB × $0.01 = **$1.00**
- **Total: ~$8/month per AZ**

## Best Practices Checklist

### Design
- [ ] Plan CIDR blocks to avoid overlap
- [ ] Use /16 for VPC, /24 for subnets
- [ ] Deploy across multiple AZs
- [ ] Separate public and private subnets
- [ ] Leave room for growth

### Security
- [ ] Use security group chaining
- [ ] Principle of least privilege
- [ ] Restrict SSH/RDP access
- [ ] Enable VPC Flow Logs
- [ ] Regular security audits
- [ ] Use VPC Endpoints for AWS services
- [ ] Separate environments (dev/staging/prod)

### High Availability
- [ ] Multi-AZ deployment
- [ ] NAT Gateway in each AZ
- [ ] Multiple route tables
- [ ] Load balancers across AZs
- [ ] Database replication

### Cost Optimization
- [ ] Use VPC Endpoints instead of NAT for AWS services
- [ ] Consider NAT instances for dev/test
- [ ] Right-size NAT Gateways
- [ ] Use VPC Peering over Transit Gateway when possible
- [ ] Monitor and optimize data transfer

### Operations
- [ ] Tag all resources
- [ ] Document architecture
- [ ] Enable CloudWatch monitoring
- [ ] Set up alerts
- [ ] Regular backups
- [ ] Disaster recovery plan

## Exam Tips (AWS Certifications)

### Solutions Architect Associate
- Understand VPC components and how they work together
- Know Security Groups vs NACLs
- Understand VPC Peering limitations (no transitive routing)
- Know when to use NAT Gateway vs NAT Instance
- Understand VPC Endpoints (Gateway vs Interface)
- Know high availability patterns

### Advanced Networking Specialty
- Deep understanding of routing
- Transit Gateway architecture
- Direct Connect and VPN
- Hybrid connectivity patterns
- Advanced security configurations
- Network troubleshooting

## Quick Decision Trees

### Need Internet Access?
```
Public subnet?
├─ Yes → Internet Gateway → Instance needs public IP
└─ No (Private) → NAT Gateway in public subnet → Route to NAT
```

### Connect VPCs?
```
How many VPCs?
├─ 2-5 → VPC Peering (simple, cost-effective)
└─ 5+ → Transit Gateway (centralized management)
```

### On-Premises Connection?
```
Bandwidth needs?
├─ Low/Medium (<100 Mbps) → Site-to-Site VPN
└─ High (>100 Mbps) → Direct Connect
```

## Additional Tools

### Online CIDR Calculators
- https://www.ipaddressguide.com/cidr
- https://cidr.xyz/
- https://www.subnet-calculator.com/

### AWS Tools
- VPC Reachability Analyzer
- Network Access Analyzer
- AWS Network Manager
- VPC Flow Logs Insights

## Common Mistakes to Avoid

1. ❌ Overlapping CIDR ranges
2. ❌ Not planning for growth
3. ❌ Single AZ deployment
4. ❌ Opening SSH/RDP to 0.0.0.0/0
5. ❌ Not using security group chaining
6. ❌ Forgetting to enable DNS hostnames
7. ❌ Not cleaning up resources (costs!)
8. ❌ Missing ephemeral ports in NACLs
9. ❌ Expecting transitive routing with VPC Peering
10. ❌ Not monitoring with Flow Logs

---

**Keep this guide handy for quick reference!** 📚
