# Module 3: VPC Security

## Introduction to VPC Security

Security in AWS VPC operates at multiple layers, providing defense in depth. Understanding these security mechanisms is crucial for protecting your cloud resources.

## Security Layers in VPC

```
┌─────────────────────────────────────────────┐
│ Internet                                    │
└─────────────────┬───────────────────────────┘
                  │
          ┌───────▼────────┐
          │  Edge Security │  ← AWS Shield, WAF
          └───────┬────────┘
                  │
          ┌───────▼────────┐
          │  VPC Layer     │  ← NACL
          └───────┬────────┘
                  │
          ┌───────▼────────┐
          │  Subnet Layer  │  ← NACL
          └───────┬────────┘
                  │
          ┌───────▼────────┐
          │  Instance      │  ← Security Groups
          └────────────────┘
```

## Security Groups

### What are Security Groups?

Security Groups act as virtual firewalls for your EC2 instances, controlling inbound and outbound traffic at the instance level.

### Key Characteristics

✅ **Stateful** - Return traffic automatically allowed  
✅ **Instance Level** - Applied to network interfaces (ENIs)  
✅ **Allow Rules Only** - Cannot create deny rules  
✅ **All Rules Evaluated** - All matching rules are applied  
✅ **Default Deny** - All traffic denied unless explicitly allowed  

### Security Group Rules

Each rule consists of:
- **Type** - Protocol (TCP, UDP, ICMP, or All)
- **Port Range** - Which ports to allow
- **Source/Destination** - IP ranges or other security groups
- **Description** - Rule purpose (optional but recommended)

### Inbound Rules Example

```
Type          Protocol  Port Range  Source              Description
HTTP          TCP       80          0.0.0.0/0           Public web access
HTTPS         TCP       443         0.0.0.0/0           Public HTTPS access
SSH           TCP       22          10.0.0.0/16         VPC only SSH
MySQL         TCP       3306        sg-app123           App tier access
Custom TCP    TCP       8080        sg-lb456            Load balancer
```

### Outbound Rules Example

```
Type          Protocol  Port Range  Destination         Description
All Traffic   All       All         0.0.0.0/0           Allow all outbound
HTTP          TCP       80          0.0.0.0/0           Web requests
HTTPS         TCP       443         0.0.0.0/0           HTTPS requests
MySQL         TCP       3306        sg-db789            Database access
```

### Stateful Nature

Security Groups are **stateful** - if you allow inbound traffic, the response is automatically allowed, regardless of outbound rules.

**Example:**
```
Inbound Rule: Allow TCP 80 from 0.0.0.0/0
Outbound Rule: Allow TCP 443 to 0.0.0.0/0

Result:
✅ Incoming HTTP on port 80 - Allowed
✅ Response to HTTP traffic - Automatically allowed (stateful)
✅ Outgoing HTTPS on port 443 - Allowed
❌ Outgoing HTTP on port 80 - Blocked (not in outbound rules)
```

### Security Group Chaining

You can reference other security groups instead of IP addresses:

```
┌──────────────────┐
│  Load Balancer   │
│  sg-lb           │  Inbound: 0.0.0.0/0:443
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Web Tier        │
│  sg-web          │  Inbound: sg-lb:80
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  App Tier        │
│  sg-app          │  Inbound: sg-web:8080
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Database        │
│  sg-db           │  Inbound: sg-app:3306
└──────────────────┘
```

**Benefits:**
- Dynamic updates (no IP changes needed)
- Easier management
- More secure (only specific resources can connect)

### Default Security Group

Every VPC has a default security group:

**Default Inbound Rules:**
- Allow all traffic from instances with same security group

**Default Outbound Rules:**
- Allow all outbound traffic

**Best Practice:** Don't use default security group for production resources.

### Security Group Limits

| Resource | Default Limit | Adjustable |
|----------|--------------|------------|
| Security groups per VPC | 2,500 | Yes (up to 10,000) |
| Rules per security group | 60 | Yes (up to 100) |
| Security groups per network interface | 5 | Yes (up to 16) |

## Network Access Control Lists (NACLs)

### What are NACLs?

Network Access Control Lists are stateless firewalls that control traffic at the subnet level.

### Key Characteristics

✅ **Stateless** - Must explicitly allow both request and response  
✅ **Subnet Level** - Applied to entire subnet  
✅ **Allow and Deny Rules** - Can explicitly deny traffic  
✅ **Ordered Rules** - Evaluated in numerical order  
✅ **Default Deny** - Ends with implicit deny all  

### NACL vs Security Groups

| Feature | Security Group | NACL |
|---------|---------------|------|
| **Level** | Instance (ENI) | Subnet |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow and Deny |
| **Rule Processing** | All rules | Order matters |
| **Applies To** | Specified instances | All instances in subnet |
| **Default** | Deny all inbound | Allow all |

