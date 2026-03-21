# AWS Nested CloudFormation Stacks
> **Parent Stack → Network • Compute • ALB • Data**

ParentStack
├── NetworkStack
│   ├── VPC
│   ├── Public Subnet AZ-1  &  Public Subnet AZ-2
│   ├── Private Subnet AZ-1  &  Private Subnet AZ-2
│   ├── Internet Gateway
│   ├── NAT Gateway (per AZ)
│   └── Security Groups (ALB SG, App SG, DB SG)
│
├── ComputeStack
│   ├── Launch Template (Nginx)
│   ├── Auto Scaling Group
│   ├── Scaling Policies (CPU Target Tracking)
│   └── IAM Role + Instance Profile (SSM Agent)
│
├── ALBStack
│   ├── Application Load Balancer
│   ├── Target Group
│   ├── Listener (HTTP :80)
│   ├── ALB Security Group
│   └── Health Check Configuration
│
└── DataStack
    ├── S3 Bucket (Static Website)
    ├── Bucket Policy (Public Read)
    ├── Website Configuration
    └── CORS Configuration
```

---

## Parent Stack

The parent stack orchestrates all child stacks using `AWS::CloudFormation::Stack` nested resources. It passes shared parameters (VPC ID, Subnet IDs, etc.) as outputs from one child stack into the inputs of the next.

### parent-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Parent stack - orchestrates Network, Compute, ALB and Data stacks

Parameters:
  EnvironmentName:
    Type: String
    Default: production
    AllowedValues: [dev, staging, production]
    Description: Environment name prefix for all resources

  TemplatesBucketURL:
    Type: String
    Description: S3 URL where child stack templates are stored
    # e.g. https://s3.amazonaws.com/my-cfn-templates

  NginxAMI:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
    Description: Latest Amazon Linux 2 AMI via SSM Parameter Store

  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues: [t3.micro, t3.small, t3.medium, t3.large]

  MinInstances:
    Type: Number
    Default: 1

  MaxInstances:
    Type: Number
    Default: 4

  DesiredInstances:
    Type: Number
    Default: 2

Resources:

  # ── 1. NETWORK STACK ──────────────────────────────────────────────────────
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub "${TemplatesBucketURL}/network-stack.yaml"
      Parameters:
        EnvironmentName: !Ref EnvironmentName
      Tags:
        - Key: StackLayer
          Value: Network

  # ── 2. COMPUTE STACK ──────────────────────────────────────────────────────
  ComputeStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: NetworkStack
    Properties:
      TemplateURL: !Sub "${TemplatesBucketURL}/compute-stack.yaml"
      Parameters:
        EnvironmentName:   !Ref EnvironmentName
        VpcId:             !GetAtt NetworkStack.Outputs.VpcId
        PrivateSubnet1Id:  !GetAtt NetworkStack.Outputs.PrivateSubnet1Id
        PrivateSubnet2Id:  !GetAtt NetworkStack.Outputs.PrivateSubnet2Id
        AppSecurityGroup:  !GetAtt NetworkStack.Outputs.AppSecurityGroupId
        NginxAMI:          !Ref NginxAMI
        InstanceType:      !Ref InstanceType
        MinInstances:      !Ref MinInstances
        MaxInstances:      !Ref MaxInstances
        DesiredInstances:  !Ref DesiredInstances
      Tags:
        - Key: StackLayer
          Value: Compute

  # ── 3. ALB STACK ──────────────────────────────────────────────────────────
  ALBStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: ComputeStack
    Properties:
      TemplateURL: !Sub "${TemplatesBucketURL}/alb-stack.yaml"
      Parameters:
        EnvironmentName:   !Ref EnvironmentName
        VpcId:             !GetAtt NetworkStack.Outputs.VpcId
        PublicSubnet1Id:   !GetAtt NetworkStack.Outputs.PublicSubnet1Id
        PublicSubnet2Id:   !GetAtt NetworkStack.Outputs.PublicSubnet2Id
        AutoScalingGroup:  !GetAtt ComputeStack.Outputs.AutoScalingGroupName
      Tags:
        - Key: StackLayer
          Value: ALB

  # ── 4. DATA STACK ─────────────────────────────────────────────────────────
  DataStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub "${TemplatesBucketURL}/data-stack.yaml"
      Parameters:
        EnvironmentName: !Ref EnvironmentName
      Tags:
        - Key: StackLayer
          Value: Data

Outputs:
  ALBDNSName:
    Description: Public DNS of the Application Load Balancer
    Value: !GetAtt ALBStack.Outputs.ALBDNSName

  StaticWebsiteURL:
    Description: S3 static website endpoint
    Value: !GetAtt DataStack.Outputs.WebsiteURL

  VpcId:
    Description: VPC ID created by Network Stack
    Value: !GetAtt NetworkStack.Outputs.VpcId
```

