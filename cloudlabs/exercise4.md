# Exercise 4: Understanding Multi-Region Deployment

## Goal
Understand how Azure supports multi-region deployments for high availability and disaster recovery.

## Estimated Time
20 minutes

## Steps

### Task 1: Create a Storage Account in a Different Region
1. Go to Azure Portal → **Resource Groups** → click on **RG-02**
2. Click **+ Create** → **Storage Account** → **Create**
3. Fill in:

   | Field | Value |
   |---|---|
   | Resource Group | Select your RG-02 |
   | Storage Account Name | Enter a unique name |
   | Region | **West US** (different from East US) |
   | Performance | Standard |
   | Redundancy | GRS (Geo-Redundant Storage) |

4. Click **Review + Create** → **Create**
<img width="1310" height="908" alt="Screenshot 2026-03-31 132319" src="https://github.com/user-attachments/assets/37a344ff-6e5b-4833-bae9-c77d6a435cad" />

### Task 2: Compare Both Storage Accounts
1. Go to RG-02 → you now have 2 storage accounts
2. Compare them:

   | Property | Storage Account 1 | Storage Account 2 |
   |---|---|---|
   | Region | East US | West US |
   | Replication | LRS | GRS |
   | Endpoint | *.eastus.* | *.westus.* |
<img width="1240" height="905" alt="Screenshot 2026-03-31 132535" src="https://github.com/user-attachments/assets/dc7d917b-cdf9-4792-babb-f16e26fce400" />
<img width="1244" height="839" alt="Screenshot 2026-03-31 132557" src="https://github.com/user-attachments/assets/2dbcfde0-bf98-4789-8746-d9b0df9551e6" />

### Task 3: Understand Azure Paired Regions
1. In Azure Portal search bar, search for **"Azure regions"**
2. Go to [Azure Paired Regions](https://learn.microsoft.com/en-us/azure/reliability/cross-region-replication-azure)
3. Find the pair for **East US** → it pairs with **West US**
4. Understand why paired regions matter:
   - Disaster recovery
   - Data residency compliance
   - Sequential updates


### Task 4: Understand GRS Replication
1. Open the West US storage account
2. Click **Geo-replication** in the left menu
3. See how data is replicated to the paired region automatically

## Key Concepts Learned
- **LRS**: Data replicated 3 times within single datacenter
- **GRS**: Data replicated to a paired region 100s of miles away
- **Paired Regions**: Azure's built-in regional pairs for disaster recovery
- **Multi-region**: Deploying resources across regions for high availability

## Expected Result
✅ Storage account created in West US  
✅ Compared East US and West US storage accounts  
✅ Understood Azure paired regions concept  
✅ Understood GRS vs LRS replication
