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

<img width="1909" height="771" alt="Screenshot 2026-03-31 122147" src="https://github.com/user-attachments/assets/a94080ba-5bc7-46ee-8548-a884ccb14c07" />


### Task 2: Explore Storage Account Properties
1. On the **Overview** page, note the following:
   - **Location**: East US
   - **Performance**: Standard
   - **Replication**: LRS (Locally Redundant Storage)
   - **Account Kind**: StorageV2
   - 
<img width="1907" height="912" alt="Screenshot 2026-03-31 123252" src="https://github.com/user-attachments/assets/70c49df3-a200-49ad-9b44-a20a6fa10ca2" />

2. Click **Configuration** in the left menu and verify:
   - Minimum TLS version: **TLS 1.2**
   - Secure transfer required: **Enabled**
   - Allow Blob public access: **Disabled**
     <img width="1901" height="877" alt="Screenshot 2026-03-31 123609" src="https://github.com/user-attachments/assets/dab444dc-9f62-4acd-89ba-8e549b6356e7" />


### Task 3: Explore Blob Storage
1. Click **Containers** in the left menu
2. Notice there are no containers yet
3. Click **+ Container** to create one:
   - Name: `training-container`
   - Public access level: **Private**
4. Click **Create**
5. Click on the new container → click **Upload** → upload any small file
   <img width="1905" height="900" alt="Screenshot 2026-03-31 123954" src="https://github.com/user-attachments/assets/6bd6d024-6f73-4083-82e1-acb60408525c" />

<img width="1902" height="875" alt="Screenshot 2026-03-31 124130" src="https://github.com/user-attachments/assets/7cfe460c-8614-4b0a-8298-838579c5cd49" />
<img width="1909" height="852" alt="Screenshot 2026-03-31 124158" src="https://github.com/user-attachments/assets/6feae970-850f-44fe-b971-47d56e1d274e" />


### Task 4: View Access Keys
1. Click **Access Keys** in the left menu
2. Click **Show** to reveal the keys
3. Note the **Connection String** — this is used by applications to connect to storage
   
   <img width="1901" height="902" alt="Screenshot 2026-03-31 124656" src="https://github.com/user-attachments/assets/63b48bf1-9ac0-4715-868d-280beba805bf" />


## Expected Result
✅ Storage account properties are visible  
✅ TLS 1.2 and HTTPS enforced  
✅ Successfully created a blob container  
✅ Uploaded a file to the container