---

## Network Stack

Provisions the full network layer across **2 Availability Zones** — public and private subnets, an Internet Gateway, NAT Gateways for private subnet egress, route tables, and all Security Groups used by the other stacks.

### network-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Network Stack - VPC, Subnets (2 AZs), NAT Gateways, Security Groups

Parameters:
  EnvironmentName:
    Type: String

Mappings:
  SubnetConfig:
    VPC:          { CIDR: "10.0.0.0/16"  }
    PublicAZ1:    { CIDR: "10.0.1.0/24"  }
    PublicAZ2:    { CIDR: "10.0.2.0/24"  }
    PrivateAZ1:   { CIDR: "10.0.10.0/24" }
    PrivateAZ2:   { CIDR: "10.0.20.0/24" }

Resources:

  # ── VPC ───────────────────────────────────────────────────────────────────
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !FindInMap [SubnetConfig, VPC, CIDR]
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-vpc"

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-igw"

  IGWAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # ── PUBLIC SUBNETS ────────────────────────────────────────────────────────
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !FindInMap [SubnetConfig, PublicAZ1, CIDR]
      AvailabilityZone: !Select [0, !GetAZs ""]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-public-subnet-az1"

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !FindInMap [SubnetConfig, PublicAZ2, CIDR]
      AvailabilityZone: !Select [1, !GetAZs ""]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-public-subnet-az2"

  # ── PRIVATE SUBNETS ───────────────────────────────────────────────────────
  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !FindInMap [SubnetConfig, PrivateAZ1, CIDR]
      AvailabilityZone: !Select [0, !GetAZs ""]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-private-subnet-az1"

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !FindInMap [SubnetConfig, PrivateAZ2, CIDR]
      AvailabilityZone: !Select [1, !GetAZs ""]
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-private-subnet-az2"

  # ── NAT GATEWAYS (one per AZ for high availability) ──────────────────────
  EIP1:
    Type: AWS::EC2::EIP
    DependsOn: IGWAttachment
    Properties:
      Domain: vpc

  EIP2:
    Type: AWS::EC2::EIP
    DependsOn: IGWAttachment
    Properties:
      Domain: vpc

  NatGateway1:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt EIP1.AllocationId
      SubnetId: !Ref PublicSubnet1
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-nat-az1"

  NatGateway2:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt EIP2.AllocationId
      SubnetId: !Ref PublicSubnet2
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-nat-az2"

  # ── ROUTE TABLES ──────────────────────────────────────────────────────────
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-public-rt"

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: IGWAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnet1RouteAssoc:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable

  PublicSubnet2RouteAssoc:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable

  PrivateRouteTable1:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-private-rt-az1"

  PrivateRoute1:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway1

  PrivateSubnet1RouteAssoc:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateSubnet1
      RouteTableId: !Ref PrivateRouteTable1

  PrivateRouteTable2:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-private-rt-az2"

  PrivateRoute2:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway2

  PrivateSubnet2RouteAssoc:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PrivateSubnet2
      RouteTableId: !Ref PrivateRouteTable2

  # ── SECURITY GROUPS ───────────────────────────────────────────────────────
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP/HTTPS from internet to ALB
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: HTTP from internet
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
          Description: HTTPS from internet
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-alb-sg"

  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow traffic from ALB to Nginx app instances
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          SourceSecurityGroupId: !Ref ALBSecurityGroup
          Description: HTTP from ALB only
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-app-sg"

  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow DB access from App instances only
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref AppSecurityGroup
          Description: MySQL from App SG only
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-db-sg"

