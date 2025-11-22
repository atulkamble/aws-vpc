# Module 6: Hands-On Labs

## Introduction

This module provides comprehensive hands-on labs to practice VPC concepts. Each lab builds on previous knowledge and includes CloudFormation templates for automation.

## Lab Prerequisites

Before starting the labs, ensure you have:

- ✅ AWS Account with admin or appropriate IAM permissions
- ✅ AWS CLI installed and configured
- ✅ Basic understanding of AWS Console
- ✅ SSH key pair created in your region
- ✅ Text editor for viewing/editing files

## Cost Warning

⚠️ **Important**: These labs will create AWS resources that may incur charges. Always clean up resources after completing labs.

**Estimated Costs per Lab:**
- VPC, Subnets, Route Tables: **Free**
- EC2 Instances: **~$0.0116/hour** (t3.micro)
- NAT Gateway: **~$0.045/hour** + data processing
- VPN Connection: **~$0.05/hour**

**Always delete resources when done!**

## Lab 1: Create a Basic VPC from Scratch

### Objective
Create a VPC with public and private subnets, Internet Gateway, and proper routing.

### Architecture
```
VPC: 10.0.0.0/16
├── Public Subnet: 10.0.1.0/24 (us-east-1a)
├── Private Subnet: 10.0.2.0/24 (us-east-1a)
├── Internet Gateway
├── Public Route Table (route to IGW)
└── Private Route Table (local only)
```

### Manual Steps

#### Step 1: Create VPC
```bash
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=lab1-vpc}]' \
  --query 'Vpc.VpcId' \
  --output text)

echo "VPC ID: $VPC_ID"

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id $VPC_ID \
  --enable-dns-hostnames
```

#### Step 2: Create Internet Gateway
```bash
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=lab1-igw}]' \
  --query 'InternetGateway.InternetGatewayId' \
  --output text)

echo "IGW ID: $IGW_ID"

# Attach to VPC
aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID
```

#### Step 3: Create Subnets
```bash
# Public Subnet
PUBLIC_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab1-public-subnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

echo "Public Subnet ID: $PUBLIC_SUBNET_ID"

# Private Subnet
PRIVATE_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=lab1-private-subnet}]' \
  --query 'Subnet.SubnetId' \
  --output text)

echo "Private Subnet ID: $PRIVATE_SUBNET_ID"

# Enable auto-assign public IP for public subnet
aws ec2 modify-subnet-attribute \
  --subnet-id $PUBLIC_SUBNET_ID \
  --map-public-ip-on-launch
```

#### Step 4: Create Route Tables
```bash
# Public Route Table
PUBLIC_RTB_ID=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=lab1-public-rtb}]' \
  --query 'RouteTable.RouteTableId' \
  --output text)

echo "Public Route Table ID: $PUBLIC_RTB_ID"

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id $PUBLIC_RTB_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# Associate with public subnet
aws ec2 associate-route-table \
  --subnet-id $PUBLIC_SUBNET_ID \
  --route-table-id $PUBLIC_RTB_ID
```

#### Step 5: Create Security Groups
```bash
# Web Server Security Group
WEB_SG_ID=$(aws ec2 create-security-group \
  --group-name lab1-web-sg \
  --description "Web server security group" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

echo "Web SG ID: $WEB_SG_ID"

# Allow HTTP
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow SSH (replace with your IP)
aws ec2 authorize-security-group-ingress \
  --group-id $WEB_SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

#### Step 6: Launch Test Instance
```bash
# Get latest Amazon Linux 2023 AMI
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=al2023-ami-*-x86_64" "Name=state,Values=available" \
  --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
  --output text)

echo "AMI ID: $AMI_ID"

# Launch instance (replace key-name with your key pair)
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t3.micro \
  --key-name YOUR_KEY_NAME \
  --security-group-ids $WEB_SG_ID \
  --subnet-id $PUBLIC_SUBNET_ID \
  --user-data '#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from Lab 1 - $(hostname -f)</h1>" > /var/www/html/index.html' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab1-web-server}]' \
  --query 'Instances[0].InstanceId' \
  --output text)

echo "Instance ID: $INSTANCE_ID"

# Wait for instance to be running
aws ec2 wait instance-running --instance-ids $INSTANCE_ID

# Get public IP
PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text)

