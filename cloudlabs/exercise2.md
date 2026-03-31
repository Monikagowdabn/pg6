# Exercise 2: Explore the Pre-Deployed Storage Account

## Goal
Explore the storage account that was automatically deployed in RG-03 via ARM template.

## Estimated Time
15 minutes

## Steps

### Task 1: Open the Storage Account
1. Go to Azure Portal → **Resource Groups** → click on **RG-03**
2. Click on the **Storage Account** resource
3. You are now on the Storage Account Overview page

### Task 2: Explore Storage Account Properties
1. On the **Overview** page, note the following:
   - **Location**: East US
   - **Performance**: Standard
   - **Replication**: LRS (Locally Redundant Storage)
   - **Account Kind**: StorageV2

2. Click **Configuration** in the left menu and verify:
   - Minimum TLS version: **TLS 1.2**
   - Secure transfer required: **Enabled**
   - Allow Blob public access: **Disabled**

### Task 3: Explore Blob Storage
1. Click **Containers** in the left menu
2. Notice there are no containers yet
3. Click **+ Container** to create one:
   - Name: `training-container`
   - Public access level: **Private**
4. Click **Create**
5. Click on the new container → click **Upload** → upload any small file

### Task 4: View Access Keys
1. Click **Access Keys** in the left menu
2. Click **Show** to reveal the keys
3. Note the **Connection String** — this is used by applications to connect to storage

## Expected Result
✅ Storage account properties are visible  
✅ TLS 1.2 and HTTPS enforced  
✅ Successfully created a blob container  
✅ Uploaded a file to the container
