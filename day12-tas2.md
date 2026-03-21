# TASK 2: Create EC2 Instance and S3 Bucket with Least Privilege Access

##  Description
This task demonstrates how to create an EC2 instance and an S3 bucket using a CloudFormation Template (CFT), while granting **least privilege access** to the EC2 instance. The EC2 instance is allowed to **only list objects** in the S3 bucket.

---

##  Steps to Deploy (CloudFormation Console)

1. Go to **CloudFormation Console**
2. Click **Create Stack → With new resources**
3. Choose **Upload a template file**
4. Upload the YAML file
5. Click **Next**
6. Enter required details:
   - **KeyPair Name**
   - **Instance Type (optional)**
7. Click **Next → Next → Create Stack**

---

## 🧾 CloudFormation Template

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: EC2 with least privilege access to list S3 bucket objects

# ---------------- PARAMETERS ----------------
Parameters:

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 KeyPair for SSH access

  InstanceType:
    Type: String
    Default: t2.micro

# ---------------- RESOURCES ----------------
Resources:

# -------- S3 Bucket --------
  MyS3Bucket:
    Type: AWS::S3::Bucket

# -------- IAM Role --------
  EC2Role:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: sts:AssumeRole

# -------- IAM Policy (Least Privilege) --------
  EC2S3Policy:
    Type: AWS::IAM::Policy
    Properties:
      PolicyName: EC2ListS3Policy
      Roles:
        - !Ref EC2Role
      PolicyDocument:
        Version: "2012-10-17"
        Statement:

          # Allow listing bucket
          - Effect: Allow
            Action:
              - s3:ListBucket
            Resource: !GetAtt MyS3Bucket.Arn

# -------- Instance Profile --------
  EC2InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Roles:
        - !Ref EC2Role

# -------- Security Group --------
  EC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0  # ⚠️ Restrict in real usage

# -------- EC2 Instance --------
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      IamInstanceProfile: !Ref EC2InstanceProfile

      ImageId: ami-0f58b397bc5c1f2e8  # Amazon Linux (update per region)

      SecurityGroupIds:
        - !Ref EC2SecurityGroup

      Tags:
        - Key: Name
          Value: EC2-S3-LeastPrivilege

# ---------------- OUTPUTS ----------------
Outputs:

  BucketName:
    Value: !Ref MyS3Bucket

  InstanceId:
    Value: !Ref EC2Instance
