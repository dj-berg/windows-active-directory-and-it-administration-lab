# 🖥️ Windows Active Directory & IT Administration

A hands-on Windows IT environment built in **Microsoft Azure** to practice **Active Directory, Windows Server administration, DNS, Group Policy, user support, and network resource access**.

---

## 🎯 Project Overview

This project simulates a small company's Windows IT environment using a **Windows Server 2022 domain controller** and a **Windows 11 Pro employee workstation** hosted in Microsoft Azure.

The server acts as the central system for managing employee accounts, computers, security settings, and shared resources. The Windows 11 workstation represents a computer that an employee would use within the company.

I used this environment to practice common IT support and administration tasks, including:

- 👤 Creating and managing employee accounts
- 💻 Connecting employee computers to a company domain
- 🔑 Resetting passwords and unlocking accounts
- 🔐 Applying account security policies
- 🌐 Configuring and troubleshooting DNS and network connectivity
- 👥 Managing employee access through security groups
- 📁 Controlling access to shared company folders

The project demonstrates how several core Windows technologies work together to centrally manage and support users in a business environment.

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
| **Cloud Platform** | Microsoft Azure |
| **Server** | Windows Server 2022 |
| **Client** | Windows 11 Pro |
| **Directory Services** | Active Directory Domain Services (AD DS) |
| **Networking** | TCP/IP, Azure Virtual Network, DNS |
| **Security Policies** | Group Policy |
| **Administration** | ADUC, DNS Manager, Event Viewer |
| **Command Line** | PowerShell |
| **Remote Access** | Remote Desktop |

![Azure Active Directory Lab Environment](images/azure-environment.png)

> **Environment:** Windows Server and Windows 11 virtual machines deployed in Microsoft Azure.

---

## ☁️ Azure Infrastructure & Networking

I first created the cloud infrastructure needed for the Windows environment.

Inside Microsoft Azure, I created the `Active-Directory-Lab` resource group and `Active-Directory-VNet`, then deployed:

- 🖥️ **dc-1** — Windows Server 2022 server
- 💻 **client-1** — Windows 11 Pro employee workstation

I configured `dc-1` with the static private IP address `10.0.1.4`, giving the server a consistent address that the workstation could use to locate important network services.

I then configured `client-1` to use `dc-1` as its **DNS server** and verified the configuration with `ipconfig /all`.

To confirm that the two computers could communicate, I tested the connection from `client-1`:

```powershell
ping 10.0.1.4
```

The test returned four successful replies with **0% packet loss**, confirming that the employee workstation could communicate with the server.

![Client to Domain Controller Connectivity](images/client-dc-connectivity.png)

> **Connectivity verification:** `client-1` successfully communicating with `dc-1` across the Azure virtual network.

> ⚠️ **Lab Note:** Windows Firewall profiles on `dc-1` were temporarily disabled as part of this isolated training environment. In a production environment, the firewall should remain enabled with only the required traffic permitted.

---

## 🏢 Active Directory & User Administration

I configured `dc-1` as a **domain controller**, allowing it to centrally manage employee accounts, computers, authentication, and security settings through **Active Directory Domain Services (AD DS)**.

I created the Windows domain:

```text
mydomain.com
```

Using **Active Directory Users and Computers (ADUC)**, I organized the environment into **Organizational Units (OUs)**, which act like folders for organizing different types of company accounts and computers.

```text
mydomain.com
│
├── _ADMINS
├── _EMPLOYEES
├── _CLIENTS
│     └── client-1
└── _GROUPS
```

I created a dedicated administrator account, joined `client-1` to the domain, and placed the workstation inside the `_CLIENTS` OU.

I also used a **provided PowerShell provisioning script** to generate employee accounts and practiced common user-support tasks such as:

- 👤 Managing employee accounts
- 🔑 Resetting passwords
- 🔓 Unlocking locked accounts
- 🚫 Disabling and re-enabling accounts
- 👥 Managing security group membership
- 💻 Testing employee domain logins
- 🔎 Reviewing security activity with Event Viewer

I used `whoami` from the employee workstation to verify that users were successfully authenticating through the domain.

![Active Directory Domain Structure](images/active-directory-structure.png)

> **Active Directory:** Employee accounts, computers, administrators, and security groups organized within the `mydomain.com` domain.

---

## 🔐 Group Policy & Account Security

I used **Group Policy**, which allows administrators to centrally apply settings and security rules to domain users and computers, to configure an account lockout policy.

The policy was configured to protect accounts from repeated failed login attempts:

| Setting | Configuration |
| --- | --- |
| **Account Lockout Threshold** | 5 invalid login attempts |
| **Account Lockout Duration** | 30 minutes |
| **Reset Lockout Counter After** | 10 minutes |

