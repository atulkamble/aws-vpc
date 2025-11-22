# AWS VPC Glossary

A comprehensive glossary of AWS VPC networking terms.

---

## A

**Availability Zone (AZ)**  
An isolated location within an AWS Region. Each Region has multiple AZs for fault tolerance.

**Amazon Resource Name (ARN)**  
Unique identifier for AWS resources.

**Application Load Balancer (ALB)**  
Layer 7 load balancer that routes HTTP/HTTPS traffic based on content.

**AWS Direct Connect**  
Dedicated network connection from on-premises to AWS.

**Auto Scaling**  
Automatic adjustment of compute resources based on demand.

---

## B

**Bastion Host**  
A server that provides secure access to private instances. Also called jump box.

**BGP (Border Gateway Protocol)**  
Routing protocol used for exchanging routing information between networks.

**Broadcast Domain**  
Network segment where broadcast packets are received by all devices.

---

## C

**CIDR (Classless Inter-Domain Routing)**  
Method for allocating IP addresses and routing. Example: 10.0.0.0/16

**Customer Gateway (CGW)**  
Physical or software device on customer side of VPN connection.

**CloudWatch**  
AWS monitoring and observability service.

**Cross-Region Peering**  
VPC peering connection between VPCs in different AWS Regions.

---

## D

**DHCP (Dynamic Host Configuration Protocol)**  
Network protocol for automatically assigning IP addresses.

**DHCP Options Set**  
Configuration parameters for DHCP in VPC (DNS servers, domain name, etc.).

**DDoS (Distributed Denial of Service)**  
Attack that overwhelms a system with traffic from multiple sources.

**DNS (Domain Name System)**  
System that translates domain names to IP addresses.

**Default VPC**  
Pre-configured VPC that comes with every AWS account.

---

## E

**Egress**  
Outbound traffic leaving a network.

**Egress-Only Internet Gateway**  
Gateway that allows outbound IPv6 traffic while blocking inbound connections.

**Elastic IP (EIP)**  
Static public IPv4 address that can be associated with instances.

**Elastic Network Interface (ENI)**  
Virtual network card that can be attached to EC2 instances.

**Ephemeral Port**  
Temporary port number assigned by operating system for outbound connections (typically 1024-65535).

---

## F

**Firewall**  
Network security system that monitors and controls incoming and outgoing traffic.

**Flow Logs**  
Feature that captures IP traffic information for VPC, subnet, or ENI.

---

## G

**Gateway Endpoint**  
VPC endpoint that uses route tables to direct traffic to AWS services (S3, DynamoDB).

**Gateway Load Balancer (GWLB)**  
Layer 3 load balancer for deploying, scaling, and managing third-party virtual appliances.

---

## H

**High Availability (HA)**  
System design that ensures operational continuity during failures.

**Hosted Zone**  
Container for DNS records in Route 53.

**Hybrid Cloud**  
Computing environment combining on-premises and cloud resources.

---

## I

**Ingress**  
Inbound traffic entering a network.

**Internet Gateway (IGW)**  
VPC component that allows communication between VPC and internet.

**Interface Endpoint**  
VPC endpoint that creates ENI with private IP for accessing AWS services.

**IPsec**  
Protocol suite for secure IP communications through encryption and authentication.

**IPv4**  
Internet Protocol version 4 (32-bit addresses like 192.168.1.1).

**IPv6**  
Internet Protocol version 6 (128-bit addresses like 2001:0db8::1).

---

## J

**Jump Box**  
See Bastion Host.

---

## K

**KMS (Key Management Service)**  
AWS service for creating and managing encryption keys.

---

## L

**Latency**  
Time delay between sending and receiving data.

**Load Balancer**  
Distributes incoming traffic across multiple targets for high availability.

---

## M

**MTU (Maximum Transmission Unit)**  
Largest packet size that can be transmitted on a network (typically 1500 bytes, 9001 for jumbo frames).

**Multicast**  
Network communication where data is sent to multiple specific recipients.

**Multi-AZ**  
Deployment across multiple Availability Zones for high availability.

---

## N

**NAT (Network Address Translation)**  
Technique for mapping private IP addresses to public IP addresses.

**NAT Gateway**  
Managed AWS service that allows private subnet instances to access internet.

**NAT Instance**  
EC2 instance configured to perform NAT functionality.

**NACL (Network Access Control List)**  
Stateless firewall that controls traffic at subnet level.

**Network Firewall**  
AWS managed stateful firewall service for VPC.

**Network Interface**  
Virtual or physical network connection point.

---

## O

**On-Premises**  
Infrastructure located at customer's physical location (not in cloud).

---

## P

**Packet**  
Unit of data transmitted over a network.

**Peering Connection**  
Networking connection between two VPCs.

**Private IP Address**  
IP address used within private network (not routable on internet).

**Private Subnet**  
Subnet without direct internet access.

**PrivateLink**  
AWS service for private connectivity to services without using public IPs.

**Public IP Address**  
IP address routable on the internet.

**Public Subnet**  
Subnet with route to Internet Gateway.

---

## Q

**Quota**  
AWS service limit (e.g., maximum number of VPCs per region).

