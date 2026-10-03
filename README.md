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
- [x] Create Azure Windows 11 Pro client `LAB-W11-CL02` on the same VNet/subnet as the domain controller
- [x] Configure `LAB-W11-CL02` to use `LAB-DC01` (`10.10.1.4`) for DNS
- [x] Verify secure client-to-domain-controller connectivity inside the Azure VNet
- [ ] Connect the original local VMware client `LAB-W11-CL01` to the Azure lab (future networking project)

### Phase 2 - Active Directory Setup
- [x] Configure server `LAB-DC01`
- [x] Install Active Directory Domain Services
- [x] Promote the server to a domain controller
- [x] Create the `corp.lab` forest/domain
- [x] Install/configure DNS with the domain controller promotion
- [x] Create organizational units
- [x] Create test users and security groups
- [x] Join Azure client `LAB-W11-CL02` to the `corp.lab` domain
- [ ] Join the original local VMware client `LAB-W11-CL01` to the domain (future extension)

### Phase 3 - Administration
- [x] Reset a user password
- [x] Review account lockout controls
- [x] Create and configure a Group Policy Object
- [x] Link a GPO to a department OU
- [x] Verify the domain-joined client has a healthy secure channel to `corp.lab`
- [x] Sign in interactively as HR user `mjohnson` and verify the HR Group Policy with `gpresult`
- [x] Create the HR departmental shared folder
- [x] Configure SMB share permissions and NTFS permissions
- [x] Test authorized HR access and denied Sales access

### Phase 4 - Troubleshooting
- [x] Create a controlled permissions problem in the lab
- [x] Diagnose the problem
- [x] Document the cause
- [x] Document the solution
- [x] Record what I learned

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


### Session 3 - Azure Domain Client, DNS, and Domain Join
Created a second Windows 11 client inside Azure so the first domain join could be completed securely without exposing Active Directory services to the public internet.

Completed:
- Created Windows 11 Pro client `LAB-W11-CL02` in Azure
- Placed the client on `vnet-northcentralus-1` and subnet `snet-servers` (`10.10.1.0/24`)
- Client received private IP `10.10.1.5`
- Configured the client NIC to use `LAB-DC01` at `10.10.1.4` as its DNS server
- Verified `corp.lab` DNS resolution
- Queried the Active Directory LDAP SRV record with `nslookup -type=SRV _ldap._tcp.dc._msdcs.corp.lab`
- Confirmed the SRV record points to `lab-dc01.corp.lab` at `10.10.1.4`, LDAP port 389
- Joined `LAB-W11-CL02` to the `corp.lab` domain
- Verified the computer/domain trust relationship with `Test-ComputerSecureChannel -Verbose`; result was `True`
- Queried domain accounts with `net user /domain`
- Confirmed HR test user Maya Johnson uses username `mjohnson`
- Tested Maya's credentials with `runas /user:CORP\\mjohnson cmd`
- Diagnosed error 1907: the user's password had to be changed before sign-in
- Reset Maya's password in Active Directory and cleared the required-password-change condition for this lab test
- Retested `runas` successfully

Architecture note:
The original VMware client `LAB-W11-CL01` remains on the local VMware NAT network and is not yet joined to the domain. `LAB-W11-CL02` was intentionally created inside Azure on the same private VNet as `LAB-DC01` to complete the domain-join and Group Policy portions of the lab without opening AD, DNS, LDAP, Kerberos, SMB, or RDP ports to the internet.

Session 3 ended before the interactive HR user and Group Policy verification. Those tasks were completed in Session 4.


### Session 4 - Group Policy Verification, SMB Sharing, and Permissions Troubleshooting
Completed the HR Group Policy verification and built the first departmental file share.

Completed:
- Connected to `LAB-W11-CL02` through Azure Bastion as HR user Maya Johnson (`mjohnson`)
- Added `CORP\mjohnson` to the local `Remote Desktop Users` group after isolating a Bastion/RDP sign-in problem
- Verified Maya's identity with `whoami` as `corp\mjohnson`
- Ran `gpupdate /force`
- Used `gpresult /r` to confirm `HR - Screen Lock Policy` applied to Maya
- Created `C:\Shares\HR` on `LAB-DC01`
- Created the `HR-Staff` global security group and added Maya
- Published the folder as the SMB share `\\LAB-DC01\HR`
- Granted `HR-Staff` Change permission at the SMB share layer
- Configured NTFS permissions so `HR-Staff` has Modify while SYSTEM and Administrators retain Full Control
- Removed broad inherited access from Authenticated Users and BUILTIN Users
- Verified Maya could read, modify, and create files in the HR share
- Used Sales user Taylor Smith (`tsmith`) as a negative access test
- Verified Taylor received Access Denied when attempting to access the HR share
- Intentionally removed the `HR-Staff` NTFS permission to create a controlled outage
- Troubleshot the outage by verifying Maya's identity, group membership, SMB share permission, and NTFS ACL
- Identified the missing NTFS permission as the root cause
- Restored `HR-Staff` Modify permission and confirmed Maya regained access

