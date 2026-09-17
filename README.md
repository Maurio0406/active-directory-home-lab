# Active Directory Home Lab

## Project Overview
This project documents my process of building a beginner Windows Active Directory lab. I am using the lab to develop practical IT support, Windows administration, networking, identity management, Group Policy, and troubleshooting skills.

This is a learning project, so I will update this repository as I build the environment, encounter problems, and learn new concepts.

## Goals
- Build a Windows domain environment
- Learn the purpose of a domain controller
- Install and configure Active Directory Domain Services
- Create and manage users, groups, and organizational units
- Join a Windows client to a domain
- Practice password resets and account lockout troubleshooting
- Configure Group Policy
- Configure shared folders and permissions
- Learn how DNS relates to Active Directory
- Practice documenting and troubleshooting technical problems

## Technologies and Tools
- Apple Silicon Mac host
- VMware Fusion
- Windows 11 Pro ARM client
- Microsoft Azure Windows Server environment
- Windows Server 2025
- Active Directory Domain Services (AD DS)
- DNS
- Group Policy Management
- PowerShell
- GitHub for documentation

## Lab Progress

### Phase 1 - Environment Setup
- [x] Choose and install virtualization software (VMware Fusion)
- [x] Create Windows 11 Pro ARM client virtual machine
- [x] Configure VM resources (2 CPU cores, 6 GB RAM, 64 GB virtual disk)
- [x] Configure NAT networking
- [x] Create local LabAdmin account
- [x] Verify network configuration with `ipconfig`
- [x] Rename Windows client to `LAB-W11-CL01`
- [x] Create Windows Server domain controller environment in Microsoft Azure
- [ ] Configure connectivity between the Windows 11 client and domain controller

### Phase 2 - Active Directory Setup
- [x] Configure server `LAB-DC01`
- [x] Install Active Directory Domain Services
- [x] Promote the server to a domain controller
- [x] Create the `corp.lab` forest/domain
- [x] Install/configure DNS with the domain controller promotion
- [x] Create organizational units
- [x] Create test users and security groups
- [ ] Join `LAB-W11-CL01` to the domain

### Phase 3 - Administration
- [x] Reset a user password
- [x] Review account lockout controls
- [x] Create and configure a Group Policy Object
- [x] Link a GPO to a department OU
- [ ] Verify Group Policy from a domain-joined client
- [ ] Create shared folders
- [ ] Configure file and folder permissions

### Phase 4 - Troubleshooting
- [ ] Create a controlled problem in the lab
- [ ] Diagnose the problem
- [ ] Document the cause
- [ ] Document the solution
- [ ] Record what I learned

## Session Log

### Session 1 - Windows Client Setup
Built the first workstation for the lab.

Completed:
- Installed and configured VMware Fusion
- Created a Windows 11 Pro ARM virtual machine
- Configured 2 CPU cores, 6 GB RAM, and a 64 GB virtual disk
- Used VMware NAT networking
- Installed Windows 11
- Created a local administrator account named `LabAdmin`
- Used PowerShell to check the hostname and network configuration
- Verified IPv4 connectivity on the VMware virtual network
- Renamed the workstation to `LAB-W11-CL01`

Network information observed during setup:
- IPv4 address: `172.16.222.128` (DHCP-assigned during this session; may change)
- Subnet mask: `255.255.255.0` (`/24`)
- Default gateway: `172.16.222.2`
- Initial DNS suffix: `localdomain`

### Session 2 - Domain Controller and Active Directory Administration
Built the server-side Active Directory environment and practiced common identity-management tasks.

Completed:
- Configured Windows Server 2025 server `LAB-DC01`
- Installed the Active Directory Domain Services role
- Promoted `LAB-DC01` as the first domain controller in a new forest
- Created the domain `corp.lab` with NetBIOS name `CORP`
- Verified the domain in Active Directory Users and Computers
- Created top-level OUs `CORP-Users` and `CORP-Computers`
- Created department OUs for IT, HR, Finance, and Sales
- Created test users in department OUs
- Practiced resetting a domain user's password and requiring a password change at next logon
- Created the `IT-Staff` security group and added an IT user to it
- Opened Group Policy Management and created `HR - Screen Lock Policy`
- Linked the GPO to the HR OU
- Enabled the screen saver, password protection, and a 300-second screen saver timeout
- Verified that the HR GPO link is enabled

Example lab users created:
- Jordan Davis - IT
- Maya Johnson - HR
- Alex Carter - Finance
- Taylor Smith - Sales

## Troubleshooting Log

### Issue 1 - VM attempted network boot
**Problem:** The new VM reached an EFI network boot timeout instead of starting Windows Setup.

**Troubleshooting:** Restarted the VM and booted from the attached Windows installation media.

**Result:** Windows 11 Setup started successfully.

**What I learned:** A VM can fall through to PXE/network boot when it does not boot from installation media or a bootable virtual disk.

### Issue 2 - Computer name exceeded NetBIOS limit
**Problem:** The planned hostname `LAB-WIN11-CLIENT01` triggered a warning because its NetBIOS representation exceeded the 15-character limit.

**Solution:** Chose the shorter hostname `LAB-W11-CL01`.

**What I learned:** Computer naming conventions should account for compatibility constraints such as the traditional 15-character NetBIOS computer-name limit.

### Issue 3 - Rename-Computer returned Access Denied
**Problem:** `Rename-Computer` failed with an Access Denied error.

**Root cause:** PowerShell was not running with elevated administrator privileges.

**Solution:** Opened Windows PowerShell using **Run as administrator** and ran the rename command again.

**Result:** The workstation successfully restarted with the hostname `LAB-W11-CL01`.

**What I learned:** Membership in the local Administrators group does not mean every process automatically runs elevated. Administrative actions may require an elevated PowerShell session through UAC.

## Screenshots
Screenshots of the environment and important configurations will be added as the project progresses. Sensitive information such as passwords, keys, or personal information will not be uploaded.

Useful screenshots from Session 2 include:
- `corp.lab` visible in Active Directory Users and Computers
- Department OU structure under `CORP-Users`
- Test users inside their department OUs
- `IT-Staff` security group membership
- `HR - Screen Lock Policy` settings
- HR GPO link showing `Link Enabled: Yes`

## What I Learned
- How a hypervisor is used to create a virtual workstation
- Basic VM resource allocation for CPU, memory, storage, and networking
- The difference between a VMware VM name and the Windows hostname
- How to use `hostname` and `ipconfig` for basic Windows/network verification
- How NAT allows the VM to communicate outside its virtual network through the host
- Why administrative PowerShell commands may require elevation
- Why consistent workstation naming matters in an IT environment
- The role of a domain controller in centralized identity management
- How AD DS organizes users and computers through domains and OUs
- The difference between an OU and a security group
- How administrators reset domain passwords and manage account options
- How Group Policy can apply security settings to users in a specific OU
- Why DNS is an important part of Active Directory domain operations

## Current Status
The `corp.lab` Active Directory domain is operational on `LAB-DC01`. The domain contains department OUs, test users, an IT security group, and an HR screen-lock Group Policy. The Windows 11 client `LAB-W11-CL01` is also operational. The next major step is establishing secure connectivity between the client and the domain controller, joining the client to `corp.lab`, signing in with a domain user, and verifying Group Policy application.