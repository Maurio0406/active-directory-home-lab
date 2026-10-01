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
- [ ] Sign in interactively as HR user `mjohnson` and verify the HR Group Policy with `gpresult`
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

Paused here:
The next step is to sign in to `LAB-W11-CL02` as `mjohnson@corp.lab`, confirm the session with `whoami`, run `gpupdate /force`, and verify `HR - Screen Lock Policy` with `gpresult`. Work paused due to network/connectivity issues in the current session.

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

## Current Status
The `corp.lab` Active Directory domain is operational on `LAB-DC01` (`10.10.1.4`). Azure Windows 11 client `LAB-W11-CL02` (`10.10.1.5`) uses the domain controller for DNS and has successfully joined `corp.lab`. DNS SRV discovery and the computer secure channel have been verified. The HR test account `mjohnson` also successfully authenticated after troubleshooting a required-password-change condition.

Work is paused due to network issues. The next step is to sign in interactively to `LAB-W11-CL02` as `mjohnson@corp.lab`, confirm `corp\\mjohnson` with `whoami`, run `gpupdate /force`, and verify that `HR - Screen Lock Policy` appears in `gpresult`. After Group Policy verification, the next planned administration tasks are shared folders and NTFS/share permissions. The original local VMware client `LAB-W11-CL01` remains available as a future networking extension.