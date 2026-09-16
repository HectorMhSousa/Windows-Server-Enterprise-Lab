# Windows Server Enterprise Lab

### A practical Active Directory & Windows Server homelab

> A simulated enterprise IT infrastructure built from scratch using Windows Server, Active Directory, DNS, DHCP, Group Policy, Windows 11 clients, and NTFS/share permissions.

---

## 📌 Project Overview

**Rhinwood Technologies** is a fictional company created for this lab.

The objective is to simulate the deployment and administration of a small corporate Windows environment, including centralized identity management, organizational structure, file access control, workstation management, and security policies.

The environment was built entirely in **VirtualBox** using an isolated virtual network.

---

## 🖥️ Current Environment

| Component | Configuration |
|---|---|
| Hypervisor | VirtualBox |
| Domain Controller | DC01 |
| Server OS | Windows Server 2025 |
| Client OS | Windows 11 |
| Domain | `rhinwood.local` |
| Network | `192.168.10.0/24` |
| DC01 IP | `192.168.10.10` |
| DNS | DC01 |
| DHCP | DC01 |
| Client | CLIENT01 |
| Network Type | Isolated Host-Only |

---

## 🏗️ Infrastructure Architecture

```text
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

```

The infrastructure is intentionally isolated from the physical network so that configuration and testing can be performed safely.

---

## 🔐 Active Directory

The first major objective was to create the company's Active Directory environment.

### Domain

`rhinwood.local`

### Organizational Units

```text
AD-Users
├── Management
├── IT
├── Sales
└── HR

AD-Computers
├── Workstations
└── Servers

```

### AD Groups

* `GRP-IT-Admins`
* `GRP-Sales`
* `GRP-HR`
* `GRP-ADM-Management`

The default AD containers such as Users, Computers, and Domain Controllers were preserved, while custom OUs were created to provide a cleaner administrative structure.

---

<img width="1029" height="790" alt="image" src="https://github.com/user-attachments/assets/780c8414-0ec0-4165-85ff-ef2440a04b46" />


## 👤 User & Group Management

Several test users were created to represent different departments:

| User | Department | Security Group |
| --- | --- | --- |
| Carlos Rodriguez | Sales | `GRP-Sales` |
| Maria Santos | IT | `GRP-IT-Admins` |
| Laura Gomez | HR | `GRP-HR` |
| David Martinez | Management | `GRP-ADM-Management` |

This structure allows permissions to be assigned through security groups instead of individual user accounts. This follows a common Active Directory administration principle: assigning access through groups makes permission management easier to maintain and scale.

---

<img width="945" height="846" alt="image" src="https://github.com/user-attachments/assets/65073136-cd82-4aa1-ab21-43cb4cadf074" />

## 📁 File Server & Permissions

A departmental file structure was created on DC01:

```text
C:\CompanyData
│
├── IT
├── Sales
├── HR
└── Management

```

The folder was shared over the network as: `\\DC01\CompanyData`

### Share Permissions

The share was configured with:

* `Everyone` → Full Control

The actual security boundary was implemented through NTFS permissions.

### NTFS Permissions

Each departmental folder grants:

* `Administrators` → Full Control
* `SYSTEM` → Full Control
* `Department Group` → Modify

Inheritance was disabled on the departmental folders so that permissions could be controlled independently. For example:

* **IT:** `GRP-IT-Admins` → Modify
* **Sales:** `GRP-Sales` → Modify
* **HR:** `GRP-HR` → Modify
* **Management:** `GRP-ADM-Management` → Modify

---

## 🧪 Permission Testing

The permissions were tested using actual domain accounts from CLIENT01.

### Maria — IT

Maria was able to create a file in `\\DC01\CompanyData\IT`, but received `Access is denied` when attempting to write to the Sales folder.

### Carlos — Sales

Carlos was able to create `carlos-test.txt` inside `\\DC01\CompanyData\Sales`, but received `Access is denied` when attempting to write to IT.

These tests demonstrated that the department-based access control was functioning correctly.

---

<img width="1026" height="855" alt="image" src="https://github.com/user-attachments/assets/d7369c76-8a8c-4e0e-9dc3-423578c7bf57" />


## 💻 Windows 10 Domain Client

A Windows 10 virtual machine was deployed as **CLIENT01**.

The machine was joined to `rhinwood.local`. After joining the domain, CLIENT01 was moved into:

```text
AD-Computers
└── Workstations

```

Domain users were then able to authenticate against Active Directory (e.g., `RHINWOOD\carlos.rodriguez`, `RHINWOOD\maria.santos`).

---