Outputs:
  VpcId:
    Value: !Ref VPC
    Export:
      Name: !Sub "${EnvironmentName}-VpcId"

  PublicSubnet1Id:
    Value: !Ref PublicSubnet1
    Export:
      Name: !Sub "${EnvironmentName}-PublicSubnet1Id"

  PublicSubnet2Id:
    Value: !Ref PublicSubnet2
    Export:
      Name: !Sub "${EnvironmentName}-PublicSubnet2Id"

  PrivateSubnet1Id:
    Value: !Ref PrivateSubnet1
    Export:
      Name: !Sub "${EnvironmentName}-PrivateSubnet1Id"

  PrivateSubnet2Id:
    Value: !Ref PrivateSubnet2
    Export:
      Name: !Sub "${EnvironmentName}-PrivateSubnet2Id"

  ALBSecurityGroupId:
    Value: !Ref ALBSecurityGroup
    Export:
      Name: !Sub "${EnvironmentName}-ALBSecurityGroupId"

  AppSecurityGroupId:
    Value: !Ref AppSecurityGroup
    Export:
      Name: !Sub "${EnvironmentName}-AppSecurityGroupId"

  DBSecurityGroupId:
    Value: !Ref DBSecurityGroup
    Export:
      Name: !Sub "${EnvironmentName}-DBSecurityGroupId"
```

---

## Compute Stack

Provisions the Nginx application layer — a **Launch Template** used by an **Auto Scaling Group** across both private subnets. The EC2 instances have an **IAM Instance Profile** with the `AmazonSSMManagedInstanceCore` policy so they are reachable via AWS Systems Manager Session Manager without needing SSH or a bastion host.

### compute-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Compute Stack - Launch Template, Auto Scaling Group, IAM Role with SSM

Parameters:
  EnvironmentName:    { Type: String }
  VpcId:              { Type: AWS::EC2::VPC::Id }
  PrivateSubnet1Id:   { Type: AWS::EC2::Subnet::Id }
  PrivateSubnet2Id:   { Type: AWS::EC2::Subnet::Id }
  AppSecurityGroup:   { Type: AWS::EC2::SecurityGroup::Id }
  NginxAMI:           { Type: AWS::EC2::Image::Id }
  InstanceType:       { Type: String, Default: t3.micro }
  MinInstances:       { Type: Number, Default: 1 }
  MaxInstances:       { Type: Number, Default: 4 }
  DesiredInstances:   { Type: Number, Default: 2 }

Resources:

  # ── IAM ROLE FOR SSM AGENT ────────────────────────────────────────────────
  EC2SSMRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub "${EnvironmentName}-ec2-ssm-role"
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-ec2-ssm-role"

  EC2InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName: !Sub "${EnvironmentName}-ec2-instance-profile"
      Roles:
        - !Ref EC2SSMRole

  # ── LAUNCH TEMPLATE ───────────────────────────────────────────────────────
  NginxLaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: !Sub "${EnvironmentName}-nginx-lt"
      LaunchTemplateData:
        ImageId: !Ref NginxAMI
        InstanceType: !Ref InstanceType
        IamInstanceProfile:
          Arn: !GetAtt EC2InstanceProfile.Arn
        SecurityGroupIds:
          - !Ref AppSecurityGroup
        # No KeyPair needed — SSM Session Manager provides shell access
        UserData:
          Fn::Base64: !Sub |
            #!/bin/bash
            set -e
            yum update -y

            # Install and start Nginx
            amazon-linux-extras install nginx1 -y
            systemctl start nginx
            systemctl enable nginx

            # Install SSM Agent (pre-installed on Amazon Linux 2 but ensure latest)
            yum install -y amazon-ssm-agent
            systemctl enable amazon-ssm-agent
            systemctl start amazon-ssm-agent

            # Custom index page
            echo "<h1>Hello from ${EnvironmentName} - $(hostname -f)</h1>" \
              > /usr/share/nginx/html/index.html
        TagSpecifications:
          - ResourceType: instance
            Tags:
              - Key: Name
                Value: !Sub "${EnvironmentName}-nginx-instance"
              - Key: Environment
                Value: !Ref EnvironmentName

  # ── AUTO SCALING GROUP ────────────────────────────────────────────────────
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AutoScalingGroupName: !Sub "${EnvironmentName}-asg"
      VPCZoneIdentifier:
        - !Ref PrivateSubnet1Id
        - !Ref PrivateSubnet2Id
      LaunchTemplate:
        LaunchTemplateId: !Ref NginxLaunchTemplate
        Version: !GetAtt NginxLaunchTemplate.LatestVersionNumber
      MinSize: !Ref MinInstances
      MaxSize: !Ref MaxInstances
      DesiredCapacity: !Ref DesiredInstances
      HealthCheckType: ELB
      HealthCheckGracePeriod: 120
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-asg"
          PropagateAtLaunch: true

  # ── SCALING POLICY (CPU Target Tracking) ─────────────────────────────────
  CPUScalingPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        TargetValue: 60.0      # Scale out when average CPU > 60%
        DisableScaleIn: false

Outputs:
  AutoScalingGroupName:
    Value: !Ref AutoScalingGroup
    Export:
      Name: !Sub "${EnvironmentName}-ASGName"

  LaunchTemplateId:
    Value: !Ref NginxLaunchTemplate
    Export:
      Name: !Sub "${EnvironmentName}-LaunchTemplateId"

  EC2SSMRoleArn:
    Value: !GetAtt EC2SSMRole.Arn
    Export:
      Name: !Sub "${EnvironmentName}-EC2SSMRoleArn"
```

