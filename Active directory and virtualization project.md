
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

Later, Windows clients will ask DC01:

"Who is Hector?"

and DC01 will be able to answer:

"Hector is a member of the IT department and belongs to these security groups."

That's the fundamental idea behind Active Directory.

