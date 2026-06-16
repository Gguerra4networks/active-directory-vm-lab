####🎬 Watch Me Build This Lab!


https://www.loom.com/share/eef9c22fc42243b48a53e82772c788db



[README.md](https://github.com/user-attachments/files/28728955/README.md)
# active-directory-vm-lab
A hands-on lab demonstrating the setup and configuration of Active Directory on a Windows Server VM
# Lab 01 — Active Directory
### Windows Server 2025 · Azure Free Account · Identity & Access Management

![Platform](https://img.shields.io/badge/Platform-Windows%20Server%202025-0078D4?style=flat-square&logo=windows)
![Cost](https://img.shields.io/badge/Cost-%240%20Free%20Tier-brightgreen?style=flat-square)
![Duration](https://img.shields.io/badge/Duration-3–5%20Hours-blue?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner–Intermediate-yellow?style=flat-square)

**Certification alignment:** CompTIA Network+ · CompTIA Security+ · Azure Administrator (AZ-104)

**Career relevance:** IT Support · Help Desk · Sysadmin · Cloud Engineer · Security Analyst

---

## About This Lab

Active Directory is the identity backbone of every Windows enterprise environment. In this lab you will build one from scratch — standing up a domain controller, structuring departments as Organizational Units, creating users and security groups, and enforcing security policies across every machine in the domain.

This is the exact environment IT teams manage every day, and the foundational knowledge required for nearly every IT and cloud role. Hybrid environments sync on-premises AD identities to Microsoft Entra ID (formerly Azure AD), meaning everything you build here maps directly to cloud roles.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Azure VM — Windows Server 2025               │
│                                                                 │
│   ┌──────────────────────────┐       ┌──────────────────────┐  │
│   │    Domain Controller     │──────▶│    Group Policy      │  │
│   │  lab.local · DNS · AD DS │       │    GPOs enforced     │  │
│   └────────────┬─────────────┘       └──────────────────────┘  │
│                │                                                │
│   ┌────────────▼─────────────────────────────────────────────┐ │
│   │              Organisational Units (OUs)                  │ │
│   │                                                          │ │
│   │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │ │
│   │  │    IT    │ │ Finance  │ │    HR    │ │  Sales   │   │ │
│   │  │alice.chen│ │bob.patel │ │c.jones   │ │d.smith   │   │ │
│   │  │IT_Admins │ │Fin_Users │ │HR_Users  │ │Sal_Users │   │ │
│   │  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │ │
│   └──────────────────────────────────────────────────────────┘ │
│                         │                                       │
│              ┌──────────▼──────────┐                           │
│              │  Domain-Joined      │                           │
│              │  Workstation        │                           │
│              │  Policies applied   │                           │
│              │  RDP access         │                           │
│              └─────────────────────┘                           │
└─────────────────────────────────────────────────────────────────┘

  Identity layer       Department OUs       Policy enforcement
```

---

## What You Will Be Able to Do

| Skill | Real-World Application |
|---|---|
| Promote a server to Domain Controller | Create and own an AD forest — the foundation of every Windows network |
| Build OU structure and security groups | Organise users by department and control access through group membership |
| Deploy Group Policy Objects (GPOs) | Enforce password policies, screen lock timers, and USB restrictions centrally |
| Perform common help desk tasks | Reset passwords, unlock accounts, disable leavers, audit group memberships |
| Join machines to the domain | Connect a workstation so it becomes a managed, policy-enforced asset |

---

## Prerequisites

| Requirement | Details |
|---|---|
| Azure free account | [azure.microsoft.com/free](https://azure.microsoft.com/free) — $200 credit, no charge within free tier |
| OR local machine | 8GB RAM minimum, 60GB free disk, quad-core CPU with virtualisation enabled |
| RDP client | Remote Desktop app (Windows built-in, or Microsoft Remote Desktop on macOS) |
| Estimated cost | **$0** — covered by Azure free tier credits |

---

## Lab Steps

### Step 1 — Provision the VM

**Option A — Azure (Recommended)**

No local hardware requirements. Runs in Microsoft's data centre; you connect via RDP.

1. Create a free Azure account at [azure.microsoft.com/free](https://azure.microsoft.com/free)
2. In the Azure portal, search **Virtual machines** → **Create**
3. Use the following settings:

| Setting | Value |
|---|---|
| Region | East US |
| Image | Windows Server 2025 Datacenter — Gen2 |
| Size | Standard_B2s (2 vCPU, 4GB RAM) |
| Authentication | Password |
| Public inbound ports | Allow RDP (3389) |
| OS disk | Standard SSD |

> **Cost tip:** Stop the VM (do not delete) when not in use. A B2s costs ~$0.05/hr. Stopping pauses compute billing so your $200 credit lasts.

**Fix clipboard before connecting:**
- Open the Remote Desktop app → enter the VM's public IP → **Show Options** → **Local Resources** tab → confirm **Clipboard** is checked → Connect.
- If using the Azure portal browser console, download the RDP file instead (**Connect → Download RDP File**) and open it with the native app for full clipboard support.

**Option B — VirtualBox (Local)**

1. Download [VirtualBox](https://virtualbox.org) — free, no account required
2. Download the [Windows Server 2025 Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/) from Microsoft
3. Create a VM: 4GB RAM, 60GB disk, type = Windows Server 2019/2022
4. Mount the ISO, boot, and select **Windows Server 2025 Datacenter with Desktop Experience**

---

### Step 2 — Install Active Directory Domain Services

RDP into the VM. Open **Server Manager** (launches automatically on login).

**Via GUI:** Manage → Add Roles and Features → Server Roles → check **Active Directory Domain Services** → Add Features → Install

**Via PowerShell:**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

Install Group Policy Management Console immediately after — you will need it in Step 5:
```powershell
Install-WindowsFeature -Name GPMC
```

---

### Step 3 — Promote the Server to Domain Controller

Installing AD DS does not create a domain. Promotion is the step that creates the forest, the domain, and makes this server the authoritative DNS and identity source.

**Via GUI:**
1. Click the yellow warning flag in Server Manager → **Promote this server to a domain controller**
2. Select **Add a new forest**
3. Set Root domain name: `lab.local`
4. Set a DSRM password (write it down — needed for disaster recovery only)
5. Accept DNS and NetBIOS defaults → **Install** — the server restarts automatically

**Via PowerShell:**
```powershell
Import-Module ADDSDeployment
Install-ADDSForest `
  -DomainName 'lab.local' `
  -DomainNetBiosName 'LAB' `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourDSRMPassword!' -AsPlainText -Force) `
  -Force:$true
```

---

### Step 4 — Build the Organisational Structure

Open **Active Directory Users and Computers (ADUC)** from the Tools menu in Server Manager.

**Create Organisational Units:**
```powershell
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```

**Create Security Groups:**
```powershell
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

**Create Users and Assign Group Memberships:**

> **Important:** Run the entire block below at once, not line by line. The `$password` variable must be defined before the `New-ADUser` commands execute.

```powershell
# Step 1 — define the password variable first
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

# Step 2 — create all 4 users
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

# Step 3 — assign users to their department groups
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

---

### Step 5 — Configure Group Policy

Open **Group Policy Management** from the Tools menu in Server Manager.

1. Expand **Forest: lab.local → Domains → lab.local**
2. Right-click the **IT OU** → **Create a GPO in this domain and link it here**
3. Name it: `IT Security Policy`
4. Right-click the new GPO → **Edit** and configure the settings below:

| Policy Path | Setting | Value |
|---|---|---|
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Minimum password length | 12 |
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Password must meet complexity requirements | Enabled |
| Computer Config → Windows Settings → Security → Local Policies → Security Options | Interactive logon: Machine inactivity limit | 900 seconds |
| Computer Config → Administrative Templates → System → Removable Storage Access | All removable storage classes: Deny all access | Enabled |

**Test your GPO:** Join a second VM to `lab.local`, move its computer account into the IT OU, run `gpupdate /force`, log in as `alice.chen`, and verify the screen lock policy applies.

---

### Step 6 — Common Help Desk Tasks

**Reset a password:**
```powershell
Set-ADAccountPassword -Identity "bob.patel" -Reset `
  -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```

**Unlock a locked account:**
```powershell
Unlock-ADAccount -Identity "carol.jones"
```

**Disable an account (offboarding):**
```powershell
# Disable preserves history and group memberships for audit purposes
Disable-ADAccount -Identity "david.smith"

# Find all currently disabled accounts
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

**Audit inactive accounts:**
```powershell
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
  -Properties LastLogonDate | Select-Object Name, LastLogonDate
```

**Check group membership:**
```powershell
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

---

## Verification Checklist

Run these commands after completing the lab to confirm everything is configured correctly:

| Check | Command | Expected Result |
|---|---|---|
| Domain controller is running | `Get-ADDomainController` | Returns DC info including forest `lab.local` |
| OUs exist | `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs |
| Users exist and are enabled | `Get-ADUser -Filter {Enabled -eq $true}` | Lists your 4 test accounts |
| Group memberships correct | `Get-ADGroupMember -Identity IT_Admins` | Returns `alice.chen` |
| GPO is linked | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows `IT Security Policy` as linked |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| PowerShell prompts for `Name:` when creating users | You ran `New-ADUser` before defining `$password`. Run the entire script block at once. |
| Cannot copy/paste into the VM | Open the RDP client → Show Options → Local Resources → check Clipboard → reconnect. Or download the RDP file from the Azure portal and open with the native Remote Desktop app. |
| Promotion fails: DNS conflict | Set the NIC's preferred DNS to `127.0.0.1` before promoting, or use the VM's static IP |
| Cannot RDP after domain join | Log in as `LAB\Administrator` (domain format), not just `Administrator` |
| GPO not applying | Run `gpupdate /force` on the target machine, then `gpresult /r` to see applied policies |
| User cannot log in after creation | Confirm the account is Enabled and `ChangePasswordAtLogon` is not blocking login |
| AD Users and Computers not showing | Run `dsa.msc` from the Run dialog, or run `Add-WindowsFeature RSAT-ADDS` |

---

## Key Concepts Reference

**Domain Controller** — The server that runs Active Directory. All authentication requests in the domain flow through it. Every machine that joins `lab.local` trusts this server to verify identities.

**Forest and Domain** — A Forest is the top-level container for your entire AD structure. A Domain is a management boundary inside the forest — ours is `lab.local`. Most organisations have one domain per forest.

**Organisational Unit (OU)** — A folder inside Active Directory for organising users, computers, and groups. OUs are where you link Group Policy — every object inside an OU inherits whatever GPO is attached to it.

**Security Group** — A container that holds user accounts. Access to resources is granted to groups, not individual users. This is role-based access control: add a user to a group and they inherit all access. Remove them and all access is revoked simultaneously.

**Group Policy Object (GPO)** — A collection of settings applied automatically to every user or computer in an OU. Password complexity rules, screen lock timers, USB restrictions — all controlled centrally without touching each machine.

---

## Notes

- Document everything you build. Record the commands you ran and what each step accomplished. This becomes a portfolio entry you can reference in interviews.
- When asked about Active Directory experience — this lab is your answer.
- AD is the most targeted system in ransomware attacks. Understanding how it works is the foundation for defending it.

---

*Lab environment: Windows Server 2025 Evaluation (180-day licence) · Azure Free Account*
