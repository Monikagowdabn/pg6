# TASK 1: Deploy an EC2 Instance using CloudFormation

## 📌 Description
This task demonstrates how to deploy an EC2 instance using a CloudFormation Template (CFT). The template provisions an EC2 instance along with a security group that allows SSH and HTTP access.

---

## 🧾 CloudFormation Template

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Simple EC2 Instance Deployment using CloudFormation

# ---------------- PARAMETERS ----------------
Parameters:

  KeyName:
    Description: Name of an existing EC2 KeyPair for SSH access
    Type: AWS::EC2::KeyPair::KeyName

  InstanceType:
    Description: EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t3.micro
      - t3.small

  LatestAmiId:
    Description: Latest Amazon Linux 2 AMI
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

# ---------------- RESOURCES ----------------
Resources:

  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      ImageId: !Ref LatestAmiId

      SecurityGroupIds:
        - !Ref InstanceSecurityGroup

      Tags:
        - Key: Name
          Value: MyEC2Instance

  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH and HTTP access
      SecurityGroupIngress:

        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0   # ⚠️ Restrict this in real projects

        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

# ---------------- OUTPUTS ----------------
Outputs:

  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref EC2Instance

  PublicIP:
    Description: Public IP of EC2 instance
    Value: !GetAtt EC2Instance.PublicIp

  PublicDNS:
    Description: Public DNS of EC2 instance
    Value: !GetAtt EC2Instance.PublicDnsName
