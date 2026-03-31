# Exercise 1: Explore Your Resource Groups

## Goal
Understand the pre-created resource groups in your lab environment.

## Estimated Time
10 minutes

## Steps

### Task 1: View Resource Groups
1. Login to [Azure Portal](https://portal.azure.com) using credentials from the **Environment** tab
2. In the search bar at the top, type **Resource Groups** and click on it
3. You will see 2 resource groups assigned to you:
   - **ODL-contoso-lab-XXXXXX-02** → Your sandbox resource group
   - **ODL-contoso-lab-XXXXXX-03** → Pre-deployed storage account resource group

### Task 2: Explore RG-02 (Sandbox)
1. Click on your **RG-02** resource group
2. Notice it is currently empty — this is your sandbox to create resources
3. Note the **Location** (East US) shown in the Overview
4. Click **Access Control (IAM)** → Click **Role Assignments**
5. Verify you have **Contributor** role assigned to your user

### Task 3: Explore RG-03 (Pre-deployed Storage)
1. Go back to Resource Groups
2. Click on your **RG-03** resource group
3. You should see a **Storage Account** already deployed here
4. Click on it to explore its properties
5. Note the **Storage Account Name** — it matches what is shown in your Environment tab

## Expected Result
✅ You can see both RG-02 and RG-03 in Azure Portal  
✅ RG-02 is empty and ready for your use  
✅ RG-03 has a pre-deployed storage account  
✅ You have Contributor access on both resource groups