### SSM Session Manager — Connect to Private Instances

Because the IAM role includes `AmazonSSMManagedInstanceCore`, there is no need for SSH keys or a bastion host:

```bash
# List all managed instances
aws ssm describe-instance-information --region us-east-1

# Start a session (replaces SSH)
aws ssm start-session \
  --target i-0abc123def456 \
  --region us-east-1
```

---

## ALB Stack

Provisions the **Application Load Balancer** in the public subnets, a **Target Group** pointing at the Auto Scaling Group instances on port 80, and an **HTTP Listener**. Health checks ensure traffic is only routed to healthy Nginx instances.

### alb-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: ALB Stack - Application Load Balancer, Target Group, Listener, Health Checks

Parameters:
  EnvironmentName:   { Type: String }
  VpcId:             { Type: AWS::EC2::VPC::Id }
  PublicSubnet1Id:   { Type: AWS::EC2::Subnet::Id }
  PublicSubnet2Id:   { Type: AWS::EC2::Subnet::Id }
  AutoScalingGroup:  { Type: String }

Resources:

  # ── ALB SECURITY GROUP ────────────────────────────────────────────────────
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: ALB - allow HTTP and HTTPS from internet
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-alb-sg"

  # ── APPLICATION LOAD BALANCER ─────────────────────────────────────────────
  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Sub "${EnvironmentName}-alb"
      Scheme: internet-facing
      Type: application
      Subnets:
        - !Ref PublicSubnet1Id
        - !Ref PublicSubnet2Id
      SecurityGroups:
        - !Ref ALBSecurityGroup
      LoadBalancerAttributes:
        - Key: idle_timeout.timeout_seconds
          Value: "60"
        - Key: deletion_protection.enabled
          Value: "false"
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-alb"

  # ── TARGET GROUP ──────────────────────────────────────────────────────────
  NginxTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: !Sub "${EnvironmentName}-nginx-tg"
      VpcId: !Ref VpcId
      Protocol: HTTP
      Port: 80
      TargetType: instance
      HealthCheckEnabled: true
      HealthCheckProtocol: HTTP
      HealthCheckPort: traffic-port
      HealthCheckPath: /
      HealthCheckIntervalSeconds: 30
      HealthCheckTimeoutSeconds: 5
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 3
      Matcher:
        HttpCode: "200"
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-nginx-tg"

  # ── HTTP LISTENER ─────────────────────────────────────────────────────────
  HTTPListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref ApplicationLoadBalancer
      Protocol: HTTP
      Port: 80
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref NginxTargetGroup

  # ── ATTACH ASG TO TARGET GROUP ────────────────────────────────────────────
  ASGTargetGroupAttachment:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      TargetGroupARNs:
        - !Ref NginxTargetGroup

