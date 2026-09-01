# 🏢 Windows Active Directory & IT Administration

A hands-on Windows administration project built in **Microsoft Azure** to simulate a small enterprise domain environment. This project covers the deployment and administration of **Active Directory Domain Services (AD DS), DNS, Group Policy, domain users and groups, Windows client authentication, and network file shares and permissions**.

The environment consists of a **Windows Server 2022 domain controller** and a **Windows 11 Pro client workstation**, allowing configurations to be administered from the server and tested from an end-user workstation.

---

## 📋 Project Overview

The goal of this project was to build and administer a functional Windows domain environment while practicing common **IT support and system administration responsibilities**.

During the project, I:

- Deployed Windows Server 2022 and Windows 11 Pro virtual machines in Microsoft Azure
- Configured private IP addressing and DNS communication between the virtual machines
- Installed Active Directory Domain Services and created a Windows domain
- Joined a Windows 11 workstation to the domain
- Created and managed Organizational Units (OUs), users, administrators, computers, and security groups
- Provisioned multiple employee accounts using a PowerShell script
- Configured and tested an account lockout policy through Group Policy
- Practiced account unlocks, password resets, account disabling, and account re-enabling
- Configured and tested DNS A and CNAME records
- Created network file shares with different access levels
- Implemented security group-based access to an Accounting network share
- Tested and troubleshot configurations from a domain-joined client workstation

---

## 🏗️ Environment & Architecture

The lab environment was hosted in **Microsoft Azure** using two virtual machines connected through the same Azure Virtual Network.

```text
                         Microsoft Azure
                               │
                    Active-Directory-VNet
                               │
              ┌────────────────┴────────────────┐
              │                                 │
            DC-1                            CLIENT-1
     Windows Server 2022                 Windows 11 Pro
      Domain Controller                Domain Workstation
      Static Private IP
              │                                 │
              │          mydomain.com           │
              ├─────────────────────────────────┤
              │                                 │
       Active Directory                  Domain Authentication
       DNS                               DNS Resolution
       Group Policy                      File Share Access
       Network Shares                    Policy Testing
```

### DC-1 — Domain Controller

`dc-1` was configured as the central server for the environment and provided:

- Active Directory Domain Services
- Domain authentication
- DNS
- Group Policy
- User and group administration
- Network file shares

### Client-1 — Domain Workstation

`client-1` was configured as a Windows 11 Pro workstation and used to simulate an employee computer.

It was used to test:

- Domain authentication
- DNS resolution
- Group Policy
- Account lockouts
- Remote Desktop access
- Network file share permissions
- Security group-based access

> ### 📸 SCREENSHOT 1 — Azure Lab Infrastructure
> **Take a screenshot of:** Microsoft Azure showing both the `dc-1` and `client-1` virtual machines.
>
> **The screenshot should clearly show:** Both VM names and that they exist inside your Azure environment. Avoid displaying public IP addresses, subscription IDs, or credentials.
>
> **Purpose:** Proves that the Windows Server and Windows 11 systems used throughout the project were deployed in Azure.

---

## 🛠️ Technologies & Skills

### Technologies

- Microsoft Azure
- Windows Server 2022
- Windows 11 Pro
- Active Directory Domain Services (AD DS)
- DNS
- Group Policy
- PowerShell
- Command Prompt
- Remote Desktop
- Windows Event Viewer
- Windows File Sharing

### Skills Demonstrated

- Windows Server administration
- Active Directory administration
- Domain user management
- Organizational Unit management
- Security group management
- Domain joining
- DNS configuration and troubleshooting
- Group Policy configuration
- Account lockout troubleshooting
- Password resets
- User account enable/disable management
- Network file sharing
- Access control and permissions
- TCP/IP connectivity troubleshooting
- Windows security event inspection

---

# 🚀 Implementation

## 1. Azure Infrastructure & Network Configuration

I began by creating the infrastructure required to host the Windows domain environment.

In Microsoft Azure, I created:

- **Resource Group:** `Active-Directory-Lab`
- **Virtual Network:** `Active-Directory-VNet`
- **Windows Server 2022 VM:** `dc-1`
- **Windows 11 Pro VM:** `client-1`

The two virtual machines were connected to the same Azure Virtual Network so they could communicate over their private IP addresses.

### Configuring the Domain Controller's Network

I changed the private IP configuration of `dc-1` from **dynamic to static**.

