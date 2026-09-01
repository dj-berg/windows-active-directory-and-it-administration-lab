# 🖥️ Windows Active Directory & IT Administration

A hands-on Windows enterprise environment built in **Microsoft Azure** to practice **Active Directory administration, Windows Server management, DNS, Group Policy, user support, and network resource access**.

---

## 🎯 Project Overview

This project simulates a small Windows business environment using a **Windows Server 2022 domain controller** and a **Windows 11 Pro client workstation** hosted in Microsoft Azure.

I built and administered the environment to practice common IT support responsibilities, including **managing domain users and computers, configuring account security policies, troubleshooting DNS and authentication issues, and controlling access to shared network resources**.

The project demonstrates how core Windows services work together to centrally manage users, computers, security, and resources within an Active Directory domain.

---

## 🏗️ Environment & Technologies

The environment consists of two Azure virtual machines connected through the same virtual network:

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
       Domain Controller              Domain Workstation
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

| Area | Technology |
| --- | --- |
| **Cloud Platform** | Microsoft Azure |
| **Server** | Windows Server 2022 |
| **Client** | Windows 11 Pro |
| **Directory Services** | Active Directory Domain Services |
| **Networking** | TCP/IP, Azure Virtual Network, DNS |
| **Policy Management** | Group Policy |
| **Administration** | ADUC, DNS Manager, Event Viewer |
| **Command Line** | PowerShell |
| **Remote Access** | Remote Desktop |

<!--
📸 SCREENSHOT 1 — AZURE ENVIRONMENT
Place here: directly below the technology table.
Capture: Azure showing both dc-1 and client-1 virtual machines.
Purpose: Proves the two-machine cloud environment was deployed.
Avoid showing: public IPs, subscription IDs, credentials, or unnecessary account information.
-->

![Azure Active Directory Lab Environment](docs/images/azure-environment.png)

> **Environment:** Windows Server and Windows 11 virtual machines deployed in Microsoft Azure.

---

## ☁️ Azure Infrastructure & Networking

I created the `Active-Directory-Lab` resource group and `Active-Directory-VNet`, then deployed:

- 🖥️ **dc-1** — Windows Server 2022
- 💻 **client-1** — Windows 11 Pro

I configured `dc-1` with the static private IP address `10.0.1.4` so it could provide a consistent address for domain and DNS services. I then configured `client-1` to use `dc-1` as its DNS server.

From `client-1`, I verified the DNS configuration with `ipconfig /all` and tested network connectivity to the server:

```powershell
ping 10.0.1.4
```

The test returned four successful replies with **0% packet loss**, confirming communication between the workstation and server.

<!--
📸 SCREENSHOT 2 — NETWORK CONNECTIVITY
Place here: immediately after the connectivity explanation.
Capture: PowerShell on client-1 showing the successful ping to 10.0.1.4.
Best screenshot: Include all four replies and the 0% packet-loss summary.
Purpose: Proves client-1 could communicate with dc-1.
-->

![Client to Domain Controller Connectivity](docs/images/client-dc-connectivity.png)

> **Connectivity verification:** `client-1` successfully communicating with `dc-1` across the Azure virtual network.

> ⚠️ **Lab Note:** The Windows Firewall profiles on `dc-1` were temporarily disabled as part of the isolated training environment. In a production environment, required traffic should instead be permitted through appropriately scoped firewall rules.

---

## 🏢 Active Directory & User Administration

After establishing network connectivity, I installed **Active Directory Domain Services (AD DS)** on `dc-1` and promoted the server to a domain controller for:

```text
mydomain.com
```

Using **Active Directory Users and Computers**, I organized domain resources into several organizational units:

```text
mydomain.com
│
├── _ADMINS
│     └── jane_admin
│
├── _EMPLOYEES
│     └── Employee Accounts
│
├── _CLIENTS
│     └── client-1
│
└── _GROUPS
      └── Security Groups
```

I created a dedicated administrative account, joined `client-1` to the domain, and moved the workstation into the `_CLIENTS` OU.

I also executed a **provided PowerShell provisioning script** to generate employee accounts in `_EMPLOYEES` and practiced common Active Directory support tasks:

- 👤 Managing domain users
- 🔑 Resetting passwords
- 🔓 Unlocking accounts
- 🚫 Disabling and re-enabling accounts
- 👥 Managing security group membership
- 💻 Authenticating domain users from `client-1`
- 🔎 Reviewing security events with Event Viewer

I validated domain authentication from the employee workstation using `whoami` to confirm the active domain identity.

<!--
📸 SCREENSHOT 3 — ACTIVE DIRECTORY
Place here: after the AD administration explanation.
Capture: Active Directory Users and Computers.
Best screenshot: Show mydomain.com expanded with _ADMINS, _EMPLOYEES,
_CLIENTS, and _GROUPS. Ideally show client-1 inside _CLIENTS.
Purpose: This is one of the strongest screenshots—it proves the domain,
OU structure, workstation, and overall AD organization.
-->

![Active Directory Domain Structure](docs/images/active-directory-structure.png)

> **Active Directory:** Domain resources organized into administrative, employee, client, and security group organizational units.

---

## 🔐 Group Policy & Account Security

I used **Group Policy Management** to configure and test centralized account security policies.

For the domain account lockout policy, I configured:

