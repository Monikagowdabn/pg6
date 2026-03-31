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
     <img width="1900" height="814" alt="Screenshot 2026-03-31 122118" src="https://github.com/user-attachments/assets/5dc72579-1c86-44ae-8f3f-420b95cf6f51" />


### Task 2: Explore RG-02 (Sandbox)
1. Click on your **RG-02** resource group
2. Notice it is currently empty — this is your sandbox to create resources
3. Note the **Location** (East US) shown in the Overview
4. Click **Access Control (IAM)** → Click **Role Assignments**
5. Verify you have **Contributor** role assigned to your user
   <img width="1905" height="916" alt="Screenshot 2026-03-31 122633" src="https://github.com/user-attachments/assets/d0e14de4-db8a-4dc5-878b-99222dd6dcb7" />


### Task 3: Explore RG-03 (Pre-deployed Storage)
1. Go back to Resource Groups
2. Click on your **RG-03** resource group
3. You should see a **Storage Account** already deployed here
4. Click on it to explore its properties
5. Note the **Storage Account Name** — it matches what is shown in your Environment tab
   <img width="1909" height="771" alt="Screenshot 2026-03-31 122147" src="https://github.com/user-attachments/assets/3841b022-54bf-462f-99b9-dccea49ad529" />


## Expected Result
✅ You can see both RG-02 and RG-03 in Azure Portal  
✅ RG-02 is empty and ready for your use  
✅ RG-03 has a pre-deployed storage account  
✅ You have Contributor access on both resource groups