echo "Public IP: $PUBLIC_IP"
echo "Access your web server at: http://$PUBLIC_IP"
```

#### Step 7: Test
```bash
# Wait a minute for the web server to start, then:
curl http://$PUBLIC_IP
```

### CloudFormation Template

Save as `lab1-basic-vpc.yaml`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Lab 1 - Basic VPC with public and private subnets'

Parameters:
  KeyName:
    Description: EC2 Key Pair for SSH access
    Type: AWS::EC2::KeyPair::KeyName

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: lab1-vpc

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: lab1-igw

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: lab1-public-subnet

  PrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: lab1-private-subnet

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: lab1-public-rtb

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: lab1-web-sg

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Sub '{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64}}'
      InstanceType: t3.micro
      KeyName: !Ref KeyName
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      SubnetId: !Ref PublicSubnet
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>Hello from Lab 1 - $(hostname -f)</h1>" > /var/www/html/index.html
      Tags:
        - Key: Name
          Value: lab1-web-server

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
  PublicSubnetId:
    Description: Public Subnet ID
    Value: !Ref PublicSubnet
  PrivateSubnetId:
    Description: Private Subnet ID
    Value: !Ref PrivateSubnet
  WebServerPublicIP:
    Description: Web Server Public IP
    Value: !GetAtt WebServer.PublicIp
  WebServerURL:
    Description: Web Server URL
    Value: !Sub 'http://${WebServer.PublicIp}'
```

**Deploy:**
```bash
aws cloudformation create-stack \
  --stack-name lab1-basic-vpc \
  --template-body file://lab1-basic-vpc.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=YOUR_KEY_NAME
```

### Cleanup
```bash
# Manual cleanup
aws ec2 terminate-instances --instance-ids $INSTANCE_ID
aws ec2 wait instance-terminated --instance-ids $INSTANCE_ID
aws ec2 delete-security-group --group-id $WEB_SG_ID
aws ec2 delete-subnet --subnet-id $PUBLIC_SUBNET_ID
aws ec2 delete-subnet --subnet-id $PRIVATE_SUBNET_ID
aws ec2 delete-route-table --route-table-id $PUBLIC_RTB_ID
aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID
aws ec2 delete-vpc --vpc-id $VPC_ID

# CloudFormation cleanup
aws cloudformation delete-stack --stack-name lab1-basic-vpc
```

## Lab 2: Three-Tier Architecture with NAT Gateway

### Objective
Build a production-ready three-tier architecture with web, app, and database layers.

### Architecture
```
VPC: 10.0.0.0/16

Availability Zone 1:
├── Public Subnet: 10.0.1.0/24
│   ├── NAT Gateway
│   └── Application Load Balancer
├── Private App Subnet: 10.0.2.0/24
│   └── Application Servers
└── Private DB Subnet: 10.0.3.0/24
    └── Database Servers

Availability Zone 2:
├── Public Subnet: 10.0.11.0/24
│   ├── NAT Gateway
│   └── Application Load Balancer
├── Private App Subnet: 10.0.12.0/24
│   └── Application Servers
└── Private DB Subnet: 10.0.13.0/24
    └── Database Servers
```

### CloudFormation Template

Save as `lab2-three-tier.yaml`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Lab 2 - Three-Tier Architecture with NAT Gateway'

Parameters:
  KeyName:
    Description: EC2 Key Pair
    Type: AWS::EC2::KeyPair::KeyName

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: lab2-three-tier-vpc

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: lab2-igw

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # Public Subnets
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: lab2-public-subnet-1

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.11.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: lab2-public-subnet-2

  # Private App Subnets
  PrivateAppSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: lab2-private-app-subnet-1

  PrivateAppSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.12.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: lab2-private-app-subnet-2

  # Private DB Subnets
  PrivateDBSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.3.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: lab2-private-db-subnet-1

  PrivateDBSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.13.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: lab2-private-db-subnet-2

  # Elastic IPs for NAT Gateways
  NATGatewayEIP1:
    Type: AWS::EC2::EIP
    DependsOn: AttachGateway
    Properties:
      Domain: vpc
      Tags:
        - Key: Name
          Value: lab2-nat-eip-1

  NATGatewayEIP2:
    Type: AWS::EC2::EIP
    DependsOn: AttachGateway
    Properties:
      Domain: vpc
      Tags:
        - Key: Name
          Value: lab2-nat-eip-2

  # NAT Gateways
  NATGateway1:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NATGatewayEIP1.AllocationId
      SubnetId: !Ref PublicSubnet1
      Tags:
        - Key: Name
          Value: lab2-nat-gateway-1

  NATGateway2:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NATGatewayEIP2.AllocationId
      SubnetId: !Ref PublicSubnet2
      Tags:
        - Key: Name
          Value: lab2-nat-gateway-2

  # Public Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: lab2-public-rtb

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable

  PublicSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable

  # Private Route Tables
  PrivateRouteTable1:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: lab2-private-rtb-1

  PrivateRoute1:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NATGateway1

  PrivateAppSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateAppSubnet1
      RouteTableId: !Ref PrivateRouteTable1

  PrivateDBSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateDBSubnet1
      RouteTableId: !Ref PrivateRouteTable1

  PrivateRouteTable2:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: lab2-private-rtb-2

  PrivateRoute2:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NATGateway2

  PrivateAppSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateAppSubnet2
      RouteTableId: !Ref PrivateRouteTable2

  PrivateDBSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateDBSubnet2
      RouteTableId: !Ref PrivateRouteTable2

  # Security Groups
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: ALB Security Group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: lab2-alb-sg

  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: App Server Security Group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          SourceSecurityGroupId: !Ref ALBSecurityGroup
      Tags:
        - Key: Name
          Value: lab2-app-sg

  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Database Security Group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref AppSecurityGroup
      Tags:
        - Key: Name
          Value: lab2-db-sg

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
  PublicSubnets:
    Description: Public Subnets
    Value: !Join [',', [!Ref PublicSubnet1, !Ref PublicSubnet2]]
  PrivateAppSubnets:
    Description: Private App Subnets
    Value: !Join [',', [!Ref PrivateAppSubnet1, !Ref PrivateAppSubnet2]]
  PrivateDBSubnets:
    Description: Private DB Subnets
    Value: !Join [',', [!Ref PrivateDBSubnet1, !Ref PrivateDBSubnet2]]
