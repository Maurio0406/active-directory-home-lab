# Active Directory Home Lab

## Project Overview
This project documents my process of building a beginner Windows Active Directory home lab. I am using the lab to develop practical IT support, Windows administration, networking, and troubleshooting skills.

This is a learning project, so I will update this repository as I build the environment, encounter problems, and learn new concepts.

## Goals
- Build a virtual Windows domain environment
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
- PowerShell
- Active Directory Domain Services (planned)
- DNS (planned)
- Group Policy (planned)
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
- [ ] Create domain controller environment
- [ ] Configure domain lab networking

### Phase 2 - Active Directory Setup
- [ ] Configure the domain controller
- [ ] Install Active Directory Domain Services
- [ ] Promote the server to a domain controller
- [ ] Create a test domain
- [ ] Create organizational units
- [ ] Create test users and groups
- [ ] Join `LAB-W11-CL01` to the domain

### Phase 3 - Administration
- [ ] Reset user passwords
- [ ] Practice locked-account troubleshooting
- [ ] Configure Group Policy
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

## What I Learned
- How a hypervisor is used to create a virtual workstation
- Basic VM resource allocation for CPU, memory, storage, and networking
- The difference between a VMware VM name and the Windows hostname
- How to use `hostname` and `ipconfig` for basic Windows/network verification
- How NAT allows the VM to communicate outside its virtual network through the host
- Why administrative PowerShell commands may require elevation
- Why consistent workstation naming matters in an IT environment

## Current Status
The Windows 11 client workstation `LAB-W11-CL01` is installed and operational. The next major step is creating the domain controller environment and preparing the client to communicate with the Active Directory domain.