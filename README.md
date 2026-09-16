A practical Active Directory & Windows Server homelab.
A simulated enterprise IT infrastructure built from scratch using Windows Server, Active Directory, DNS, DHCP, Group Policy, Windows 11 clients, and NTFS/share permissions.

1. Project Overview
Rhinwood Technologies is a fictional company created for this lab.

The objective is to simulate the deployment and administration of a small corporate Windows environment, including centralized identity management, organizational structure, file access control, workstation management, and security policies.

The environment was built entirely in VirtualBox using an isolated virtual network.

Current environment.

Component	            Configuration
Hypervisor	          VirtualBox
Domain Controller	    DC01
Server OS	            Windows Server 2025
Client OS	            Windows 10
Domain	              rhinwood.local
Network              	192.168.10.0/24
DC01 IP	              192.168.10.10
DNS	                  DC01
DHCP	                DC01
Client	              CLIENT01
Network type	        Isolated Host-Only

2. Infrastructure Architecture

                         RHINWOOD.LOCAL
                              │
                              │
                    ┌─────────▼─────────┐
                    │       DC01        │
                    │ Windows Server    │
                    │      2025         │
                    │                   │
                    │ AD DS             │
                    │ DNS               │
                    │ DHCP              │
                    │ Group Policy      │
                    └─────────┬─────────┘
                              │
                       192.168.10.0/24
                              │
                    ┌─────────▼─────────┐
                    │     CLIENT01      │
                    │    Windows 11     │
                    │ Domain Joined     │
                    └───────────────────┘
The infrastructure is intentionally isolated from the physical network so that configuration and testing can be performed safely.


3. Active Directory

The first major objective was to create the company's Active Directory environment.

Domain: rhinwood.local

Organizational Units:

  AD-Users
  ├── Management
  ├── IT
  ├── Sales
  └── HR
  
  AD-Computers
  ├── Workstations
  └── Servers
  
  AD-Groups
  ├── GRP-IT-Admins
  ├── GRP-Sales
  ├── GRP-HR
  └── GRP-ADM-Management

The default AD containers such as Users, Computers, and Domain Controllers were preserved, while custom OUs were created to provide a cleaner administrative structure.




















