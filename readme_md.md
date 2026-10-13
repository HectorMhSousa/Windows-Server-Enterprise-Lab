# Systems Administration Lab: Active Directory, SMB, and Zabbix

![Windows Server](https://img.shields.io/badge/Windows_Server-Active_Directory-0078D6?style=flat&logo=windows)
![Ubuntu Server](https://img.shields.io/badge/Ubuntu_Server-Linux-E95420?style=flat&logo=ubuntu)
![Monitoring](https://img.shields.io/badge/Monitoring-Zabbix-D22730?style=flat&logo=zabbix)
![Networking](https://img.shields.io/badge/Networking-TCP%2FIP-00599C?style=flat)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat)

---

## 1. Project Overview

This project involved building a virtual systems administration lab to practice technologies commonly used in IT infrastructure.

The lab began with the configuration of Windows Server and Active Directory Domain Services (AD DS) to centralize the management of users, computers, and authentication. We then worked with SMB file shares and troubleshot connectivity issues.

Finally, we deployed Zabbix Server on Ubuntu Server, installed Zabbix Agent 2, and configured communication between the monitoring server and the monitored hosts.

The goal was to build a functional infrastructure and learn how to diagnose real-world systems administration issues.

### Objectives
* Install and configure Active Directory.
* Configure DNS for domain name resolution.
* Centrally manage users and computers.
* Practice configuring SMB file shares.
* Troubleshoot network connectivity and resource access issues.
* Install Zabbix Server on Ubuntu Server.
* Configure Zabbix Agent 2.
* Monitor hosts through the Zabbix web interface.
* Troubleshoot issues involving IP addresses, ports, services, and agent configuration.

---

## 2. Lab Architecture

The environment was built using virtual machines connected to a private lab network. The network used during the monitoring tests belonged to the `192.168.10.0/24` subnet.

### Components

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **Domain Controller** | Windows Server | Authentication and centralized administration |
| **Directory Service** | Active Directory Domain Services | Domain users, computers, and groups |
| **Name Resolution** | DNS | Name resolution and discovery of Active Directory services |
| **Windows Client** | Windows | Authentication and resource access testing |
| **File Sharing** | SMB | Access to shared network folders |
| **Monitoring Server** | Ubuntu Server | Hosts Zabbix Server |
| **Monitoring Agent** | Zabbix Agent 2 | Collects system metrics |
| **Management Interface** | Zabbix Web | Monitoring visualization and management |

### Logical Diagram

```text
                  PRIVATE LAB NETWORK (192.168.10.0/24)
  +-----------------------+-----------------------+-----------------------+
  |                       |                       |                       |
  v                       v                       v                       v
+-------------------+   +-------------------+   +-------------------+
|  Windows Server   |   |  Windows Client   |   |   Ubuntu Server   |
|-------------------|   |-------------------|   |-------------------|
| Active Directory  |   | Domain-joined     |   | Zabbix Server     |
| DNS               |   | workstation       |   | Web Interface     |
| SMB               |   |                   |   | Agent 2           |
+-------------------+   +-------------------+   +-------------------+
```

> **Note:** The diagram represents the logical components. The exact placement of Zabbix Agent 2 depends on the machines where it was installed during the lab exercises.

### Installation Parameters

*Fill in this table with the actual machine settings before publishing the repository.*

| Parameter | Value |
| :--- | :--- |
| **Hypervisor** | `[VirtualBox / VMware / other]` |
| **Active Directory domain** | `[domain.local]` |
| **Domain Controller** | `[DC01]` |
| **Domain Controller IP** | `[Domain Controller IP address]` |
| **Zabbix Server** | `[ZABBIX01]` |
| **Zabbix Server IP** | `[Zabbix Server IP address]` |
| **Lab network** | `192.168.10.0/24` |
| **Monitoring server OS** | Ubuntu Server |
| **Zabbix version** | 8.0 branch |

---

## 3. Virtual Machine Preparation

Before configuring the services, we prepared the virtual machines and checked their network connectivity.

**Initial tasks:**
1. Preparing the virtual machines.
2. Installing the operating systems.
3. Configuring the network adapters.
4. Checking IP addressing.
5. Verifying communication between hosts.

### 3.1. Checking Network Configuration on Windows

Display network configuration:
```cmd
ipconfig /all
```

Test connectivity to another host:
```cmd
ping <DESTINATION_IP>
```

### 3.2. Checking Network Configuration on Ubuntu

Display network interfaces:
```bash
ip address
```

Display routing table:
```bash
ip route
```

Test connectivity:
```bash
ping -c 4 <DESTINATION_IP>
```

---

## 4. Active Directory Installation and Configuration

### 4.1. What Is Active Directory?
Active Directory Domain Services is Microsoft's directory service for centrally managing identities, computers, and organizational resources. Instead of managing each computer independently, an administrator can manage users and computers from a domain controller.

**Main functions:**
* Centralized authentication.
* User and group management.
* Computer organization.
* Permission management.
* DNS integration.
* Group Policy management.

### 4.2. Installing the AD DS Role
On Windows Server, we installed the Active Directory Domain Services role.

**General procedure:**
1. Open **Server Manager**.
2. Select **Add Roles and Features**.
3. Choose a **role-based or feature-based installation**.
4. Select the destination server.
5. Check **Active Directory Domain Services**.
6. Install the required features and complete the wizard.

*Note: After installing the role, the server must still be promoted to a domain controller.*

### 4.3. Promoting the Server to a Domain Controller
From Server Manager, launch the server promotion wizard.

**General procedure:**
1. Select the option to **add a new forest**.
2. Enter the domain's DNS name.
3. Configure domain controller options and set the Directory Services Restore Mode (DSRM) password.
4. Review DNS options and run prerequisite checks.
5. Promote and restart the server.

### 4.4. Verifying the Services
After restarting, verify that the key services are running in PowerShell:

```powershell
Get-Service NTDS
Get-Service DNS
```

Check the overall health of the domain controller:
```cmd
dcdiag
```

---

## 5. DNS Configuration and Domain Joining

### 5.1. Why DNS Matters
DNS is essential to Active Directory. Domain clients use DNS to resolve names and locate domain services, including domain controllers. A client may have IP connectivity to the server and still be unable to join the domain if it is using an incorrect DNS server.

### 5.2. DNS Configuration
On domain clients, configure the primary DNS server to point to the Active Directory Domain Controller IP.

Check client network settings:
```cmd
ipconfig /all
```

Test domain name resolution:
```cmd
nslookup <AD_DOMAIN>
```

Query domain controller SRV records:
```cmd
nslookup -type=SRV _ldap._tcp.dc._msdcs.<AD_DOMAIN>
```

### 5.3. Joining a Windows Client to the Domain
1. Open **System Properties** > **Computer Name**.
2. Select **Change...** and choose **Domain**.
3. Enter the domain name and provide authorized credentials when prompted.
4. Restart the computer and sign in using a domain account.

---

## 6. User and Computer Management

Management is primarily performed using the **Active Directory Users and Computers** console.

### 6.1. Users and Groups
Security groups should be used to assign permissions rather than assigning them directly to individual user accounts.

### 6.2. Organizational Units (OUs)
Example hierarchy:

```text
DOMAIN
├── Users
│   ├── Administration
│   ├── Systems
│   └── Support
├── Computers
│   ├── Clients
│   └── Servers
└── Groups
```

### 6.3. Validation
* Users exist in the directory.
* Computers appear in the domain.
* Authorized accounts can log in.
* Group memberships are assigned correctly.

---

## 7. SMB File Share Configuration

### 7.1. What Is SMB?
SMB (Server Message Block) is used to share files, directories, and printers across a network. Shared resources are accessed using UNC paths: `\\SERVER\SHARE`.

### 7.2. Creating a Shared Resource
1. Create a folder on the server.
2. Open **Properties** > **Sharing** > **Advanced Sharing**.
3. Check **Share this folder**, specify a share name, and configure access permissions.

### 7.3. SMB and NTFS Permissions
Effective access is determined by the most restrictive combination of **Share** and **NTFS** permissions.

* **Best practices:** Use Active Directory security groups, follow the principle of least privilege, test access with standard user accounts, and avoid disabling the host firewall as a permanent solution.

### 7.4. Checking SMB Connectivity
SMB primarily operates over TCP port 445.

Test connectivity using PowerShell:
```powershell
Test-NetConnection <SERVER> -Port 445
```

List active SMB shares on the server:
```powershell
Get-SmbShare
```

---

## 8. Troubleshooting SMB System Error 67

System error 67 typically indicates that the network name cannot be found.

### 8.1. Check Checklist
1. **UNC Path:** Verify the `\\SERVER\SHARE` syntax.
2. **DNS Resolution:** Run `nslookup SERVER` and `ping <SERVER_IP>`.
3. **TCP Port 445:** Run `Test-NetConnection <SERVER> -Port 445`.
4. **Share Publication:** Verify the share exists on the server via `Get-SmbShare`.

### 8.2. Lessons Learned
Troubleshooting should always follow a structured, layered approach:
$$\text{UNC Path} \longrightarrow \text{DNS} \longrightarrow \text{IP Ping} \longrightarrow \text{TCP Port 445} \longrightarrow \text{SMB Service} \longrightarrow \text{Share/NTFS Permissions}$$

---

## 9. Installing Zabbix Server on Ubuntu

### 9.1. Monitoring Objectives
Zabbix tracks host availability, resources, and service metrics via active/passive agents and network checks.

### 9.2. Preparing Ubuntu
Update local package indexes:
```bash
sudo apt update && sudo apt upgrade -y
lsb_release -a
```

### 9.3. Official Zabbix Repository
This lab utilizes the **Zabbix 8.0** package branch for Ubuntu.
* **Repository:** `https://repo.zabbix.com/zabbix/8.0/ubuntu/`
* Refer to official documentation for installation instructions: [Zabbix Downloads](https://www.zabbix.com/download)

### 9.4. Zabbix Core Architecture

| Component | Purpose |
| :--- | :--- |
| **Zabbix Server** | Processes monitoring configurations and data |
| **Database** | Stores configurations and historical telemetry |
| **Web Server** | Hosts the web-based management interface |
| **Zabbix Agent 2** | Collects metrics from target endpoints |

### 9.5. Verifying Services
```bash
systemctl status zabbix-server
systemctl status apache2
systemctl list-units --type=service | grep -i zabbix
```

---

## 10. Installing and Configuring Zabbix Agent 2

### 10.1. Package Availability Check
```bash
apt-cache policy zabbix-agent2
```

### 10.2. Installation and Service Management
```bash
sudo apt install zabbix-agent2 -y
sudo systemctl enable --now zabbix-agent2
sudo systemctl status zabbix-agent2
```

### 10.3. Agent Configuration
Configuration file: `/etc/zabbix/zabbix_agent2.conf`

Key parameters:
```ini
Server=<ZABBIX_SERVER_IP>
ServerActive=<ZABBIX_SERVER_IP>
Hostname=<HOST_NAME>
```

| Parameter | Purpose |
| :--- | :--- |
| `Server` | IP addresses allowed to perform passive checks |
| `ServerActive` | Target server/proxy for active checks |
| `Hostname` | Unique hostname matching the host configured in Zabbix |

> **Crucial Warning on `127.0.0.1`:** Using `127.0.0.1` in `Server` or `ServerActive` instructs the agent to look for Zabbix Server on its local loopback interface. If Zabbix Server is hosted on a separate VM, you **must** specify the external IP address reachable across the lab subnet.

Restart service to apply changes:
```bash
sudo systemctl restart zabbix-agent2
```

---

## 11. Zabbix Connectivity and Ports

### 11.1. Port Reference

| TCP Port | Protocol / Service | Direction / Role |
| :---: | :---: | :--- |
| **445** | SMB | Resource Sharing |
| **10050** | Zabbix Agent | Passive checks (Server $\rightarrow$ Agent) |
| **10051** | Zabbix Server/Proxy | Active checks (Agent $\rightarrow$ Server) |

### 11.2. Verification Commands
Check active socket bindings for port 10050:
```bash
sudo ss -lntp | grep 10050
```

---

## 12. Adding Hosts to Zabbix

1. Open Zabbix Web UI $\rightarrow$ **Configuration** $\rightarrow$ **Hosts** $\rightarrow$ **Create host**.
2. Set **Host name** (must exactly match the `Hostname` string configured in `zabbix_agent2.conf`).
3. Assign host to a **Host Group**.
4. Add an **Agent Interface** with the target machine's IP address.
5. Link appropriate **Templates** (e.g., Linux by Zabbix agent active/passive).
6. Save and verify status.

---

## 13. Troubleshooting the Gray ZBX Indicator

### 13.1. Problem & Common Causes
A gray `ZBX` availability icon indicates Zabbix Server has not yet established communication with the host agent.

**Diagnosis Steps:**
1. Verify `zabbix-agent2` process state.
2. Confirm matching host IP and agent listening interface.
3. Validate `Server`, `ServerActive`, and `Hostname` values.
4. Ensure firewall rules allow TCP traffic on port `10050`.

### 13.2. Connection Reset Error
If the following error occurs:
```text
Get value error: cannot read from socket: [104] Connection reset by peer
```
Verify that the Zabbix Server IP is explicitly allowed in the agent's `/etc/zabbix/zabbix_agent2.conf` file under `Server=`.

### 13.3. Diagnostic Commands
Test socket access from Zabbix Server to Agent:
```bash
nc -vz <AGENT_IP> 10050
```

Review log output:
```bash
# On Agent host:
sudo journalctl -u zabbix-agent2 --since "30 minutes ago"

# On Server host:
sudo journalctl -u zabbix-server --since "30 minutes ago"
```

---

## 14. Summary of Issues Encountered & Solutions

| Issue | Root Cause | Solution |
| :--- | :--- | :--- |
| **SMB System error 67** | Incorrect path, share name, or DNS failure | Correct UNC path syntax, verify DNS resolution, check `Get-SmbShare` |
| **Gray ZBX status** | Agent communication unconfirmed | Verify agent service status, network path, host settings, and port 10050 |
| **Connection reset by peer** | Connection refused by host firewall or agent | Add server IP to agent's `Server=` config, adjust firewall rules |
| **Agent unreachable** | Service down, listening on loopback, or network blocked | Adjust network binding, verify `systemctl` status, check routes |
| **Misuse of `127.0.0.1`** | Loopback assigned for remote target host | Replace loopback address with host's actual network IP |
| **Package available but offline** | Repository added without package installation | Execute `apt install` and enable systemd unit |

### Systematic Troubleshooting Methodology
1. **Service Layer:** `systemctl status <service>`
2. **Configuration Layer:** Audit IP bindings, hostnames, and config files.
3. **Network Layer:** `ping -c 4 <ip>`
4. **Transport Layer:** `ss -lntp` and `nc -vz <ip> <port>`
5. **Application Layer:** Validate settings across server/agent pair.
6. **Logging Layer:** Check `journalctl` for exact error events.

---

## 15. Potential Future Improvements

- [ ] Build out a multi-tiered Active Directory OU and GPO structure.
- [ ] Implement an NTFS/Share permissions matrix.
- [ ] Add trigger thresholds for CPU, Memory, and Storage inside Zabbix.
- [ ] Configure alert integrations (Email, Webhooks, Slack).
- [ ] Create automated system verification scripts via PowerShell & Bash.

---

## 16. Official References

* [Microsoft Learn Documentation](https://learn.microsoft.com/)
* [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
* [Zabbix Documentation & Downloads](https://www.zabbix.com/documentation/current/)

---

> **Author Note:** This repository serves as a practical lab for systems administration training (ASIR framework) covering Directory Services, Network File Sharing, and Infrastructure Monitoring.