Because `dc-1` would provide DNS and Active Directory services to domain clients, maintaining a consistent private IP address allows clients to reliably locate the server.

I then configured the DNS settings of `client-1` to use the private IP address of `dc-1` as its DNS server.

```text
CLIENT-1
    │
    │ DNS Requests
    ▼
  DC-1
10.0.1.4
```

After restarting `client-1` to apply the network changes, I tested connectivity from the client to the server using PowerShell:

```powershell
ping 10.0.1.4
```

The test returned four successful replies with **0% packet loss**, confirming network communication between the two virtual machines.

I also used:

```powershell
ipconfig /all
```

to verify that `client-1` was configured to use `dc-1` as its DNS server.

> ### 📸 SCREENSHOT 2 — Client-to-Server Connectivity
> **Take a screenshot of:** PowerShell on `client-1` after running `ping 10.0.1.4`.
>
> **The screenshot should clearly show:** Four successful replies and the ping statistics showing `0% loss`.
>
> **Purpose:** Demonstrates successful private network connectivity between the Windows 11 workstation and Domain Controller.

---

## 2. Active Directory Domain Deployment

With network communication established, I installed the **Active Directory Domain Services (AD DS)** server role on `dc-1`.

I then promoted `dc-1` to a **Domain Controller** and created a new forest with the domain:

```text
mydomain.com
```

This established centralized domain services for the lab environment.

Instead of managing `dc-1` and `client-1` as completely independent Windows computers, Active Directory could now centrally manage users, computers, authentication, groups, and policies.

---

## 3. Organizational Units & Domain Administration

Using **Active Directory Users and Computers**, I created Organizational Units to logically separate different types of domain objects.

The domain was organized into:

```text
mydomain.com
│
├── _ADMINS
├── _EMPLOYEES
├── _CLIENTS
└── _GROUPS
```

The OUs were used for different purposes:

- `_ADMINS` — Administrative user accounts
- `_EMPLOYEES` — Standard domain employee accounts
- `_CLIENTS` — Domain-joined client computers
- `_GROUPS` — Security groups used for resource access

### Creating a Domain Administrator

Inside `_ADMINS`, I created a dedicated administrative account and added it to the built-in:

```text
Domain Admins
```

security group.

I then authenticated to `dc-1` using the domain administrator account and used it for subsequent domain administration.

### Joining Client-1 to the Domain

`client-1` initially belonged to a standard Windows workgroup.

I changed its membership from the workgroup to:

```text
mydomain.com
```

and authenticated the domain join using the administrator account.

After successfully joining the domain, the `client-1` computer object appeared in Active Directory. I moved it from the default Computers container into the `_CLIENTS` Organizational Unit.

> ### 📸 SCREENSHOT 3 — Active Directory OU Structure
> **Take a screenshot of:** Active Directory Users and Computers on `dc-1`.
>
> **The screenshot should clearly show:** `mydomain.com` expanded with `_ADMINS`, `_EMPLOYEES`, `_CLIENTS`, and `_GROUPS` visible.
>
> If possible, select `_CLIENTS` so that `client-1` is visible in the right pane.
>
> **Purpose:** This is one of the most important screenshots in the project. It proves that the Active Directory domain exists and shows how you organized its users, computers, and groups.

---

## 4. Domain User Provisioning & Authentication

After establishing the domain, I configured `client-1` to allow members of **Domain Users** to remotely access the workstation.

To populate the domain with simulated employees, I executed a provided **PowerShell user-provisioning script** from `dc-1`.

The generated employee accounts were placed inside:

```text
_EMPLOYEES
```

This created multiple standard domain users that could be used to test authentication, Group Policy, account management, and permissions.

> ### 📸 SCREENSHOT 4 — Domain Employee Accounts
> **Take a screenshot of:** Active Directory Users and Computers with `_EMPLOYEES` selected.
>
> **The screenshot should clearly show:** Multiple generated employee accounts in the right pane.
>
> **Purpose:** Demonstrates domain-user provisioning and centralized employee account management.

### Testing Domain Authentication

I selected one of the generated employee accounts and used it to sign into `client-1`.

```text
Employee Credentials
        │
        ▼
     CLIENT-1
        │
        ▼
       DC-1
        │
        ▼
Active Directory
        │
        ▼
Successful Authentication
```