Outputs:
  ALBDNSName:
    Description: DNS name of the Application Load Balancer
    Value: !GetAtt ApplicationLoadBalancer.DNSName
    Export:
      Name: !Sub "${EnvironmentName}-ALBDNSName"

  ALBArn:
    Value: !Ref ApplicationLoadBalancer
    Export:
      Name: !Sub "${EnvironmentName}-ALBArn"

  TargetGroupArn:
    Value: !Ref NginxTargetGroup
    Export:
      Name: !Sub "${EnvironmentName}-TargetGroupArn"
```

### Health Check Configuration

| Property | Value | Description |
|---|---|---|
| Path | `/` | Nginx default page |
| Protocol | HTTP | Plain HTTP health check |
| Interval | 30s | Time between checks |
| Timeout | 5s | Max wait for response |
| Healthy threshold | 2 | Consecutive passes to mark healthy |
| Unhealthy threshold | 3 | Consecutive failures to mark unhealthy |
| Success codes | `200` | Expected HTTP response code |

---

## Data Stack

Provisions an **S3 bucket** configured for **static website hosting**, with a public-read **bucket policy**. Suitable for hosting frontend assets (HTML, CSS, JS) served directly from S3.

### data-stack.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Data Stack - S3 Bucket for Static Website Hosting

Parameters:
  EnvironmentName:
    Type: String

Resources:

  # ── S3 BUCKET ─────────────────────────────────────────────────────────────
  StaticWebsiteBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${EnvironmentName}-static-website-${AWS::AccountId}"
      # Block public access settings — must be false to allow public website
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
      CorsConfiguration:
        CorsRules:
          - AllowedHeaders: ["*"]
            AllowedMethods: [GET]
            AllowedOrigins: ["*"]
            MaxAge: 3600
      VersioningConfiguration:
        Status: Enabled
      LifecycleConfiguration:
        Rules:
          - Id: DeleteOldVersions
            Status: Enabled
            NoncurrentVersionExpiration:
              NoncurrentDays: 30
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-static-website"
        - Key: Environment
          Value: !Ref EnvironmentName

  # ── BUCKET POLICY (Public Read) ───────────────────────────────────────────
  StaticWebsiteBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref StaticWebsiteBucket
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Sid: PublicReadGetObject
            Effect: Allow
            Principal: "*"
            Action: s3:GetObject
            Resource: !Sub "arn:aws:s3:::${StaticWebsiteBucket}/*"

Outputs:
  BucketName:
    Description: S3 bucket name
    Value: !Ref StaticWebsiteBucket
    Export:
      Name: !Sub "${EnvironmentName}-StaticBucketName"

  WebsiteURL:
    Description: S3 static website endpoint URL
    Value: !GetAtt StaticWebsiteBucket.WebsiteURL
    Export:
      Name: !Sub "${EnvironmentName}-WebsiteURL"

  BucketArn:
    Value: !GetAtt StaticWebsiteBucket.Arn
    Export:
      Name: !Sub "${EnvironmentName}-StaticBucketArn"
```

### Upload Static Files

```bash
# Upload website files to S3
aws s3 sync ./website/ s3://${ENVIRONMENT_NAME}-static-website-${ACCOUNT_ID}/ \
  --delete \
  --region us-east-1

# Verify website is accessible
aws s3 website s3://${ENVIRONMENT_NAME}-static-website-${ACCOUNT_ID}/ \
  --index-document index.html \
  --error-document error.html
```

---

## Deployment Guide

### Prerequisites

