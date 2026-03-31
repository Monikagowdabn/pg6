# Exercise 3: Explore Virtual Machine Configuration in Azure Portal

## Goal
Understand how to explore and manage Virtual Machine configurations in Microsoft Azure Portal.

## Estimated Time
20 minutes

## Steps

### Task 1: Navigate to Virtual Machines
1. Login to [Azure Portal](https://portal.azure.com) using credentials from the **Environment** tab
2. In the search bar, type **Virtual Machines** and click on it
3. You will see the list of VMs available in your subscription
 
<img width="1917" height="938" alt="Screenshot 2026-03-31 122049" src="https://github.com/user-attachments/assets/492f83e0-b01e-48a5-b3e1-0ec14d63707d" />
<img width="1902" height="879" alt="Screenshot 2026-03-31 125641" src="https://github.com/user-attachments/assets/21a8cd56-298b-4cff-a299-2af451147197" />


### Task 2: Explore VM Overview
1. Click on any VM to open its Overview page
2. Note the following details:

   | Property | Description |
   |---|---|
   | VM Name | Name of the virtual machine |
   | Operating System | Windows or Linux OS version |
   | Location | Azure region where VM is deployed |
   | Size | Number of vCPUs and RAM |
   | Status | Running, Stopped, Deallocated |
   | Public IP | IP address for remote access |
   | DNS Name | Fully qualified domain name |
 <img width="1904" height="866" alt="Screenshot 2026-03-31 125934" src="https://github.com/user-attachments/assets/73429e2f-435b-435a-8722-2a15182af8c1" />
<img width="1159" height="645" alt="Screenshot 2026-03-31 130055" src="https://github.com/user-attachments/assets/8b068647-d76a-484a-a3af-3c5861519cb9" />
<img width="1008" height="318" alt="Screenshot 2026-03-31 130110" src="https://github.com/user-attachments/assets/3715b54a-fd56-4bb6-b42f-91bb9437a90d" />

### Task 3: Explore VM Networking
1. Click **Networking** in the left menu
2. Explore the **Network Interface** attached to the VM
3. Click on the **Network Security Group (NSG)**
4. Review the inbound and outbound security rules
5. Understand how ports control access to the VM:

   | Port | Protocol | Purpose |
   |---|---|---|
   | 3389 | RDP | Remote Desktop access |
   | 22 | SSH | Linux remote access |
   | 80 | HTTP | Web traffic |
   | 443 | HTTPS | Secure web traffic |  
<img width="1913" height="844" alt="Screenshot 2026-03-31 130454" src="https://github.com/user-attachments/assets/15c42228-c28b-411d-9248-b7b1e57ed36b" />
<img width="1904" height="907" alt="Screenshot 2026-03-31 130518" src="https://github.com/user-attachments/assets/43f980ad-cac7-427e-b087-256d22bb6762" />
<img width="1877" height="802" alt="Screenshot 2026-03-31 130556" src="https://github.com/user-attachments/assets/9a70d216-2868-40ca-8533-91706333592c" />

### Task 4: Explore VM Disks
1. Click **Disks** in the left menu
2. Review the following:
   - **OS Disk** — operating system disk
   - **Data Disks** — additional storage disks
   - **Disk Type** — Standard HDD, Standard SSD, Premium SSD
   - **Encryption** — disk encryption settings
    
<img width="1874" height="838" alt="Screenshot 2026-03-31 130332" src="https://github.com/user-attachments/assets/01c94cae-7279-449f-ac70-1ec0980a88fc" />

 <img width="1903" height="588" alt="Screenshot 2026-03-31 130352" src="https://github.com/user-attachments/assets/3bcd846d-8fa1-453f-8d39-21707606c2e7" />
### Task 5: Explore VM Size Options
1. Click **Size** in the left menu
2. Browse the available VM sizes
3. Compare different sizes by:
   - vCPUs
   - RAM (GiB)
   - Data disks
   - Max IOPS
   - Monthly cost
4. Understand the different VM series:

   | Series | Best For |
   |---|---|
   | B-series | Low cost, burstable workloads |
   | D-series | General purpose workloads |
   | F-series | Compute optimized |
   | E-series | Memory optimized |
<img width="1891" height="918" alt="Screenshot 2026-03-31 130739" src="https://github.com/user-attachments/assets/3624622e-2d86-48f2-9d74-9901a11baabb" />

### Task 6: Explore VM Tags
1. Click **Tags** in the left menu
2. Understand how tags help with:
   - Resource organization
   - Cost management
   - Billing tracking
   - Environment identification
     <img width="1882" height="910" alt="Screenshot 2026-03-31 130840" src="https://github.com/user-attachments/assets/2dad80a5-070a-45b9-94eb-aaecfa48f68a" />


## Expected Result
✅ Navigated to Virtual Machines in Azure Portal
✅ Explored VM overview properties
✅ Understood networking and NSG rules
✅ Explored VM disk configuration
✅ Compared different VM sizes and series
✅ Understood the purpose of VM tags
