# 🌐 Contoso Global Solutions  
## Multi-Region Deployment Lab – CloudLabs Guide

---

## 📘 Overview

This lab is designed to provide hands-on experience with **Microsoft Azure multi-region deployments** using the CloudLabs platform.

Participants will work in an isolated environment where infrastructure is automatically provisioned using **ARM templates**.

---

## 🏗️ Architecture Summary

Each participant environment is automatically created with:

- **3 Resource Groups**
- **1 Jump VM (browser-accessible)**
- **1 Pre-deployed Storage Account**
- **User-controlled sandbox environment**

---

## ☁️ CloudLabs Configuration

| Setting | Value |
|--------|------|
| Cloud Platform | Microsoft Azure |
| Subscription Type | Dedicated Subscription |
| Deployment Plan | 3 Resource Groups |
| Regions | East US, Central US |
| Max Users | 20 |
| Duration | 14 Days (20,160 minutes) |
| Attempts | 1 |
| Access Type | Browser-based VM (RDP over HTTP) |

---

## 📦 Resource Group Details

### 🔹 RG1 – Jump VM
- Contains the **Virtual Machine**
- Used for accessing the lab environment
- Includes:
  - VM
  - Public IP
  - Network Security Group
  - Virtual Network

---

### 🔹 RG2 – Sandbox
- Empty resource group
- Users can:
  - Create resources
  - Test deployments
  - Practice Azure services

---

### 🔹 RG3 – Pre-Deployed Resources
- Deployed using **ARM Template**
- Contains:
  - Azure Storage Account (StorageV2)
- Used for exploration and management

---

## 🖥️ Virtual Machine (Jump VM)

- OS: Windows Server
- Access: Browser-based RDP
- No local setup required
- Used to:
  - Access Azure Portal
  - Perform lab tasks
  - Run commands/tools

---

## 🔐 Permissions & Access Control

Participants are assigned:

- **Contributor Role on RG2**
- **Contributor Role on RG3**

This allows:
- Creating, modifying, deleting resources
- Full control within assigned resource groups

---

## ⚙️ Deployment Details

- Infrastructure is deployed using **ARM Templates**
- Templates are stored in **Azure Blob Storage**
- CloudLabs triggers deployment automatically per user

### ARM Template Includes:
- Storage Account (RG3)
- VM + Networking (RG1)

---

## 🌍 Multi-Region Concept

This lab uses multiple regions:
- **East US**
- **Central US**

### Why Multi-Region?
- High Availability
- Disaster Recovery
- Better Performance (low latency)
- Fault Isolation

---

## 🚀 User Workflow

1. Register using CloudLabs link
2. Launch lab environment
3. Connect to Jump VM (browser)
4. Open Azure Portal inside VM
5. Perform exercises:
   - Explore RGs
   - Check Storage Account
   - Create resources in RG2

---

## 🧠 Key Learning Outcomes

- Azure Resource Group structure
- ARM Template-based deployments
- Role-Based Access Control (RBAC)
- Virtual Machine access via browser
- Multi-region deployment fundamentals

---

## ⚠️ Important Guidelines

- Do NOT delete resource groups
- Use only RG2 for creating resources
- Ensure resource names are unique
- Lab expires after 2 weeks
- Only one attempt allowed

---

## 🛠️ Troubleshooting

| Issue | Solution |
|------|---------|
| VM not loading | Refresh browser |
| Login failed | Verify credentials |
| Resource not visible | Refresh Azure Portal |
| Deployment error | Use unique resource names |
| Permission denied | Check RG2 / RG3 access |

---

## 📊 Summary

This lab provides a **safe, pre-configured Azure environment** where users can:
- Learn by doing
- Explore real cloud resources
- Understand multi-region architecture

---

## 🎯 Conclusion

By completing this lab, participants gain practical experience in:
- Managing Azure resources
- Working with cloud infrastructure
- Understanding distributed systems

---

## 🚀 Next Steps

- Explore Azure Virtual Machines
- Learn Azure Networking
- Practice ARM Templates
- Prepare for AZ-900 Certification

---

## 🎉 Completion

You have successfully completed the **Contoso Multi-Region Deployment Lab**!