<img width="1023" height="853" alt="image" src="https://github.com/user-attachments/assets/3c3e2db8-e770-432c-8569-443b7d8ccb14" />


## ⚙️ Group Policy

A domain-level Group Policy Object was created: **RHINWOOD - Workstation Security**

The first security policy implemented was:

* **Interactive Logon Inactivity Limit:** 15 minutes

Configured through:

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Local Policies
                └── Security Options
                    └── Interactive logon: Machine inactivity limit

```

The policy was successfully applied to CLIENT01 using:

```bash
gpupdate /force

```

Verification was performed using:

```bash
gpresult /scope computer /r

```

The resulting Windows configuration was also verified through the registry:

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v InactivityTimeoutSecs

```

Expected value: `InactivityTimeoutSecs REG_DWORD 0x384` (`0x384` corresponds to 900 seconds / 15 minutes).

---

<img width="870" height="956" alt="image" src="https://github.com/user-attachments/assets/8fb2d456-1bf6-45f2-a70b-60f4d5cd4e6e" />


## 🛠️ Troubleshooting Experience

One of the most valuable parts of the project was troubleshooting a real Active Directory problem. Initially, `gpupdate /force` failed on CLIENT01 because the computer's clock was not synchronized correctly with the domain controller.

### Investigation

Windows Time was examined using:

```cmd
w32tm /query /status
w32tm /query /source
w32tm /stripchart /computer:DC01 /samples:5 /dataonly

```

The stripchart initially showed an offset of approximately `+32388` seconds (roughly 9 hours). Further investigation revealed incorrect timezone and system-time configuration in the lab environment.

After correcting the configuration and synchronizing CLIENT01 with the domain hierarchy, the offset dropped to approximately `+34` seconds.

The Group Policy update subsequently completed successfully:

```text
Computer Policy update has completed successfully.
User Policy update has completed successfully.

```

### Troubleshooting Tools Used

* `w32tm`, `tzutil`, `gpupdate`, `gpresult`
* Registry verification
* Active Directory tools

### Key Takeaways

* Windows Time Service troubleshooting & domain time hierarchy
* Active Directory authentication time requirements
* Group Policy troubleshooting, diagnosing configuration problems, and verifying remediation

---

## 🧰 Technologies & Skills

* **Windows Server:** Windows Server 2025, Active Directory Domain Services, DNS, DHCP, Windows Time Service, Group Policy
* **Active Directory:** Domains, Organizational Units, Users, Security Groups, Computer Accounts, Domain Joining, Group Membership, Centralized Authentication
* **File Services:** SMB shares, NTFS permissions, Share permissions, Permission inheritance, Department-based access control
* **Windows Administration:** Windows 11 administration, Domain authentication, Command-line utilities (`gpupdate`, `gpresult`, `w32tm`, `tzutil`), Registry verification, Troubleshooting
* **Virtualization:** VirtualBox, Virtual Machines, Host-only networking, Isolated lab environments

---

## 📊 Project Status

**Status:** 🟢 Active Development

### Completed

* [x] VirtualBox infrastructure & isolated networking
* [x] Windows Server 2025 & static networking
* [x] Active Directory Domain Services, DNS, DHCP
* [x] Organizational Units, Users, and security groups
* [x] SMB file sharing & NTFS permissions
* [x] Windows 10 domain client integration
* [x] Group Policy implementation
* [x] Domain time synchronization & Group Policy troubleshooting

### Planned

* [ ] Password policy & Account lockout policy
* [ ] Department-specific GPOs
* [ ] Automatic network drive mapping
* [ ] Additional Windows client & Linux server integration
* [ ] Monitoring & Backup systems
* [ ] Additional security controls

---

## 🎯 Learning Objectives

This project is designed to develop practical skills in:

* Deploying Windows Server infrastructure
* Managing Active Directory environments & centralized authentication
* Designing organizational structures, managing users, and security groups
* Implementing NTFS and SMB permissions
* Joining and managing Windows clients
* Deploying and troubleshooting Group Policy
* Diagnosing Windows Time synchronization issues
* Building and documenting a realistic IT infrastructure

---

## 📚 Project Philosophy

This lab is designed as a hands-on simulation rather than a theoretical exercise. Whenever possible, configurations are tested using real domain accounts and client machines.

The goal is not simply to make the infrastructure work, but to understand **why** each component is required, **how** they interact, **how** to troubleshoot failures, and **how** to verify configurations. The environment will continue to evolve as new technologies and administration scenarios are introduced.

---

## 👨‍💻 Author

**Héctor Manuel Hernández**

*ASIR Student | Systems Administration | Networking | Windows & Linux*

This project is part of a personal technical portfolio focused on developing practical systems administration and infrastructure skills.
