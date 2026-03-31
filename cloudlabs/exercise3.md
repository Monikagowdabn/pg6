# Exercise 3: Create Resources in Your Sandbox (RG-02)

## Goal
Create your own Azure Storage Account in the sandbox resource group RG-02.

## Estimated Time
20 minutes

## Steps

### Task 1: Create a Storage Account in RG-02
1. Go to Azure Portal → **Resource Groups** → click on **RG-02**
2. Click **+ Create**
3. Search for **Storage Account** → click **Create**
4. Fill in the following details:

   | Field | Value |
   |---|---|
   | Resource Group | Select your RG-02 |
   | Storage Account Name | Enter a unique name (3-24 lowercase letters/numbers only) |
   | Region | East US |
   | Performance | Standard |
   | Redundancy | LRS |

5. Click **Review + Create** → **Create**
6. Wait for deployment to complete (~1-2 minutes)

### Task 2: Explore Your New Storage Account
1. Click **Go to resource** after deployment completes
2. Explore the Overview page
3. Compare it with the pre-deployed storage account in RG-03:
   - Are they in the same region?
   - Do they have the same replication type?

### Task 3: Create a Blob Container and Upload a File
1. Click **Containers** → **+ Container**
2. Name it: `my-sandbox-container`
3. Click **Create**
4. Click on the container → **Upload**
5. Upload any file from your computer
6. Verify the file appears in the container

## Expected Result
✅ New storage account created in RG-02  
✅ Storage account is in East US region  
✅ Blob container created successfully  
✅ File uploaded to the container