---

## R

**Region**  
Geographic area containing multiple Availability Zones.

**Route Table**  
Set of rules (routes) that determine where network traffic is directed.

**Router**  
Network device that forwards data packets between networks.

**Route 53**  
AWS DNS and domain registration service.

**RFC 1918**  
Standard defining private IP address ranges:
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

---

## S

**Stateful**  
Maintains state of connections (return traffic automatically allowed).

**Stateless**  
Does not maintain connection state (each direction must be explicitly allowed).

**Subnet**  
Subdivision of IP network within a VPC.

**Security Group**  
Stateful virtual firewall that controls inbound and outbound traffic at instance level.

**Site-to-Site VPN**  
Encrypted connection between on-premises network and AWS VPC.

---

## T

**TCP (Transmission Control Protocol)**  
Connection-oriented protocol ensuring reliable data delivery.

**Transit Gateway**  
Network hub that connects VPCs and on-premises networks.

**Throughput**  
Amount of data transferred over time (measured in Gbps or Mbps).

**TTL (Time To Live)**  
Value indicating how long data should be kept or how many hops a packet can make.

---

## U

**UDP (User Datagram Protocol)**  
Connectionless protocol for fast, low-overhead communication.

**Unicast**  
Network communication between single sender and single receiver.

---

## V

**VGW (Virtual Private Gateway)**  
VPN endpoint on AWS side of Site-to-Site VPN connection.

**VLAN (Virtual Local Area Network)**  
Logical grouping of network devices.

**VPC (Virtual Private Cloud)**  
Logically isolated virtual network in AWS cloud.

**VPC Endpoint**  
Enables private connections between VPC and AWS services.

**VPC Flow Logs**  
Feature that captures information about IP traffic in VPC.

**VPC Peering**  
Networking connection between two VPCs.

**VPN (Virtual Private Network)**  
Encrypted connection over less secure network.

---

## W

**WAF (Web Application Firewall)**  
Firewall that monitors and filters HTTP/HTTPS traffic.

**Well-Architected Framework**  
AWS best practices for building secure, high-performing, resilient, and efficient infrastructure.

---

## X

**XOR (Exclusive OR)**  
Logical operation used in networking calculations.

---

## Common Acronyms

| Acronym | Full Form |
|---------|-----------|
| ACL | Access Control List |
| ALB | Application Load Balancer |
| ARN | Amazon Resource Name |
| ASG | Auto Scaling Group |
| AZ | Availability Zone |
| BGP | Border Gateway Protocol |
| CDN | Content Delivery Network |
| CGW | Customer Gateway |
| CIDR | Classless Inter-Domain Routing |
| CLI | Command Line Interface |
| DDoS | Distributed Denial of Service |
| DHCP | Dynamic Host Configuration Protocol |
| DMS | Database Migration Service |
| DNS | Domain Name System |
| DX | Direct Connect |
| EC2 | Elastic Compute Cloud |
| ECS | Elastic Container Service |
| EFS | Elastic File System |
| EIP | Elastic IP |
| EKS | Elastic Kubernetes Service |
| ELB | Elastic Load Balancer |
| ENI | Elastic Network Interface |
| GWLB | Gateway Load Balancer |
| HA | High Availability |
| HTTPS | Hypertext Transfer Protocol Secure |
| IAM | Identity and Access Management |
| ICMP | Internet Control Message Protocol |
| IGW | Internet Gateway |
| IP | Internet Protocol |
| IPS | Intrusion Prevention System |
| IPv4 | Internet Protocol version 4 |
| IPv6 | Internet Protocol version 6 |
| LB | Load Balancer |
| MTU | Maximum Transmission Unit |
| NACL | Network Access Control List |
| NAT | Network Address Translation |
| NLB | Network Load Balancer |
| RDS | Relational Database Service |
| RFC | Request for Comments |
| S3 | Simple Storage Service |
| SDK | Software Development Kit |
| SG | Security Group |
| SMTP | Simple Mail Transfer Protocol |
| SNS | Simple Notification Service |
| SQS | Simple Queue Service |
| SSH | Secure Shell |
| SSL | Secure Sockets Layer |
| TCP | Transmission Control Protocol |
| TGW | Transit Gateway |
| TLS | Transport Layer Security |
| TTL | Time To Live |
| UDP | User Datagram Protocol |
| VGW | Virtual Private Gateway |
| VLAN | Virtual Local Area Network |
| VPC | Virtual Private Cloud |
| VPN | Virtual Private Network |
| WAF | Web Application Firewall |

---

## Network Protocol Numbers

| Protocol | Number | Description |
|----------|--------|-------------|
| ICMP | 1 | Internet Control Message Protocol |
| TCP | 6 | Transmission Control Protocol |
| UDP | 17 | User Datagram Protocol |
| IPv6 | 41 | IPv6 Encapsulation |
| GRE | 47 | Generic Routing Encapsulation |
| ESP | 50 | Encapsulating Security Payload |
| AH | 51 | Authentication Header |
| ICMPv6 | 58 | ICMP for IPv6 |

---

**Keep learning! Understanding these terms is key to mastering AWS networking.** 📖