Permission model tested:
- SMB: `CORP\HR-Staff` = Change
- NTFS: `CORP\HR-Staff` = Modify
- NTFS: SYSTEM = Full Control
- NTFS: Administrators = Full Control

Key commands used:
```powershell
gpupdate /force
gpresult /r
whoami
whoami /groups
Get-SmbShareAccess -Name HR
icacls "C:\Shares\HR"
dir "\\LAB-DC01\HR"
```

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

### Issue 4 - HR domain user could not sign in remotely
**Problem:** Attempting to use the HR test account `mjohnson` for the domain-client session failed.

**Checks performed:** Verified the computer trust relationship with `Test-ComputerSecureChannel -Verbose`. The result was `True`, confirming that `LAB-W11-CL02` still had a healthy secure channel with `corp.lab`.

**Troubleshooting:** Ran `runas /user:CORP\\mjohnson cmd` from the domain-joined client.

**Root cause:** Windows returned error 1907 indicating that the user's password had to be changed before signing in.

**Solution:** Reset Maya Johnson's password in Active Directory and removed the immediate password-change requirement for the lab test.

**Result:** The `runas` authentication test succeeded.

**What I learned:** A remote domain-login failure does not automatically mean the domain join or network is broken. Testing the secure channel and then testing the user account separately can isolate computer-trust problems from account/password problems.


### Issue 5 - Domain user authenticated but Azure Bastion session failed
**Problem:** Maya's domain credentials worked with `runas`, but Azure Bastion did not establish an interactive RDP session.

**Troubleshooting:** Verified the domain account separately, checked active RDP sessions with `qwinsta`, and inspected the local `Remote Desktop Users` group.

**Root cause:** The domain user did not have the required local Remote Desktop Users membership for this lab's remote-access configuration.

**Solution:** Added `CORP\mjohnson` to the local `Remote Desktop Users` group on `LAB-W11-CL02`.

**Result:** Maya successfully connected through Bastion and received a full interactive domain session.

**What I learned:** Successful domain authentication and permission to create an interactive remote desktop session are separate checks.

### Issue 6 - Password-change requirement blocked Bastion sign-in
**Problem:** Taylor Smith (`tsmith`) received a Bastion connection error even after being prepared for remote access.

**Root cause:** Taylor's account required a password change at next sign-in, the same condition previously encountered with Maya.

**Solution:** Reset Taylor's lab password and removed the required-password-change condition for the test.

**Result:** Taylor successfully signed in through Bastion as `tsmith@corp.lab`.

**What I learned:** Account state can cause a remote sign-in to fail even when domain connectivity and RDP configuration are working.

### Issue 7 - HR user suddenly lost access to departmental share
**Problem:** Maya previously had working access to `\\LAB-DC01\HR` and then received Access Denied.

**Troubleshooting:** Verified `corp\mjohnson` with `whoami`, confirmed `CORP\HR-Staff` in `whoami /groups`, confirmed the SMB share still granted HR-Staff Change access, then inspected the folder ACL with `icacls`.

**Root cause:** The `CORP\HR-Staff` NTFS Modify permission had been intentionally removed.

**Solution:** Restored the NTFS permission:
```powershell
icacls "C:\Shares\HR" /grant "CORP\HR-Staff:(OI)(CI)M"
```

**Result:** Maya immediately regained access to the HR share.

**What I learned:** Windows network-file access depends on both SMB share permissions and NTFS permissions. Checking identity, group membership, share permissions, and NTFS ACLs in order helps isolate the failing layer.

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
- How Active Directory clients locate domain controllers through DNS SRV records
- How to verify a Windows computer's domain trust with `Test-ComputerSecureChannel`
- How to distinguish a domain connectivity problem from a user-account authentication problem
- Why keeping AD traffic on a private Azure VNet is safer than exposing domain-service ports to the internet
- How to verify user-applied Group Policy with `gpupdate` and `gpresult`
- How SMB share permissions and NTFS permissions work together
- Why AD security groups are preferable to assigning departmental permissions directly to individual users
- How to validate authorized access with an HR user and denied access with a non-HR user
- How to troubleshoot an Access Denied problem by checking identity, group membership, SMB permissions, and NTFS ACLs

## Current Status
The `corp.lab` Active Directory domain is operational on `LAB-DC01` (`10.10.1.4`). Azure Windows 11 client `LAB-W11-CL02` (`10.10.1.5`) is joined to the domain and uses the domain controller for DNS. Maya Johnson successfully signed in with a full domain session, and `gpresult /r` confirmed that `HR - Screen Lock Policy` applies to her HR account.

The first departmental file share is also operational. `\\LAB-DC01\HR` uses the `HR-Staff` AD security group, SMB Change permission, and NTFS Modify permission. Maya successfully read, modified, and created files, while Sales user Taylor Smith received Access Denied. A controlled NTFS permissions failure was created, diagnosed, repaired, and retested successfully.

Next session: build the IT, Finance, and Sales departmental shares, assign group-based permissions, test cross-department access, and then configure automatic network-drive mapping through Group Policy. The original VMware client `LAB-W11-CL01` remains a future networking extension.