```

**Deploy:**
```bash
aws cloudformation create-stack \
  --stack-name lab2-three-tier \
  --template-body file://lab2-three-tier.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=YOUR_KEY_NAME
```

### Testing

1. Deploy the stack
2. Wait for completion (5-10 minutes)
3. Review created resources in console
4. Test connectivity between tiers
5. Verify private instances can reach internet via NAT Gateway

### Cleanup
```bash
aws cloudformation delete-stack --stack-name lab2-three-tier
```

## Additional Lab Resources

Additional labs provided in separate files:

1. **Lab 3: VPC Peering** - `lab3-vpc-peering.yaml`
2. **Lab 4: VPN Connection** - `lab4-vpn-connection.yaml`
3. **Lab 5: VPC Endpoints** - `lab5-vpc-endpoints.yaml`
4. **Lab 6: Flow Logs Analysis** - `lab6-flow-logs.yaml`

## Lab Summary

| Lab | Duration | Difficulty | Cost/Hour | Key Concepts |
|-----|----------|------------|-----------|--------------|
| Lab 1 | 30 min | Beginner | ~$0.01 | VPC, Subnets, IGW |
| Lab 2 | 60 min | Intermediate | ~$0.10 | NAT Gateway, Multi-AZ |
| Lab 3 | 45 min | Intermediate | ~$0.02 | VPC Peering |
| Lab 4 | 60 min | Advanced | ~$0.10 | VPN, Hybrid |
| Lab 5 | 30 min | Intermediate | ~$0.01 | VPC Endpoints |
| Lab 6 | 45 min | Intermediate | ~$0.01 | Flow Logs, Athena |

## Common Issues and Solutions

### Issue: CloudFormation Stack Failed

**Solution:**
1. Check CloudFormation events for error message
2. Common issues:
   - Key pair doesn't exist
   - Insufficient permissions
   - Resource limits reached

### Issue: Cannot Connect to Instance

**Solution:**
1. Verify security group allows your IP
2. Check instance has public IP (for public subnet)
3. Verify route table has route to IGW
4. Check NACL rules

### Issue: Private Instance Can't Reach Internet

**Solution:**
1. Verify NAT Gateway is in public subnet
2. Check NAT Gateway state is "available"
3. Verify private subnet route table has route to NAT Gateway
4. Check security groups allow outbound traffic

## Next Steps

After completing these labs:

1. ✅ Experiment with modifications
2. ✅ Combine concepts (e.g., VPC Peering + VPC Endpoints)
3. ✅ Build your own architectures
4. ✅ Practice troubleshooting
5. ✅ Explore AWS Well-Architected Framework

## Additional Resources

- [AWS VPC Workshop](https://catalog.workshops.aws/)
- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/)
- [AWS CLI Reference](https://docs.aws.amazon.com/cli/)
- [AWS VPC Examples](https://github.com/aws-samples/)

## Course Completion

🎉 **Congratulations!** You've completed the AWS VPC Full Course!

You now have the knowledge to:
- ✅ Design and implement AWS VPCs
- ✅ Configure routing and connectivity
- ✅ Secure your network infrastructure
- ✅ Connect VPCs and on-premises networks
- ✅ Use advanced VPC features
- ✅ Troubleshoot networking issues

### Recommended Next Steps

1. **AWS Certification**: Consider AWS Certified Solutions Architect
2. **Real Project**: Build a production VPC for a real application
3. **Explore**: AWS Transit Gateway, Network Firewall
4. **Learn More**: Kubernetes networking on EKS, Service Mesh
5. **Stay Updated**: Follow AWS networking announcements

## Feedback

We'd love your feedback! What worked well? What could be improved?

**Happy Networking! 🚀**