The successful login verified that `client-1` was communicating with the Domain Controller and could authenticate domain employees through Active Directory.

> ### 📸 SCREENSHOT 5 — Domain-Authenticated Employee
> **Take a screenshot of:** `client-1` after successfully logging in as one of the generated domain users.
>
> A good option is to open Command Prompt or PowerShell and run:
>
> ```powershell
> whoami
> ```
>
> **The screenshot should clearly show:** The domain and employee username returned by `whoami`.
>
> **Purpose:** Proves that a regular domain employee can authenticate to the domain-joined Windows 11 workstation.

---

# 🔐 Group Policy & Account Administration

## 5. Configuring an Account Lockout Policy

I used **Group Policy Management** on `dc-1` to configure an account lockout policy through the Default Domain Policy.

The environment was configured to lock an account after:

```text
Failed Login Threshold: 5 attempts
Lockout Duration:       30 minutes
```

I then forced `client-1` to retrieve the updated Group Policy settings using:

```powershell
gpupdate /force
```

This allowed the new security policy to take effect without waiting for the normal Group Policy refresh interval.

> ### 📸 SCREENSHOT 6 — Group Policy Account Lockout
> **Take a screenshot of:** Group Policy Management / Group Policy Management Editor showing the Account Lockout Policy settings.
>
> **The screenshot should clearly show:** The `5` failed-attempt threshold and `30 minute` lockout setting.
>
> **Purpose:** Demonstrates that you configured a domain-wide security policy rather than relying only on default Windows settings.

---

## 6. Testing & Troubleshooting Account Lockouts

To verify that the new policy worked, I intentionally entered an incorrect password repeatedly for a test employee account on `client-1`.

```text
Incorrect Password Attempts
           │
           ▼
5 Failed Authentication Attempts
           │
           ▼
      Account Locked
```

The account became locked as expected.

From `dc-1`, I located the affected employee through **Active Directory Users and Computers** and manually unlocked the account.

I then successfully authenticated to `client-1` again and used:

```powershell
whoami
```

to verify the logged-in domain account.

This simulated a common Help Desk scenario in which an employee becomes locked out of their account and requires assistance from IT.

> ### 📸 SCREENSHOT 7 — Locked Account / Account Recovery
> **Take a screenshot of:** Active Directory Users and Computers showing the test employee's Account properties after the lockout.
>
> **Ideally show:** The account lockout/unlock area or the account immediately before you unlock it.
>
> **Purpose:** Demonstrates hands-on Active Directory account troubleshooting and recovery.
>
> **Alternative:** If the account is no longer locked and recreating it is inconvenient, use a screenshot showing the employee's Account properties in AD rather than manufacturing a fake lockout.

### Additional Account Administration

I also practiced common Active Directory account-management tasks, including:

- Resetting employee passwords
- Disabling user accounts
- Re-enabling user accounts
- Unlocking locked accounts

I opened **Windows Event Viewer** using:

```text
eventvwr.msc
```

to explore Windows security logs and see how authentication and security-related activity can be investigated.

---

# 🌐 DNS Administration & Troubleshooting

## 7. Creating DNS A Records

Because `dc-1` also provided DNS services for the domain, I used **DNS Manager** to configure and test name resolution.

Initially, `client-1` could not resolve the hostname:

```text
mainframe
```

I created a new **Host (A) record** on `dc-1` mapping:

```text
mainframe
    │
    ▼
10.0.1.4
```

After creating the record, I tested name resolution from `client-1` using:

```powershell
ping mainframe
```

The hostname successfully resolved to the configured IP address.

> ### 📸 SCREENSHOT 8 — DNS Records
> **Take a screenshot of:** DNS Manager on `dc-1`.
>
> **The screenshot should clearly show:** Your `mainframe` A record. If possible, also have the `search` CNAME record visible in the same screenshot.
>
> **Purpose:** Demonstrates that you created and administered DNS records on the Windows DNS server.

---

## 8. DNS Cache & Name Resolution Troubleshooting

While testing DNS, I also worked with the local Windows DNS resolver cache.

I used:

```powershell
ipconfig /displaydns
```

to inspect locally cached DNS information and:

```powershell
ipconfig /flushdns
```

to clear cached records and force the client to request updated DNS information.

This demonstrated how cached DNS information can affect troubleshooting after DNS records have been changed.

I also used commands such as:

```powershell
hostname
ipconfig
ping
```