```bash
# Install/update AWS CLI
aws --version

# Configure credentials
aws configure

# Create S3 bucket to store CloudFormation templates
aws s3 mb s3://my-cfn-templates-bucket --region us-east-1

# Upload all child stack templates
aws s3 cp network-stack.yaml  s3://my-cfn-templates-bucket/
aws s3 cp compute-stack.yaml  s3://my-cfn-templates-bucket/
aws s3 cp alb-stack.yaml      s3://my-cfn-templates-bucket/
aws s3 cp data-stack.yaml     s3://my-cfn-templates-bucket/
```

### Deploy the Parent Stack

```bash
aws cloudformation create-stack \
  --stack-name production-parent-stack \
  --template-body file://parent-stack.yaml \
  --parameters \
    ParameterKey=EnvironmentName,ParameterValue=production \
    ParameterKey=TemplatesBucketURL,ParameterValue=https://s3.amazonaws.com/my-cfn-templates-bucket \
    ParameterKey=InstanceType,ParameterValue=t3.micro \
    ParameterKey=MinInstances,ParameterValue=1 \
    ParameterKey=MaxInstances,ParameterValue=4 \
    ParameterKey=DesiredInstances,ParameterValue=2 \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

### Monitor Deployment

```bash
# Watch stack events in real time
aws cloudformation describe-stack-events \
  --stack-name production-parent-stack \
  --region us-east-1 \
  --query 'StackEvents[*].[Timestamp,LogicalResourceId,ResourceStatus]' \
  --output table

# Check overall stack status
aws cloudformation describe-stacks \
  --stack-name production-parent-stack \
  --query 'Stacks[0].StackStatus'
```

### Update the Stack

```bash
aws cloudformation update-stack \
  --stack-name production-parent-stack \
  --template-body file://parent-stack.yaml \
  --parameters \
    ParameterKey=EnvironmentName,UsePreviousValue=true \
    ParameterKey=TemplatesBucketURL,UsePreviousValue=true \
    ParameterKey=DesiredInstances,ParameterValue=3 \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

### Teardown

```bash
# Empty the S3 bucket first (CloudFormation cannot delete non-empty buckets)
aws s3 rm s3://production-static-website-<ACCOUNT_ID>/ --recursive

# Delete the parent stack (automatically deletes all child stacks)
aws cloudformation delete-stack \
  --stack-name production-parent-stack \
  --region us-east-1
```

---

## Stack Outputs & Cross-Stack References

The parent stack wires child stack outputs into the next child's inputs via `!GetAtt StackName.Outputs.OutputKey`. Here is the complete dependency and data flow:

```
NetworkStack
    │
    ├── Outputs.VpcId              ──► ComputeStack.VpcId
    │                              ──► ALBStack.VpcId
    │
    ├── Outputs.PrivateSubnet1Id   ──► ComputeStack.PrivateSubnet1Id
    ├── Outputs.PrivateSubnet2Id   ──► ComputeStack.PrivateSubnet2Id
    │
    ├── Outputs.PublicSubnet1Id    ──► ALBStack.PublicSubnet1Id
    ├── Outputs.PublicSubnet2Id    ──► ALBStack.PublicSubnet2Id
    │
    └── Outputs.AppSecurityGroupId ──► ComputeStack.AppSecurityGroup

ComputeStack
    └── Outputs.AutoScalingGroupName ──► ALBStack.AutoScalingGroup
```

### Summary of All Stack Outputs

| Stack | Output Key | Used By |
|---|---|---|
| Network | `VpcId` | Compute, ALB |
| Network | `PublicSubnet1Id` / `PublicSubnet2Id` | ALB |
| Network | `PrivateSubnet1Id` / `PrivateSubnet2Id` | Compute |
| Network | `AppSecurityGroupId` | Compute |
| Network | `ALBSecurityGroupId` | ALB |
| Compute | `AutoScalingGroupName` | ALB |
| Compute | `LaunchTemplateId` | — |
| ALB | `ALBDNSName` | Parent Output |
| ALB | `TargetGroupArn` | — |
| Data | `WebsiteURL` | Parent Output |
| Data | `BucketName` | Upload scripts |