### NACL Rule Components

Each rule has:
- **Rule Number** - Evaluation order (1-32766)
- **Type** - Protocol
- **Port Range** - Affected ports
- **Source/Destination** - CIDR blocks
- **Allow/Deny** - Action

### NACL Rules Example

**Inbound Rules:**
```
Rule #  Type          Protocol  Port Range  Source         Allow/Deny
100     HTTP          TCP       80          0.0.0.0/0      ALLOW
110     HTTPS         TCP       443         0.0.0.0/0      ALLOW
120     SSH           TCP       22          10.0.0.0/16    ALLOW
130     Custom TCP    TCP       1024-65535  0.0.0.0/0      ALLOW (ephemeral)
*       All Traffic   All       All         0.0.0.0/0      DENY
```

**Outbound Rules:**
```
Rule #  Type          Protocol  Port Range  Destination    Allow/Deny
100     HTTP          TCP       80          0.0.0.0/0      ALLOW
110     HTTPS         TCP       443         0.0.0.0/0      ALLOW
120     Custom TCP    TCP       1024-65535  0.0.0.0/0      ALLOW (ephemeral)
*       All Traffic   All       All         0.0.0.0/0      DENY
```

### Ephemeral Ports

Because NACLs are stateless, you must allow ephemeral ports for return traffic:

| OS | Ephemeral Port Range |
|----|---------------------|
| Linux | 32768-60999 |
| Windows Server 2003-2008 | 1025-5000 |
| Windows Server 2008+ | 49152-65535 |
| Elastic Load Balancer | 1024-65535 |

**Best Practice:** Allow 1024-65535 for broad compatibility.

### NACL Rule Evaluation

Rules are evaluated in order from lowest to highest number:

```
Example Request: SSH from 203.0.113.5

Rule #  Port  Source          Action      Result
100     22    203.0.113.0/24  DENY        ← Matches! Traffic DENIED
200     22    0.0.0.0/0       ALLOW       ← Not evaluated
*       All   0.0.0.0/0       DENY        ← Not reached
```

**Key Point:** First matching rule wins, rest are ignored.

### Default NACL

Every VPC has a default NACL that:
- Allows all inbound traffic
- Allows all outbound traffic
- Associated with all subnets by default

**Default NACL Rules:**
```
Inbound:
Rule #  Type          Port Range  Source      Allow/Deny
100     All Traffic   All         0.0.0.0/0   ALLOW
*       All Traffic   All         0.0.0.0/0   DENY

Outbound:
Rule #  Type          Port Range  Dest        Allow/Deny
100     All Traffic   All         0.0.0.0/0   ALLOW
*       All Traffic   All         0.0.0.0/0   DENY
```

### Custom NACL

When you create a custom NACL:
- All traffic is denied by default (inbound and outbound)
- Must explicitly add allow rules
- Must handle stateless nature (allow both directions)

## Security Groups vs NACLs in Practice

### Scenario 1: Web Server Security

**Security Group (sg-web):**
```
Inbound:
- HTTP (80) from 0.0.0.0/0
- HTTPS (443) from 0.0.0.0/0
- SSH (22) from 10.0.0.0/16

Outbound:
- All traffic to 0.0.0.0/0
```

**NACL (nacl-public):**
```
Inbound:
- 100: HTTP (80) from 0.0.0.0/0 - ALLOW
- 110: HTTPS (443) from 0.0.0.0/0 - ALLOW
- 120: SSH (22) from 10.0.0.0/16 - ALLOW
- 130: Ephemeral (1024-65535) from 0.0.0.0/0 - ALLOW
- *: All traffic - DENY

Outbound:
- 100: HTTP (80) to 0.0.0.0/0 - ALLOW
- 110: HTTPS (443) to 0.0.0.0/0 - ALLOW
- 120: Ephemeral (1024-65535) to 0.0.0.0/0 - ALLOW
- *: All traffic - DENY
```

### Scenario 2: Block Specific IP

