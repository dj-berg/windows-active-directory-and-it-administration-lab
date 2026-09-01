# 🖥️ Windows Active Directory & IT Administration Lab

A hands-on Windows IT environment built in **Microsoft Azure** to practice **Active Directory, Windows Server administration, DNS, Group Policy, user support, and network resource access**.

---

## 🎯 Project Overview

This project simulates a small company's Windows IT environment using a **Windows Server 2022 domain controller** and a **Windows 11 Pro employee workstation** hosted in Microsoft Azure.

The server acts as the central system for managing employee accounts, computers, security settings, and shared resources. I used the workstation to test the environment from an employee's perspective.

Through this project, I practiced common IT support and administration tasks:

- 👤 Creating and managing employee accounts
- 💻 Joining a workstation to a Windows domain
- 🔑 Resetting passwords and unlocking accounts
- 🔐 Applying account security policies
- 🌐 Configuring and troubleshooting DNS and network connectivity
- 👥 Managing access with security groups
- 📁 Controlling access to shared network folders

The result is a working Windows domain environment that demonstrates how core IT services work together to manage and support users.

---

## 🏗️ Environment & Technologies

The environment consists of two Windows virtual machines connected through the same **Microsoft Azure virtual network**.

```text
                       Microsoft Azure
                             │
                  Active-Directory-VNet
                             │
               ┌─────────────┴─────────────┐
               │                           │
             DC-1                      CLIENT-1
      Windows Server 2022             Windows 11 Pro
               │                           │
       Domain Controller              Employee Workstation
       Static Private IP                    │
               │                           │
               ├───── mydomain.com ───────┤
               │                           │
               ▼                           ▼
      Active Directory              Domain Authentication
      DNS                           DNS Resolution
      Group Policy                  Policy Testing
      Network Shares                Resource Access
```

### 🛠️ Technologies

| Area | Technology |
| --- | --- |
| **Cloud** | Microsoft Azure |
| **Server** | Windows Server 2022 |
| **Client** | Windows 11 Pro |
| **Directory Services** | Active Directory Domain Services (AD DS) |
| **Networking** | TCP/IP, Azure Virtual Network, DNS |
| **Security** | Group Policy |
| **Administration** | ADUC, DNS Manager, Event Viewer |
| **Command Line** | PowerShell |
| **Remote Access** | Remote Desktop |

![Azure Active Directory Lab Environment](images/azure-environment.png)

> **Environment:** Windows Server and Windows 11 virtual machines deployed in Microsoft Azure.

---

## ☁️ Azure Infrastructure & Networking

I created the `Active-Directory-Lab` resource group and `Active-Directory-VNet`, then deployed:

- 🖥️ **dc-1** — Windows Server 2022
- 💻 **client-1** — Windows 11 Pro

I assigned `dc-1` the static private IP address `10.0.1.4` so the workstation could consistently locate the server and its network services. I then configured `client-1` to use `dc-1` as its DNS server.

To verify that the two systems could communicate, I tested connectivity from `client-1`:

```powershell
ping 10.0.1.4
```

The test returned **0% packet loss**, confirming successful communication between the workstation and server.

![Client to Domain Controller Connectivity](images/client-dc-connectivity.png)

> **Connectivity:** `client-1` successfully communicating with `dc-1` across the Azure virtual network.

> ⚠️ **Lab Note:** Windows Firewall profiles on `dc-1` were temporarily disabled for this isolated training environment. In a production environment, the firewall should remain enabled with only required traffic permitted.

---

## 🏢 Active Directory & User Administration

I configured `dc-1` as a **domain controller** using **Active Directory Domain Services (AD DS)**. This allowed the server to centrally manage employee accounts, computers, authentication, and security settings.

I created the domain:

```text
mydomain.com
```

Using **Active Directory Users and Computers (ADUC)**, I organized the environment into **Organizational Units (OUs)**, which work like folders for separating different types of users and computers.

```text
mydomain.com
│
├── _ADMINS
├── _EMPLOYEES
├── _CLIENTS
│     └── client-1
└── _GROUPS
```

I created a dedicated administrator account, joined `client-1` to the domain, and placed the workstation in the `_CLIENTS` OU.

I also used a **provided PowerShell provisioning script** to generate employee accounts and practiced common support tasks:

- 👤 Managing employee accounts
- 🔑 Resetting passwords
- 🔓 Unlocking locked accounts
- 🚫 Disabling and re-enabling accounts
- 👥 Managing security group membership
- 💻 Testing employee domain logins
- 🔎 Reviewing security activity with Event Viewer