to inspect the workstation and test connectivity and name resolution.

> ### 📸 SCREENSHOT 9 — Successful DNS Resolution
> **Take a screenshot of:** PowerShell or Command Prompt on `client-1` after running:
>
> ```powershell
> ping mainframe
> ```
>
> **The screenshot should clearly show:** `mainframe` resolving to `10.0.1.4` and receiving successful replies.
>
> **Purpose:** Screenshot 8 proves you configured DNS; this screenshot proves that the DNS configuration actually worked from the client.

---

## 9. Creating a CNAME Record

I also created a **CNAME (Canonical Name) record** to practice working with DNS aliases.

The alias:

```text
search
```

was configured to reference:

```text
www.google.com
```

I then tested the alias from `client-1` and verified that it resolved successfully.

This demonstrated the difference between two common DNS record types:

```text
A Record
Hostname ──────────► IP Address

CNAME Record
Alias ─────────────► Canonical Hostname
```

---

# 📁 Network File Shares & Permissions

## 10. Creating Network File Shares

On `dc-1`, I created several folders to simulate organizational network resources:

```text
\\dc-1
│
├── read-access
├── write-access
├── no-access
└── accounting
```

Each folder was shared with different access requirements.

| Network Share | Authorized Users | Access |
| --- | --- | --- |
| `read-access` | Domain Users | Read |
| `write-access` | Domain Users | Read/Write |
| `no-access` | Domain Admins | Read/Write |
| `accounting` | ACCOUNTANTS | Read/Write |

From `client-1`, I connected to the server using:

```text
\\dc-1
```

and tested each share while authenticated as a regular domain employee.

> ### 📸 SCREENSHOT 10 — Network Shares from Client-1
> **Take a screenshot of:** File Explorer on `client-1` after navigating to `\\dc-1`.
>
> **The screenshot should clearly show:** `read-access`, `write-access`, `no-access`, and `accounting`.
>
> **Purpose:** Demonstrates that the Windows Server is providing network resources that are discoverable from the domain workstation.

---

## 11. Testing File Share Permissions

I tested each permission level from the perspective of a regular domain employee.

### Read-Only Share

The employee could open the `read-access` share but could not create new files.

```text
Employee
   │
   ▼
read-access
   │
   ├── ✓ Read
   └── ✗ Write
```

### Read/Write Share

The employee could access `write-access` and successfully create files.

```text
Employee
   │
   ▼
write-access
   │
   ├── ✓ Read
   └── ✓ Write
```

### Restricted Share

The regular employee could not access `no-access`, while Domain Administrators were permitted.

```text
Employee ──────────► no-access ──► ✗ ACCESS DENIED

Domain Admin ──────► no-access ──► ✓ ACCESS
```

These tests verified that the configured permissions were actually being enforced from the end-user workstation.

> ### 📸 SCREENSHOT 11 — File Permission Enforcement
> **Take a screenshot of:** One of your permission tests from `client-1`.
>
> **Best option:** Attempt to open `no-access` while logged in as a standard domain employee and capture the Windows access-denied message.
>
> **Alternative:** Show the employee successfully creating a file inside `write-access`.
>
> **Purpose:** Provides visible proof that the permissions configured on the server were being enforced for domain users.

---

## 12. Security Group-Based Access Control

I created the `_GROUPS` Organizational Unit and created an Active Directory security group named:

```text
ACCOUNTANTS
```

I then configured the `accounting` network share so that members of the `ACCOUNTANTS` security group received **Read/Write** access.

Initially, a regular employee who was not a member of the group could not access the Accounting share.

```text
Employee
   │
   ▼
ACCOUNTANTS membership?
   │
   └── No
       │
       ▼
Accounting Share
       │
       ✗ ACCESS DENIED
```

I then added the employee to:

```text
ACCOUNTANTS
```

After logging out and back into `client-1` so that the updated group membership would apply:

```text
Employee
   │
   ▼
ACCOUNTANTS
   │
   ▼
Accounting Share
   │
   ✓ READ / WRITE
```

the employee could successfully access the Accounting folder.

This demonstrated how **Active Directory security groups can be used to control access to organizational resources based on an employee's role**.

