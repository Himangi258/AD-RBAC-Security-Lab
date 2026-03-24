# AD-RBAC-Security-Lab
Secure enterprise network using Active Directory with centralized authentication, role-based access control (RBAC), firewall protection, and intrusion detection using Suricata IDS.

## Project Overview
This project demonstrates a real-world **enterprise security setup** using:

- **Active Directory (AD)** — centralized user and group management
- **Role-Based Access Control (RBAC)** — permissions tied to roles, not individuals
- **Folder-Level Security** — NTFS permissions enforcing least-privilege access
- **Windows Defender Firewall** — network-level traffic control

Built to simulate how IT security teams manage access in corporate environments — where the wrong person accessing the wrong folder is a compliance violation, not just an inconvenience.

---

## Architecture

```
Windows Server (Domain Controller)
│
├── Domain: company.local
│
├── Organizational Units (OUs)
│   ├── HR
│   │   └── hr_user
│   └── IT
│       └── it_user
│
├── Security Groups
│   ├── HR_Group  → access to C:\HR_Folder only
│   └── IT_Group  → access to C:\IT_Folder only
│
└── Shared Folders
    ├── C:\HR_Folder  (HR_Group: Full Control | Domain Users: None)
    └── C:\IT_Folder  (IT_Group: Full Control | Domain Users: None)
```

---

##  Setup Guide

### Step 1 — Install Active Directory Domain Services

1. Open **Server Manager** → `Add Roles and Features`
2. Select: **Active Directory Domain Services**
3. After install → Click **Promote this server to a domain controller**
4. Choose: `Add a new forest`
5. Root domain name: `company.local`
6. Set DSRM password → Complete setup → Restart

---

### Step 2 — Create Organizational Units

```
Active Directory Users and Computers
└── company.local
    ├── New OU → "HR"
    └── New OU → "IT"
```

---

### Step 3 — Create Users

| Username | OU | Password |
|---|---|---|
| `hr_user` | HR | `Hr@12345` |
| `it_user` | IT | `It@12345` |

> Right-click OU → New → User

---

### Step 4 — Create Security Groups

| Group Name | Type | Scope |
|---|---|---|
| `HR_Group` | Security | Global |
| `IT_Group` | Security | Global |

> Add `hr_user` → `HR_Group`  
> Add `it_user` → `IT_Group`

---

### Step 5 — Create Folders

```powershell
# Run in PowerShell as Administrator
New-Item -ItemType Directory -Path "C:\HR_Folder"
New-Item -ItemType Directory -Path "C:\IT_Folder"
```

---

### Step 6 — Set NTFS Permissions (RBAC)

**For `C:\HR_Folder`:**
1. Right-click → Properties → Security → Advanced
2. **Disable inheritance** → Convert
3. **Remove** `Users` group
4. **Add** `HR_Group` → Full Control
**For `C:\IT_Folder`:**
1. Same process
2. Remove `Users`
3. Add `IT_Group` → Full Control

> This is the **core RBAC implementation** — access is role-based, not user-based.

---

#Step 7 — Configure Firewall Rules

Allow AD Traffic:
A firewall rule was configured to allow essential Active Directory communication by permitting TCP traffic on ports 53, 88, 135, 139, 389, and 445. The rule allows connections for Domain and Private profiles, ensuring proper functioning of authentication, DNS, and directory services within the network.

Block HTTP Traffic:
A firewall rule was created to block TCP traffic on port 80, preventing HTTP communication. This rule applies to Domain and Private profiles and helps reduce the attack surface by restricting unnecessary or unsecured web traffic within the network.

## 📸 Screenshots

> All screenshots are in the `/screenshots` folder.

| # | Screenshot | What It Proves |
|---|---|---|
| 1 | `ad-users.png` | AD users and groups created |
| 2 | `group-members.png` | hr_user is member of HR_Group |
| 3 | `folder-structure.png` | HR_Folder and IT_Folder exist |
| 4 | `folder-permissions.png` | NTFS permissions — HR_Group only |
| 5 | `access-denied.png` | hr_user blocked from IT_Folder ✅ |
| 6 | `hr-access.png` | hr_user can open HR_Folder ✅ |
| 7 | `firewall-rules.png` | Allow_AD_Traffic + Block_HTTP rules |
| 8 | `login.png` | Domain login as company\hr_user |

## Key Result
When `hr_user` attempts to open `C:\IT_Folder`:
```
You don't have permission to access this folder.
```
When `hr_user` opens `C:\HR_Folder`:
```
Access granted — folder opens successfully.
```

#Technologies Used
![Windows Server](https://img.shields.io/badge/Windows_Server-2019%2F2022-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active_Directory-Domain_Services-0078D6?style=for-the-badge&logo=microsoft&logoColor=white)
![Suricata IDS](https://img.shields.io/badge/Suricata-IDS-EF3B2D?style=for-the-badge)
![VirtualBox](https://img.shields.io/badge/VirtualBox-Virtualization-183A61style=forthebadge&logo=virtualbox&logoColor=white)