![Active Directory Domain Structure](images/active-directory-structure.png)

> **Active Directory:** Administrators, employees, computers, and security groups organized within the `mydomain.com` domain.

---

## 🔐 Group Policy & Account Security

I used **Group Policy**, which allows administrators to apply security rules across a Windows domain, to configure an account lockout policy.

| Setting | Configuration |
| --- | --- |
| **Lockout Threshold** | 5 invalid login attempts |
| **Lockout Duration** | 30 minutes |
| **Reset Lockout Counter After** | 10 minutes |

I applied the policy to `client-1` using:

```powershell
gpupdate /force
```

I then intentionally exceeded the failed-login limit with a test employee account and confirmed that the account became locked.

To simulate a common **Help Desk scenario**, I located the employee account in Active Directory, unlocked it, and verified that the employee could successfully log in again.

![Group Policy Account Lockout Settings](images/account-lockout-policy.png)

> **Account security:** Domain account lockout settings configured and tested through Group Policy.

---

## 🌐 DNS & Name Resolution

I configured and tested **DNS**, which allows computers to locate network resources by name instead of requiring users to remember IP addresses.

Initially, `client-1` could not resolve the hostname:

```powershell
ping mainframe
```

Using DNS Manager on `dc-1`, I created an **A record** connecting the hostname to the server's IP address:

```text
mainframe → 10.0.1.4
```

After creating the record, I tested it again from `client-1` and successfully resolved:

```text
mainframe.mydomain.com → 10.0.1.4
```

I also practiced clearing the DNS cache with:

```powershell
ipconfig /flushdns
```

and created a **CNAME alias** to practice alternate DNS names:

```text
search → www.google.com
```

![DNS Name Resolution Test](images/dns-mainframe-resolution.png)

> **DNS verification:** `client-1` successfully resolving `mainframe.mydomain.com` to `10.0.1.4`.

**Troubleshooting flow:** Name fails to resolve → check DNS → configure record → retest → verify the fix.

---

## 📁 File Shares & Access Control

I created shared network folders on `dc-1` to practice controlling employee access to company resources.

| Resource | Authorized Users | Access |
| --- | --- | --- |
| `read-access` | Domain Users | Read |
| `write-access` | Domain Users | Read/Write |
| `no-access` | Domain Admins | Read/Write |
| `accounting` | ACCOUNTANTS | Read/Write |

From the employee workstation, I connected to:

```text
\\dc-1
```

I verified that an employee could:

- 👁️ View but not create files in `read-access`
- ✏️ Create files in `write-access`
- 🚫 Not access the restricted `no-access` folder

![Network File Shares](images/network-file-shares.png)

> **Network shares:** Shared resources hosted on `dc-1` and accessed from the employee workstation.

### 👥 Security Group-Based Access

I created an Active Directory security group named `ACCOUNTANTS` so access to the Accounting folder could be controlled by group membership.

The test employee was initially unable to access the folder. After adding the employee to `ACCOUNTANTS` and refreshing the user's session, the employee could access the folder and create and edit a test file.

```text
Employee
   │
   ▼
ACCOUNTANTS Group
   │
   ▼
Accounting Share
   │
   ▼
Read / Write Access ✓
```

![Accounting Group Resource Access](images/accounting-group-access.png)

> **Access verification:** An authorized employee successfully creating and editing a file within the Accounting network share.

This simulated a common IT support request: **an employee needs access to a department resource, and access is granted through the appropriate security group.**

---

## 💡 Key Takeaways

This project gave me hands-on experience supporting a **Windows business environment** from both the administrator and employee perspectives.

I practiced:

- 🏢 **Active Directory** — Managing users, computers, groups, and organizational units
- 👤 **User Support** — Resetting passwords, unlocking accounts, and resolving access issues
- 🔐 **Group Policy** — Applying centralized account security rules
- 🌐 **DNS & Networking** — Connecting systems and troubleshooting name resolution
- 📁 **File Sharing** — Creating shared resources with different access levels
- 👥 **Access Control** — Managing resource access through security groups
- ☁️ **Microsoft Azure** — Hosting and connecting Windows Server and Windows client systems
- 🛠️ **Troubleshooting** — Identifying issues, applying solutions, and verifying results

Most importantly, I learned how **users, computers, authentication, security policies, DNS, and shared resources work together within a Windows domain**.

---

### 📚 Project Context

This project was completed as part of hands-on **CourseCareers IT training** and documented to demonstrate the technical skills and support workflows I practiced.

This is an **educational lab environment** designed to simulate common Windows IT administration and Help Desk responsibilities.
