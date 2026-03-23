# AWS Cost Estimation 

## Overview
This document provides a cost estimation for various AWS resources deployed in the **N. Virginia region**.  
The calculation considers whether resources are paused and includes per-user and total cost analysis.

---

## Assumptions

- **Lab Duration:** 1440 hours (1 month)
- **VM Uptime:** 50 hours
- **Total Users:** 150
- **Region:** N. Virginia
- **Paused Resource Indicator:**  
  - `1` → Resource paused (no cost or reduced cost)  
  - `0` → Resource active (cost applied)

---

## Resource Cost Breakdown

| Resource                  | Type & Size                          | Paused (1/0) | Cost/Month ($) | Count | Total Cost ($) |
|--------------------------|--------------------------------------|--------------|----------------|-------|----------------|
| EC2 Instance             | c6in.xlarge (4 Core, 8 GiB)          | 1            | 258.71         | 1     | 17.97          |
| ECS (Fargate)            | 5 GB RAM                             | 0            | 0.03           | 1     | 0.06           |
| Public IP                | Dynamic IP                           | 0            | 3.65           | 1     | 7.30           |
| Gateway Load Balancer    | 10 GB/hour                           | 0            | 32.85          | 1     | 65.70          |
| EBS Volume               | gp3 100 GB                           | 0            | 8.00           | 1     | 16.00          |
| S3 Storage               | Standard 50 GB + 100k requests       | 0            | 1.19           | 1     | 2.38           |
| RDS Instance             | db.t3.micro MySQL (20 GB)            | 1            | 14.82          | 1     | 1.03           |

---

## Cost Formula
=IF(C6=1,(D6/720)*$E$3*F6,(D6/720)*$E$2*F6)
for one user
sum(all rows)
for 150 users
=one users total cost*150

<img width="1554" height="685" alt="Screenshot 2026-03-23 094532" src="https://github.com/user-attachments/assets/06fd927b-0c7c-4810-835f-a964602b1813" />