Use NACL to block IP (Security Groups can't deny):

```
NACL Inbound Rules:
10:  All traffic from 203.0.113.0/24 - DENY  ← Blocks bad actor
100: HTTP from 0.0.0.0/0 - ALLOW
110: HTTPS from 0.0.0.0/0 - ALLOW
```

### When to Use Each

**Use Security Groups for:**
- ✅ Instance-level security
- ✅ Application access control
- ✅ Service-to-service communication
- ✅ Dynamic IP ranges
- ✅ Most security requirements

**Use NACLs for:**
- ✅ Subnet-level security
- ✅ Explicit deny rules
- ✅ Blocking specific IPs or ranges
- ✅ Compliance requirements
- ✅ Additional layer of defense

## Security Best Practices

### 1. Principle of Least Privilege

**❌ Bad:**
```
Inbound: 0.0.0.0/0 on all ports
```

**✅ Good:**
```
Inbound: 
- 10.0.0.0/16 on port 22 (SSH from VPC only)
- sg-lb on port 80 (HTTP from load balancer only)
```

### 2. Use Security Group Chaining

**❌ Bad:**
```
Database SG: Allow 0.0.0.0/0 on port 3306
```

**✅ Good:**
```
Database SG: Allow sg-app on port 3306
```

### 3. Restrict SSH/RDP Access

**❌ Bad:**
```
SSH: 0.0.0.0/0 on port 22
RDP: 0.0.0.0/0 on port 3389
```

**✅ Good:**
```
SSH: 10.0.0.0/16 on port 22 (VPC only)
Or use AWS Systems Manager Session Manager (no inbound ports needed)
```

### 4. Regular Security Audits

Use AWS tools to audit security:
- **AWS Config** - Track security group changes
- **AWS Security Hub** - Compliance checks
- **VPC Flow Logs** - Monitor traffic patterns
- **AWS GuardDuty** - Threat detection

### 5. Descriptive Naming and Tagging

**❌ Bad:**
```
Security Group: sg-12345
Description: (empty)
```

**✅ Good:**
```
Security Group: prod-web-tier-sg
Description: Security group for production web servers
Tags:
  Environment: Production
  Tier: Web
  Owner: TeamA
```

### 6. Separate Environments

Create separate security groups for:
- Development
- Staging
- Production

**Never mix production and non-production security groups.**

### 7. Document Security Rules

Add descriptions to every rule:
```
Rule: Allow TCP 443 from 0.0.0.0/0
Description: Public HTTPS access for customer portal
```

### 8. Use VPC Flow Logs

Enable Flow Logs to monitor:
- Rejected connections (potential attacks)
- Accepted connections (audit trail)
- Traffic patterns (baseline normal behavior)

### 9. Defense in Depth

Use multiple security layers:
```
1. Network ACLs (Subnet level)
2. Security Groups (Instance level)
3. Host-based firewalls (OS level)
4. Application security (App level)
```

### 10. Avoid Default Security Group

Create custom security groups instead of using the default.

## Common Security Patterns

### Pattern 1: Three-Tier Web Application

```
┌─────────────────────────────────────────────┐
│ Load Balancer (sg-lb)                       │
│ In: 0.0.0.0/0:443                          │
│ Out: sg-web:80                             │
└─────────────────┬───────────────────────────┘
                  │
┌─────────────────▼───────────────────────────┐
│ Web Tier (sg-web)                           │
│ In: sg-lb:80                                │
│ Out: sg-app:8080                            │
└─────────────────┬───────────────────────────┘
                  │
┌─────────────────▼───────────────────────────┐
│ App Tier (sg-app)                           │
│ In: sg-web:8080                             │
│ Out: sg-db:3306                             │
└─────────────────┬───────────────────────────┘
                  │
┌─────────────────▼───────────────────────────┐
│ Database Tier (sg-db)                       │
│ In: sg-app:3306                             │
│ Out: None                                   │
└─────────────────────────────────────────────┘
```

### Pattern 2: Bastion Host

```
┌─────────────────────────────────────────────┐
│ Bastion Host (sg-bastion)                   │
│ In: Corporate-IP:22                         │
│ Out: 10.0.0.0/16:22                        │
└─────────────────┬───────────────────────────┘
                  │
                  │ SSH Tunnel
                  │
┌─────────────────▼───────────────────────────┐
│ Private Instances (sg-private)              │
│ In: sg-bastion:22                           │
│ Out: Internet via NAT                       │
└─────────────────────────────────────────────┘
```

### Pattern 3: Database with Replica

```
┌─────────────────────────────────────────────┐
│ Primary Database (sg-db-primary)            │
│ In: sg-app:3306, sg-db-replica:3306        │
│ Out: sg-db-replica:3306                     │
└─────────────────┬───────────────────────────┘
                  │
                  │ Replication
                  │
┌─────────────────▼───────────────────────────┐
│ Replica Database (sg-db-replica)            │
│ In: sg-db-primary:3306                      │
│ Out: None                                   │
└─────────────────────────────────────────────┘
```

## Security Group Management with AWS CLI

### Create Security Group
```bash
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web server security group" \
  --vpc-id vpc-12345678
```

### Add Inbound Rule
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

### Add Rule Referencing Another SG
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 3306 \
  --source-group sg-87654321
```

### Remove Rule
```bash
aws ec2 revoke-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

### Describe Security Groups
```bash
aws ec2 describe-security-groups --group-ids sg-12345678
```

## NACL Management with AWS CLI

### Create NACL
```bash
aws ec2 create-network-acl \
  --vpc-id vpc-12345678 \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=custom-nacl}]'
```

### Add NACL Rule
```bash
aws ec2 create-network-acl-entry \
  --network-acl-id acl-12345678 \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --egress false \
  --rule-action allow
```

### Delete NACL Rule
```bash
aws ec2 delete-network-acl-entry \
  --network-acl-id acl-12345678 \
  --rule-number 100 \
  --egress false
```

## Hands-On Lab 1: Create Security Groups

### Objective
Create a three-tier security group architecture.

### Steps

#### 1. Create Load Balancer SG
```bash
aws ec2 create-security-group \
  --group-name lb-sg \
  --description "Load Balancer Security Group" \
  --vpc-id <vpc-id>

# Allow HTTPS from internet
aws ec2 authorize-security-group-ingress \
  --group-id <sg-lb-id> \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

#### 2. Create Web Tier SG
```bash
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web Tier Security Group" \
  --vpc-id <vpc-id>

# Allow HTTP from Load Balancer
aws ec2 authorize-security-group-ingress \
  --group-id <sg-web-id> \
  --protocol tcp \
  --port 80 \
  --source-group <sg-lb-id>
```

#### 3. Create Database SG
```bash
aws ec2 create-security-group \
  --group-name db-sg \
  --description "Database Security Group" \
  --vpc-id <vpc-id>

# Allow MySQL from Web Tier
aws ec2 authorize-security-group-ingress \
  --group-id <sg-db-id> \
  --protocol tcp \
  --port 3306 \
  --source-group <sg-web-id>
```

## Hands-On Lab 2: Configure NACLs

### Objective
Create custom NACL for public subnet with restricted access.

### Implementation

See [lab2-nacl-configuration.yaml](./cloudformation/lab2-nacl-configuration.yaml)

## Troubleshooting Security Issues

### Instance Cannot Connect to Database

**Checklist:**
1. ✅ Database security group allows source security group
2. ✅ Database security group allows correct port (3306 for MySQL)
3. ✅ NACL allows traffic on database port
4. ✅ NACL allows ephemeral ports for return traffic
5. ✅ Route table allows traffic between subnets
6. ✅ Database is running and accessible

### Cannot SSH to Instance

**Checklist:**
1. ✅ Security group allows SSH (22) from your IP
2. ✅ NACL allows SSH inbound
3. ✅ NACL allows ephemeral ports outbound
4. ✅ Instance has public IP (if accessing from internet)
5. ✅ Route table has route to IGW (public) or correct route
6. ✅ Correct SSH key pair
7. ✅ SSH service running on instance

### Web Server Not Accessible

**Checklist:**
1. ✅ Security group allows HTTP/HTTPS from 0.0.0.0/0
2. ✅ NACL allows HTTP/HTTPS inbound
3. ✅ NACL allows ephemeral ports outbound
4. ✅ Web server is running
5. ✅ Instance has public IP or is behind load balancer
6. ✅ Route table configured correctly

## Security Audit Checklist

- [ ] No security groups allow 0.0.0.0/0 on SSH/RDP
- [ ] Database security groups only allow application tier
- [ ] All security groups have descriptions
- [ ] Security groups use chaining (not IP addresses)
- [ ] NACLs are configured for subnet protection
- [ ] VPC Flow Logs are enabled
- [ ] Regular review of security group rules
- [ ] Unused security groups are removed
- [ ] Tags are applied to all security groups
- [ ] Alerts configured for security group changes

## Key Takeaways

✅ Security Groups are stateful, instance-level firewalls  
✅ NACLs are stateless, subnet-level firewalls  
✅ Use Security Groups for most requirements  
✅ Use NACLs for explicit deny rules  
✅ Security Group rules are all evaluated (no order)  
✅ NACL rules are evaluated in order (first match wins)  
✅ Always apply principle of least privilege  
✅ Use security group chaining for better security  
✅ Document all rules with descriptions  
✅ Regular audits are essential  

## Quiz

1. What is the difference between stateful and stateless firewalls?
2. Can Security Groups deny traffic? Can NACLs?
3. What are ephemeral ports and why are they important for NACLs?
4. How does rule evaluation differ between Security Groups and NACLs?
5. What's the advantage of security group chaining over IP-based rules?
6. If you need to block a specific IP address, should you use Security Groups or NACLs?

## Next Steps

Continue to [Module 4: VPC Connectivity](../04-vpc-connectivity/README.md) to learn about:
- VPC Peering
- Transit Gateway
- VPN Connections
- Direct Connect
- PrivateLink

## Additional Resources

- [Security Groups Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [Network ACLs Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [VPC Security Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)
