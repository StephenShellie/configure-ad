<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used</h2>

- Windows Server 2022
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Preparing the AD Infrastructure in Azure
- Deploying Active Directory
- Creating Users with PowerShell
- Group Policy and Managing Accounts

---

<h2>Deployment and Configuration Steps</h2>

### Part 1: Preparing the AD Infrastructure in Azure

#### **Setup Domain Controller in Azure**
1. **Create a Resource Group**:
   - Navigate to the Azure Portal and create a new Resource Group for the lab environment.

![image](https://github.com/user-attachments/assets/983af9d6-d1da-44f8-95ed-937dde319add)

![image](https://github.com/user-attachments/assets/498067d5-52e4-4cd3-a6b8-71c27f97b4d5)

2. **Create a Virtual Network and Subnet**:
   - Set up a Virtual Network with a subnet to host your VMs.

![image](https://github.com/user-attachments/assets/f7078c3e-eef6-4365-84d5-85abffb94de3)

![image](https://github.com/user-attachments/assets/79c0c20a-8035-4cc8-9312-3d56910b33ba)

3. **Create the Domain Controller VM (Windows Server 2022)**:
   - Name the VM: `DC-1`.
   - Ensure that the VM is on the Virtual Network created previously.

![image](https://github.com/user-attachments/assets/5d48c445-14bc-4873-b569-3f91ef15de0b)

![image](https://github.com/user-attachments/assets/edcc433e-36e1-426e-a8ad-373f813e7cab)

![image](https://github.com/user-attachments/assets/abaabcdc-617c-469e-b44b-dcfbc03f7156)


4. **Set Static Private IP for DC-1**:
   - After the VM is created, navigate to its Network Interface Card (NIC) settings and set the private IP to static.
- **Navigate to the Virtual Machines window and select the DC-1 VM**
![image](https://github.com/user-attachments/assets/c730afd9-a961-491d-9655-f7e16e06fb54)

- **Navigate to the Network Setting to access the Network Interface Card**
![image](https://github.com/user-attachments/assets/e5413596-98fe-4b9e-9c6b-4ef6cdcdaf31)

- **Set the Allocation to Static underneath the Private IP Address Settings**
![image](https://github.com/user-attachments/assets/25a0214e-4709-42a1-a171-471eef6286e8)

5. **Disable Windows Firewall**:
   - Log in to `DC-1` and disable the Windows Firewall for testing connectivity.

![image](https://github.com/user-attachments/assets/ca0ea3fc-5f81-4021-be1f-4f8f6fd59d25)

![image](https://github.com/user-attachments/assets/50af1532-5ce4-4802-b0b8-69f8a58d9b7e)

#### **Setup Client-1 in Azure**
1. **Create the Client VM (Windows 10 22H2)**:
   - Name the VM: `Client-1`.

![image](https://github.com/user-attachments/assets/0073efc2-8f0a-4a51-9eb8-28b80b02ef5b)

![image](https://github.com/user-attachments/assets/8296f67d-6e67-43e8-89d5-680565c29811)

2. **Attach Client-1 to the Same Region and Virtual Network**:
   - Ensure it is in the same Virtual Network and subnet as `DC-1`.

![image](https://github.com/user-attachments/assets/8f031e3c-5ea7-49e8-bc5f-401d72464642)

3. **Set DNS Settings**:
   - Update `Client-1`'s DNS settings to point to `DC-1`'s private IP address. (navigate to the vm's network interface card)

![image](https://github.com/user-attachments/assets/2529f31c-52e8-42e7-96cb-24db754a3a17)

4. **Test Connectivity**:
   - Restart `Client-1` from the Azure Portal.
   - Log into `Client-1` and use the `ping` command to test connectivity with `DC-1`.

5. **Verify DNS Settings**:
   - Run `ipconfig /all` in PowerShell on `Client-1` to ensure the DNS points to `DC-1`.

![image](https://github.com/user-attachments/assets/14f9fa11-d976-491c-9b0b-94a611871eb1)

---

### Part 2: Deploying Active Directory

#### **Install Active Directory**
1. Log in to `DC-1`.
2. Install Active Directory Domain Services (AD DS).
3. Promote `DC-1` as a Domain Controller and set up a new forest (e.g., `mydomain.com`).
4. Restart `DC-1` and log in as `mydomain.com\labuser`.
 
 **Open Server Manager then add roles and features** 
![image](https://github.com/user-attachments/assets/e50809fe-5013-4863-bc93-accd49c2f3d7)

**Add the features from the Active Directory Domain Services**
![image](https://github.com/user-attachments/assets/1a8ae232-4388-4e5c-8328-a10e21b1ca1e)

**Open the noticiation window and select "promote this server to a domain controller"**
![image](https://github.com/user-attachments/assets/ce7ab7dc-4a19-49a6-8f71-714efd22f4c9)

**Add mydomain.com as a new forest**
![image](https://github.com/user-attachments/assets/7e03dd2f-6633-4945-b12b-fc297a6359b4)

**Deselect "Create DNS delegation**
![image](https://github.com/user-attachments/assets/b7ce680a-dfb0-4c8e-be5e-0e64b85ab119)

**Finish the setup wizard and install**
![image](https://github.com/user-attachments/assets/f326b790-90b8-4cd6-9ad3-94211488806f)

**The DC-1 will automatically restart**

**DC-1 is a domain now, in order to complete the next steps, we will have to login using the proper domain context (mydomain.com\labuser will be our username - same passwoord)**
![image](https://github.com/user-attachments/assets/47977908-e43e-45d6-9c2e-b3aa11fc22dc)

#### **Create a Domain Admin User**
1. Open Active Directory Users and Computers (ADUC).
2. Create an Organizational Unit (OU) named `_EMPLOYEES`.
3. Create another OU named `_ADMINS`.
4. Add a new user:
   - Name: `Jane Doe`
   - Username: `jane_admin`
   - Password: `Cyberlab123!`
5. Add `jane_admin` to the `Domain Admins` security group.
6. Log out and log back in as `mydomain.com\jane_admin`.

![image](https://github.com/user-attachments/assets/d359f4d6-5ddd-4b25-831e-3ef21b36182a)

![image](https://github.com/user-attachments/assets/e716ad23-8d7e-4423-adaf-ea4a5dd0de63)

![image](https://github.com/user-attachments/assets/a9965f35-3095-4026-8d91-955d7459ff52)

![image](https://github.com/user-attachments/assets/403eae7c-dc03-4194-9c3d-0a860e56a933)

![image](https://github.com/user-attachments/assets/a0a2f956-645a-4eb9-8334-507dd3227521)

![image](https://github.com/user-attachments/assets/4f782da8-8ab8-4caa-bb05-92d07a32528e)

![image](https://github.com/user-attachments/assets/f6d514d1-bd9f-4f3b-be1a-a90f094ad339)

#### **Join Client-1 to the Domain**
1. Log in as the local admin and join `Client-1` to the domain.
2. Create a new OU titled '_CLIENTS' & add `Client-1` in ADUC to `_CLIENTS`.

**Log into DC-1 as Jane the Admin**
![image](https://github.com/user-attachments/assets/7b4db476-afd5-402d-83cc-b776ac2d0d50)

**Navigate to the system window by right clicking the windows button**
![image](https://github.com/user-attachments/assets/75f2c0f9-a133-4f3c-be28-ce8e0f4dc3bc)

**Join Client-1 to the domain by using the 'rename this pc' tool**
![image](https://github.com/user-attachments/assets/bf3118ec-0f69-477f-8cfd-1d610fdac29d)

**Verify that Client-1 has joined the domain**
![image](https://github.com/user-attachments/assets/901e07d2-e124-4af7-8c47-2dd537a2176d)

**Create a new folder named '_CLIENTS' and drag/drop the Client-1 computer into it**
![image](https://github.com/user-attachments/assets/92260c70-0a04-4a6a-be54-c2c3fa3719d4)
---

### Part 3: Creating Users with PowerShell

#### **Setup Remote Desktop for Domain Users**
1. Log into `Client-1` as `mydomain\jane_admin`.
2. Open System Properties and enable Remote Desktop.
3. Allow "domain users" access to Remote Desktop.

![image](https://github.com/user-attachments/assets/a4f56223-7b7b-4528-aa98-3d89c01f426c)

![image](https://github.com/user-attachments/assets/74d7cafa-10be-4032-acbb-e4133fa03843)

#### **Create Users with PowerShell**
1. Log in to `DC-1` as `jane_admin`.
2. Open PowerShell ISE as an administrator.
3. Create multiple new users using a script (script link: https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1).
4. Verify users appear in the `_EMPLOYEES` OU in ADUC.
5. Attempt to log into `Client-1` with one of the created accounts.

![image](https://github.com/user-attachments/assets/0e7179ae-7591-4ccc-8230-56810a748306)

**Create a new file**
![image](https://github.com/user-attachments/assets/b7836c22-49a0-4bb7-9437-330b497c57d4)

![image](https://github.com/user-attachments/assets/40458e9b-fdf1-4d0b-b402-053268ffaaac)

![image](https://github.com/user-attachments/assets/a85410d6-128e-4594-bb8a-76eb739f5911)

![image](https://github.com/user-attachments/assets/e46183bc-4bbe-4f5b-ba4a-6b8c667caa1d)

![image](https://github.com/user-attachments/assets/89715764-a310-410a-83d3-203b356a18dc)

---

### Part 4: Group Policy and Managing Accounts

#### **Account Lockout Configuration**
1. Log in to `DC-1`.
2. Open Group Policy Management.
3. Edit the Default Domain Policy:
   - Set account lockout threshold to 5 invalid attempts.
4. Attempt to log in with a user account using incorrect passwords. Observe the account lockout behavior.
5. Unlock the account in ADUC and reset the password.

<p>
<img src="https://i.imgur.com/4xZVZxJ.png" height="80%" width="80%" alt="Lab 5"/>
</p>

<p>
<img src="https://i.imgur.com/KW95ZOG.png" height="80%" width="80%" alt="Lab 5"/>
</p>

<p>
<img src="https://i.imgur.com/uWoFU61.png" height="80%" width="80%" alt="Lab 5"/>
</p>

#### **Enable and Disable Accounts**
1. Disable a user account in ADUC.
2. Attempt to log in with the disabled account and observe the error message.
3. Re-enable the account and log in successfully.

<p>
<img src="https://i.imgur.com/CUUcX8S.png" height="80%" width="80%" alt="Lab 5"/>
</p>

<p>
<img src="https://i.imgur.com/TkMtlW1.png" height="80%" width="80%" alt="Lab 5"/>
</p>

<p>
<img src="https://i.imgur.com/KWKL3VZ.png" height="80%" width="80%" alt="Lab 5"/>
</p>

#### **Observing Logs**
1. Review authentication and account-related logs in Event Viewer:
   - Log on `DC-1` for domain-level events (shown below).
   - Log on `Client-1` for local events.

<p>
<img src="https://i.imgur.com/YwM0mgQ.png" height="80%" width="80%" alt="Lab 5"/>
</p>

---

### Completion

Congratulations! You have successfully deployed and configured an on-premises Active Directory environment in Azure.
