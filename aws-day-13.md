Task 1 — Deploy VPC + Public EC2 + Private RDS MySQL via CloudFormation
Objective: Provision a production-style VPC with public EC2 and a fully private RDS MySQL instance. EC2 can reach RDS over port 3306; RDS is not internet-accessible. MySQL client installed on EC2 via UserData for direct DB connectivity testing.

CFT: stack2-vpc-ec2-rds-mysql.json