| Setting | Configuration |
| --- | --- |
| **Lockout Threshold** | 5 failed login attempts |
| **Lockout Duration** | 30 minutes |

I applied the updated policy to `client-1` using:

```powershell
gpupdate /force
```

To validate the policy, I intentionally exceeded the failed-login threshold with a test employee account. The account was successfully locked by the domain policy.

I then approached the issue as an administrator: located the affected account in Active Directory, unlocked it, and successfully authenticated again from `client-1`.

```text
Failed Login Attempts
        │
        ▼
  Account Locked
        │
        ▼
  Identify User in AD
        │
        ▼
   Unlock Account
        │
        ▼
  Verify User Login ✓
```

<!--
📸 SCREENSHOT 4 — GROUP POLICY
Place here: after the lockout workflow.
Capture: Group Policy Management showing the account lockout configuration.
Best screenshot: Clearly show the 5-attempt threshold and 30-minute duration.
Purpose: Proves you configured a centralized domain security policy.
-->

![Group Policy Account Lockout Settings](docs/images/account-lockout-policy.png)

> **Account security:** Domain account lockout policy configured and tested through Group Policy.

---

## 🌐 DNS & Name Resolution

With `dc-1` providing DNS services, I practiced configuring and troubleshooting **Windows DNS name resolution** from the domain workstation.

Initially, `client-1` could not resolve the hostname:

```powershell
ping mainframe
```

Using DNS Manager on `dc-1`, I created an **A record** mapping:

```text
mainframe → 10.0.1.4
```

I then retested from `client-1`, and the hostname successfully resolved to `10.0.1.4`.

During testing, I also cleared the client's DNS resolver cache with:

```powershell
ipconfig /flushdns
```

I additionally created a **CNAME alias** to practice hostname aliasing:

```text
search → www.google.com
```

and verified the alias from `client-1`.

<!--
📸 SCREENSHOT 5 — DNS RESOLUTION
Place here: after the DNS explanation.
Capture: PowerShell on client-1 showing a successful "ping mainframe".
Best screenshot: Make sure "mainframe" resolving to 10.0.1.4 is visible.
Purpose: Shows the RESULT of your DNS configuration rather than simply
showing that a record exists in DNS Manager.
-->

![DNS Name Resolution Test](docs/images/dns-mainframe-resolution.png)

> **DNS verification:** `client-1` successfully resolving the `mainframe` hostname through the DNS service running on `dc-1`.

This exercise demonstrated a basic troubleshooting workflow:

**Name fails to resolve → check DNS → configure record → retest → verify resolution.**

---

## 📁 File Shares & Access Control

I configured network shares on `dc-1` and tested access from domain users on `client-1`.

| Resource | Authorized Users / Group | Access |
| --- | --- | --- |
| `read-access` | Domain Users | Read |
| `write-access` | Domain Users | Read/Write |
| `no-access` | Domain Admins | Read/Write |
| `accounting` | ACCOUNTANTS | Read/Write |

From `client-1`, I accessed the server through:

```text
\\dc-1
```

I verified that an employee could read but not create files in `read-access`, create files in `write-access`, and was denied access to the restricted `no-access` resource.

### 👥 Group-Based Resource Access

I created an `ACCOUNTANTS` Active Directory security group and configured the `accounting` resource for members of that group.

A test employee initially could not access the resource because the account was not a member of `ACCOUNTANTS`.

```text
Employee
   │
   │ Not a Member
   ▼
ACCOUNTANTS
   │
   ✕
Accounting
ACCESS DENIED
```

After adding the employee to `ACCOUNTANTS` and signing out and back into `client-1`:

```text
Employee
   │
   ▼
ACCOUNTANTS
   │
   ▼
Accounting
ACCESS GRANTED ✓
```

This simulated a common IT support scenario where an employee cannot access a department resource because the account does not have the required group membership.

<!--
📸 SCREENSHOT 6 — FILE SHARE / ACCOUNTANTS ACCESS
Place here: after the ACCOUNTANTS scenario.
Capture: Your strongest evidence of the ACCOUNTANTS exercise.
Ideal option: client-1 successfully accessing the accounting share after
the employee was added to ACCOUNTANTS.
Alternative: ADUC showing the employee as a member of ACCOUNTANTS if that
more clearly demonstrates what you configured.
Purpose: Proves group-based access control worked from the user's perspective.
-->

![Accounting Group Resource Access](docs/images/accounting-group-access.png)

> **Access verification:** Employee access to the Accounting resource after receiving membership in the appropriate Active Directory security group.

---

## 💡 Key Takeaways

This project gave me hands-on experience administering and troubleshooting a **Windows Active Directory domain** from both the administrator and employee perspectives.

I practiced deploying Windows systems in Azure, administering Active Directory users and computers, applying Group Policy, resolving account access issues, configuring DNS, managing security groups, and controlling access to shared network resources.

Most importantly, the project connected these individual technologies into one environment and demonstrated how **identity, authentication, networking, security policies, DNS, and resource permissions work together to support users in a Windows domain**.

### 📚 Project Context

This project was completed as part of hands-on **CourseCareers IT training** and documented as a portfolio project to demonstrate the technical skills and troubleshooting workflows I practiced.

The environment is an **educational lab** designed to simulate common Windows IT administration and support responsibilities rather than a production deployment.
