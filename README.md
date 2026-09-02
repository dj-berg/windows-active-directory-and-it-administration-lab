# 🖥️ Windows Active Directory & IT Administration Lab

A hands-on Windows IT lab using Microsoft Azure, Windows Server 2022, and Windows 11 to practice Active Directory administration, user support, Group Policy, DNS, and network file sharing.

I created a small Windows domain environment with a domain controller and employee workstation, then used it to practice common IT administration and Help Desk tasks.

---

## 🛠️ Technologies Used

- Microsoft Azure
- Windows Server 2022
- Windows 11 Pro
- Active Directory Domain Services (AD DS)
- Active Directory Users and Computers (ADUC)
- Group Policy
- DNS
- PowerShell
- Remote Desktop (RDP)
- Windows File Sharing
- TCP/IP

---

## ☁️ 1. Build the Azure Environment

I started by creating two virtual machines in Microsoft Azure:

- **dc-1** — Windows Server 2022
- **client-1** — Windows 11 Pro

Both computers were connected through the same Azure virtual network.

```text
Microsoft Azure
      │
Active-Directory-VNet
      │
      ├── dc-1
      │   Windows Server 2022
      │
      └── client-1
          Windows 11 Pro
```

I assigned `dc-1` the static private IP address:

```text
10.0.1.4
```

I then configured `client-1` to use `dc-1` as its DNS server.

![Azure Active Directory Lab Environment](images/azure-environment.png)

---

## 🌐 2. Test Network Connectivity

Before configuring Active Directory, I made sure the client could communicate with the server.

From `client-1`, I ran:

```powershell
ping 10.0.1.4
```

The test returned **0% packet loss**, confirming that the two virtual machines could communicate across the Azure network.

![Client to Domain Controller Connectivity](images/client-dc-connectivity.png)

> **Lab Note:** Windows Firewall profiles on `dc-1` were temporarily disabled in this isolated lab environment. In a production environment, the firewall should remain enabled and only necessary traffic should be allowed.

---

## 🏢 3. Set Up Active Directory

Next, I installed **Active Directory Domain Services (AD DS)** on `dc-1` and promoted the server to a domain controller.

I created the domain:

```text
mydomain.com
```

This allowed `dc-1` to centrally manage users, computers, authentication, and security settings.

I organized the domain using Organizational Units (OUs):

```text
mydomain.com
│
├── _ADMINS
├── _EMPLOYEES
├── _CLIENTS
└── _GROUPS
```

I also created a dedicated administrator account and used a provided PowerShell script to generate employee accounts for the environment.

![Active Directory Domain Structure](images/active-directory-structure.png)

---

## 💻 4. Join the Windows 11 Client to the Domain

Once the domain was running, I joined `client-1` to:

```text
mydomain.com
```

The workstation was then added to the `_CLIENTS` OU in Active Directory.

This allowed domain users to sign into the Windows 11 workstation using accounts managed from `dc-1`.

With the client joined to the domain, I practiced several common user administration tasks:

- Creating and managing user accounts
- Resetting employee passwords
- Unlocking locked accounts
- Disabling and re-enabling accounts
- Managing security group membership
- Testing employee domain logins
- Reviewing account activity with Event Viewer

---

## 🔐 5. Configure and Test Account Lockout Policy

I used Group Policy to create an account lockout policy for domain users.

| Setting | Configuration |
|---|---|
| **Lockout Threshold** | 5 invalid login attempts |
| **Lockout Duration** | 30 minutes |
| **Reset Lockout Counter After** | 10 minutes |

On `client-1`, I updated Group Policy with:

```powershell
gpupdate /force
```

I then intentionally entered an incorrect password enough times to trigger the policy.

The employee account became locked as expected.

Using Active Directory, I located the employee account, unlocked it, and confirmed that the employee could log in again.

![Group Policy Account Lockout Settings](images/account-lockout-policy.png)

This simulated a common Help Desk request where a user becomes locked out of their account and needs an administrator to restore access.

---

## 🌐 6. Configure and Troubleshoot DNS

Next, I practiced DNS configuration and troubleshooting.

From `client-1`, I initially tried:

```powershell
ping mainframe
```

The hostname could not be resolved.

On `dc-1`, I opened DNS Manager and created an **A record**:

```text
mainframe → 10.0.1.4
```

I tested the hostname again from `client-1` and it successfully resolved:

```text
mainframe.mydomain.com → 10.0.1.4
```

![DNS Name Resolution Test](images/dns-mainframe-resolution.png)

I also practiced clearing the local DNS cache:

```powershell
ipconfig /flushdns
```

and created a CNAME alias:

```text
search → www.google.com
```

The troubleshooting process was simple:

```text
Hostname does not resolve
        │
        ▼
Check DNS configuration
        │
        ▼
Create DNS record
        │
        ▼
Test again
        │
        ▼
Hostname resolves ✓
```

---

## 📁 7. Create Network File Shares

With the domain and DNS working, I created shared folders on `dc-1` and tested access from the employee workstation.

I created several shares with different permissions:

| Share | Users | Access |
|---|---|---|
| `read-access` | Domain Users | Read |
| `write-access` | Domain Users | Read/Write |
| `no-access` | Domain Admins | Read/Write |
| `accounting` | ACCOUNTANTS | Read/Write |

From `client-1`, I accessed the server using:

```text
\\dc-1
```

I tested the permissions using an employee account.

The employee could:

- View files in `read-access`
- Create files in `write-access`
- Not access the restricted `no-access` folder

![Network File Shares](images/network-file-shares.png)

This showed how different permissions can be used to control what employees are allowed to do with shared company resources.

---

## 👥 8. Control Access with Security Groups

Finally, I created an Active Directory security group called:

```text
ACCOUNTANTS
```

The `accounting` network share was configured so that members of this group received read/write access.

I first attempted to access the folder using an employee who was **not** a member of `ACCOUNTANTS`.

Access was denied.

I then:

1. Added the employee to the `ACCOUNTANTS` group
2. Signed out of `client-1`
3. Signed back in
4. Opened the Accounting share again

The employee could now access the folder and create and edit a test file.

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

This simulated a common IT request where an employee needs access to a department resource and access is granted through the correct security group.

---

## 🧠 What I Learned

This project helped me understand how the main parts of a Windows business environment work together.

I gained hands-on experience with:

- Creating Windows virtual machines in Microsoft Azure
- Configuring a Windows Server domain controller
- Creating and managing Active Directory users and OUs
- Joining Windows workstations to a domain
- Resetting passwords and unlocking user accounts
- Applying account policies through Group Policy
- Configuring and troubleshooting DNS
- Creating network file shares
- Managing file and folder permissions
- Granting access through Active Directory security groups
- Testing and troubleshooting from an employee workstation

The biggest takeaway was seeing how **Active Directory, DNS, Group Policy, user accounts, workstations, and network resources all work together in a Windows domain environment.**

---

## 📁 Repository Structure

```text
windows-active-directory-and-it-administration-lab/
│
├── README.md
│
└── images/
    ├── azure-environment.png
    ├── client-dc-connectivity.png
    ├── active-directory-structure.png
    ├── account-lockout-policy.png
    ├── dns-mainframe-resolution.png
    ├── network-file-shares.png
    └── accounting-group-access.png
```