I applied the updated policy to `client-1` using:

```powershell
gpupdate /force
```

I then intentionally entered an incorrect password multiple times with a test employee account. After five failed attempts, the account became locked as expected.

I approached the issue as an IT administrator by locating the employee in Active Directory, unlocking the account, and confirming that the employee could successfully log in again.

```text
Failed Login Attempts
        │
        ▼
  Account Locked
        │
        ▼
  Locate User in AD
        │
        ▼
   Unlock Account
        │
        ▼
  Verify User Login ✓
```

![Group Policy Account Lockout Settings](images/account-lockout-policy.png)

> **Account security:** A domain-wide account lockout policy configured and tested through Group Policy.

This exercise simulated a common **Help Desk scenario**: restoring access for an employee whose account became locked after too many failed login attempts.

---

## 🌐 DNS & Name Resolution

I configured and tested **DNS**, the service that allows computers to find network resources by name instead of requiring users to remember IP addresses.

For example, `client-1` initially could not find the hostname:

```powershell
ping mainframe
```

Using DNS Manager on `dc-1`, I created an **A record** connecting the name `mainframe` to the server's IP address:

```text
mainframe → 10.0.1.4
```

After creating the record, I tested the hostname again from `client-1`.

Windows successfully translated:

```text
mainframe.mydomain.com → 10.0.1.4
```

and communicated with the server.

During troubleshooting, I also cleared previously stored DNS information from the workstation using:

```powershell
ipconfig /flushdns
```

I also practiced creating a **CNAME alias**, which allows one hostname to act as an alternate name for another:

```text
search → www.google.com
```

![DNS Name Resolution Test](images/dns-mainframe-resolution.png)

> **DNS verification:** `client-1` successfully resolving `mainframe.mydomain.com` to `10.0.1.4` through the DNS server.

The basic troubleshooting process was:

**Name does not resolve → check DNS → configure the record → retest → verify the fix.**

---

## 📁 File Shares & Access Control

I configured shared network folders on `dc-1` to practice controlling which employees could access company resources.

Different folders were given different access levels:

| Resource | Who Can Access It | Access |
| --- | --- | --- |
| `read-access` | Domain Users | Read |
| `write-access` | Domain Users | Read/Write |
| `no-access` | Domain Admins | Read/Write |
| `accounting` | ACCOUNTANTS | Read/Write |

From the employee workstation, I connected to the server using:

```text
\\dc-1
```

I then tested the permissions from an employee account:

- 👁️ `read-access` — employee could view files but could not create new ones
- ✏️ `write-access` — employee could create files
- 🚫 `no-access` — employee was denied access

![Network File Shares](images/network-file-shares.png)

> **Network shares:** Shared company resources hosted on `dc-1` and accessed from the employee workstation.

### 👥 Controlling Access with Security Groups

I also created an Active Directory security group named `ACCOUNTANTS`.

The goal was simple:

> Only employees who belong to the `ACCOUNTANTS` group should be able to access the Accounting folder.

A test employee initially could not access the folder because the account was not part of the group.

I added the employee to `ACCOUNTANTS`, signed out and back into `client-1` to refresh the user's access, and tested the folder again.

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

The employee could now open the Accounting folder and create and edit a test file.

![Accounting Group Resource Access](images/accounting-group-access.png)

> **Access verification:** An authorized employee successfully creating and editing a file within the Accounting network share.

This simulated a common IT support request where an employee needs access to a department resource and an administrator grants that access through the appropriate security group.

---

## 💡 Key Takeaways

This project gave me hands-on experience supporting a **Windows business environment** from both the administrator and employee perspectives.

I practiced:

- 🏢 **Active Directory** — Managing employee accounts, computers, groups, and organizational units
- 👤 **User Support** — Resetting passwords, unlocking accounts, and resolving access issues
- 🔐 **Group Policy** — Applying centralized account security rules
- 🌐 **DNS & Networking** — Connecting systems and troubleshooting name resolution
- 📁 **File Sharing** — Creating shared company resources with different access levels
- 👥 **Access Control** — Using security groups to give employees access based on their role
- ☁️ **Microsoft Azure** — Hosting and connecting Windows Server and Windows client systems
- 🛠️ **Troubleshooting** — Identifying an issue, applying a solution, and verifying that the solution worked

Most importantly, this project helped me understand how **user accounts, employee computers, authentication, security policies, DNS, and shared resources work together in a Windows domain environment**.

### 📚 Project Context

This project was completed as part of hands-on **CourseCareers IT training** and documented as a portfolio project to demonstrate the technical skills and support workflows I practiced.

The environment is an **educational lab** that simulates common Windows IT administration and Help Desk responsibilities rather than a production deployment.