> ### 📸 SCREENSHOT 12 — ACCOUNTANTS Group-Based Access
> **Take a screenshot of:** Active Directory Users and Computers with the `ACCOUNTANTS` group properties open on the **Members** tab.
>
> **The screenshot should clearly show:** Your test employee listed as a member of `ACCOUNTANTS`.
>
> **Even better:** If you want to combine two images here, follow it with a small screenshot showing that same employee successfully accessing the `accounting` share from `client-1`.
>
> **Purpose:** Demonstrates role/group-based access control rather than assigning permissions individually to every employee.

---

# 🔧 IT Administration & Troubleshooting Scenarios

The environment was also used to practice several scenarios similar to issues an IT support technician or system administrator could encounter.

## 🔐 Scenario 1 — Employee Account Lockout

**Problem:**  
An employee exceeded the domain's permitted number of failed login attempts and became locked out.

**Investigation:**  
Located the employee account in Active Directory and confirmed the account lockout.

**Resolution:**  
Unlocked the account from the Domain Controller and verified that the employee could authenticate successfully again.

**Skills demonstrated:**  
`Active Directory` · `Group Policy` · `Account Management` · `Authentication Troubleshooting`

---

## 🌐 Scenario 2 — DNS Name Resolution

**Problem:**  
`client-1` could not initially resolve the hostname `mainframe`.

**Investigation:**  
The required DNS record did not exist on the DNS server.

**Resolution:**  
Created an A record on `dc-1` and verified successful name resolution from `client-1`.

**Skills demonstrated:**  
`DNS` · `Windows Server` · `TCP/IP` · `Name Resolution` · `Troubleshooting`

---

## 📁 Scenario 3 — Employee Cannot Access Accounting Share

**Problem:**  
A domain employee could not access the Accounting network share.

**Investigation:**  
The employee was not a member of the `ACCOUNTANTS` security group that had permission to the resource.

**Resolution:**  
Added the employee to `ACCOUNTANTS`, refreshed the user's session by signing out and back in, and verified successful access to the Accounting share.

**Skills demonstrated:**  
`Active Directory` · `Security Groups` · `File Sharing` · `Permissions` · `Access Control`

---

# 💻 Commands Used

Several Windows networking and administration commands were used throughout the project.

```powershell
# Test network connectivity
ping 10.0.1.4

# View network configuration
ipconfig
ipconfig /all

# View locally cached DNS records
ipconfig /displaydns

# Clear the local DNS resolver cache
ipconfig /flushdns

# Force Group Policy to update
gpupdate /force

# Display the currently authenticated user
whoami

# Display the computer hostname
hostname
```

Administrative tools used throughout the project included:

```text
Server Manager
Active Directory Users and Computers
DNS Manager
Group Policy Management
Windows Event Viewer
PowerShell / PowerShell ISE
Remote Desktop Connection
```

---

# 💡 Key Takeaways

This project gave me hands-on experience building and administering a Windows domain environment from both the **administrator and end-user perspectives**.

Key concepts I practiced included:

- Deploying Windows infrastructure in Microsoft Azure
- Understanding the relationship between Active Directory and DNS
- Centrally managing domain users and computers
- Organizing domain resources with Organizational Units
- Managing administrative privileges and security groups
- Enforcing authentication policies through Group Policy
- Troubleshooting account lockouts and authentication problems
- Creating and troubleshooting DNS records
- Understanding and managing the Windows DNS cache
- Configuring network file shares
- Controlling resource access through group membership and permissions
- Testing administrative configurations from an end-user workstation

Most importantly, the project demonstrated how several Windows enterprise technologies work together:

```text
Active Directory
      │
      ├── Centralized Identity & Authentication
      │
DNS ──┼── Name Resolution
      │
GPO ──┼── Domain Security Policies
      │
Groups┼── Role-Based Access
      │
Shares┴── Organizational Resources
```

Rather than treating these technologies as isolated concepts, I was able to configure and test how they interact inside a functional Windows domain environment.

---

# 📚 Project Background

This project was completed as part of hands-on IT training through **CourseCareers**. The environment was used to apply Windows administration concepts in a simulated enterprise setting.

The configurations, testing, troubleshooting, and documentation shown in this repository reflect the hands-on work I completed while building and administering the environment.

---

# 🔒 Security

All credentials used during the lab were created specifically for the temporary lab environment and have been intentionally excluded from this repository.

Screenshots included in this documentation are reviewed before publication to avoid exposing:

- Passwords
- Public IP addresses
- Azure subscription information
- Sensitive account information
- Unnecessary identifying information

---
