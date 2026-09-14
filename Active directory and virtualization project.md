
**Hardware Specs:**
**CPU:** Ryzen 7 5800x.
**RAM:** 32 GB.
**Free disk space:** 230 GB.
**Virtualization:** Enabled.
**Virtualization software**: VirtualBox

Phase 1:  Windows Server Virtual installation:

<img width="1025" height="849" alt="Pasted image 20260914130826" src="https://github.com/user-attachments/assets/63422ad9-8ec6-4542-8a8b-e823a87a78db" />

We'll proceed with the Windows Server Evaluation version intallation. We already configure the VM network to Host-Only. And, for now, we will install the Desktop Experience version. 

<img width="1024" height="854" alt="Pasted image 20260914131026" src="https://github.com/user-attachments/assets/e375860f-08c9-42a0-ad69-1742bfb97772" />

Once we have correctly installed Windows Server, we are going to Rename the VM to DC01 and configure the IP adress:

<img width="1029" height="860" alt="image" src="https://github.com/user-attachments/assets/71374b8b-3a6b-4279-bfff-2d317acdd223" />

<img width="463" height="557" alt="image" src="https://github.com/user-attachments/assets/0be7e3c7-efda-4612-b807-7e45a7263478" />

Why no gateway?

Because this is an isolated laboratory network.

Our fictional company doesn't need Internet access yet.

Later, we'll create a more sophisticated topology where the lab can access the Internet through a router/firewall VM.

For now we have a 192.168.10.0/24 corporate network

<img width="613" height="694" alt="image" src="https://github.com/user-attachments/assets/aad484f5-d1d0-437c-8daf-cc22a87d78e2" />

We have correctly installed and configured a VM with Windows Server working. Our checkpoint is this:

Rhinwood Technologies
        │
        │
   Corporate LAN
192.168.10.0/24
        │
        ▼
      DC01
 192.168.10.10
        │
        ├── Future Active Directory
        ├── Future DNS
        └── Future DHCP

**Phase 2
Converting DC01 into a Domain Controller**

Right now we have:

DC01
192.168.10.10
Windows Server 2025

We're going to turn it into:

             Rhinwood Technologies
                      │
               rhinwood.local
                      │
                    DC01
                      │
          ┌───────────┴───────────┐
          │                       │
     Active Directory            DNS


once we installed the Active Directory feature though the Installationn Manager, we'll promote the VM to Controller Domain 
<img width="1031" height="857" alt="image" src="https://github.com/user-attachments/assets/1811d512-0a4d-45ef-b3cc-8fac4dbc70a1" />

<img width="767" height="564" alt="image" src="https://github.com/user-attachments/assets/34e950db-b28b-4e06-9da9-75490b1a4ea3" />
We will create a password

<img width="758" height="559" alt="image" src="https://github.com/user-attachments/assets/109ba01d-2273-464f-ae21-e818349f2fb5" />

We are done with the AC administrator assigment
<img width="1028" height="834" alt="image" src="https://github.com/user-attachments/assets/44774ff2-8271-490b-8c80-ffef5be41102" />

just wait to the VM to restart 

<img width="1027" height="820" alt="image" src="https://github.com/user-attachments/assets/4a26d2c3-7b14-462a-b547-d507ec9232a9" />
Now we have correctly installed the VM and the Active Directory with the Admin role for this user.

**Phase 3 — Build the Active Directory structure**

Now we start doing something that actually resembles a real company environment.

We'll create:

rhinwood.local
AD-Users:
Management
IT
Sales
HR

AD-Computers:
Workstations
Servers

AD-Groups:
GRP-IT-Admins
GRP-Sales
GRP-HR
GRP-ADM-Management

OUs (Organizational Units) let administrators:

Apply different Group Policies to different departments.
Organize users and computers.
Delegate administrative permissions.
Manage hundreds/thousands of objects systematically.
Separate workstations from servers.

We create everything and it looks like this 

<img width="949" height="785" alt="image" src="https://github.com/user-attachments/assets/bb078ac4-bf42-41c6-8f47-b97e440d8324" />


