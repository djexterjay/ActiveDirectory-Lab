Active Directory Lab
Detailed steps for configuring an Active Directory lab on Azure Windows Server (mylab.local).
Configuration Steps
1. Set Up Azure VM

Logged into Azure Portal (portal.azure.com).  
Clicked “Create a resource” > “Virtual machine.”  
Configured VM:  
Subscription: Selected my subscription.  
Resource group: Created new (mylab-rg).  
VM name: WIN-XXXX (noted for reference).  
Region: Chose closest (e.g., East US).  
Image: Windows Server 2019 Datacenter.  
Size: B2s (2 vCPUs, 4 GiB RAM).  
Username: azureuser, Password: Strong (e.g., P@ssw0rd!2025).


Networking:  
Public IP: Enabled (noted IP address).  
Ports: Allowed RDP (3389).


Clicked “Review + create” > “Create,” waited 5-10 minutes.  
In Portal, went to VM > “Connect” > “RDP,” downloaded RDP file.  
Opened RDP file, entered mylab\azureuser and password, connected.

2. Install Active Directory

In Server Manager (opened automatically), clicked “Manage” > “Add Roles and Features.”  
Clicked “Next” > selected “Role-based or feature-based installation” > “Next.”  
Selected my server (WIN-XXXX.mylab.local) > “Next.”  
Checked “Active Directory Domain Services” > “Add Features” > “Next” > “Install.”  
Waited 5-10 minutes, clicked “Close.”  
In Server Manager, clicked yellow flag > “Promote this server to a domain controller.”  
Chose “Add a new forest,” entered mylab.local as Root domain name > “Next.”  
Set Forest/Domain functional level to Windows Server 2016.  
Unchecked “Domain Name System (DNS) server” (already installed).  
Set Directory Services Restore Mode password: Restore!2025 > “Next” > “Next” through defaults.  
Clicked “Install,” waited 10-15 minutes, server restarted.  
Logged in via RDP as mylab\azureuser.

3. Create User and Password Policy

Opened Active Directory Users and Computers (Server Manager > Tools > ADUC).  
Expanded mylab.local, right-clicked > “New” > “User.”  
Entered:  
Full name: Jane Smith  
User logon name: jane  
Clicked “Next,” set Password: P@ssw0rd123, unchecked “User must change password.”  
Clicked “Finish.”

Added screenshot: https://github.com/djexterjay/ActiveDirectory-Lab/blob/57c8a2bff73628b196ffb8be88f56d1874117f8f/image%20(2).jpeg


Opened Computer Management (Start > type “compmgmt.msc”).  
Went to Local Users and Groups > Groups > “Remote Desktop Users” > “Add” > typed jane > “Check Names” > “OK.”  
Opened Group Policy Management (Server Manager > Tools).  
Right-clicked mylab.local > “Create a GPO in this domain, and Link it here” > named Default Password Policy.  
Right-clicked Default Password Policy > “Edit.”  
Navigated: Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy.  
Set “Minimum password length” to 10 characters > “OK.”  
Closed editor, selected Default Password Policy, ensured “Authenticated Users” in Security Filtering.  
Opened Command Prompt (admin): Start > “cmd” > right-click > “Run as administrator.”  
Ran gpupdate /force, waited for “Computer policy” and “User policy” updates.

Added screenshot: https://github.com/djexterjay/ActiveDirectory-Lab/blob/8da970bb3f77f428326895bf9d2b5b7e69d057ea/Screenshot%202025-04-17%20155523.png


FIXED RDP ISSUE:  
Opened Local Security Policy (Start > “secpol.msc”).  
Went to Local Policies > User Rights Assignment > “Allow log on through Remote Desktop Services.”  
Clicked “Add User or Group” > typed Remote Desktop Users > “Check Names” > “OK” > “Apply.”

Added screenshot: https://github.com/djexterjay/ActiveDirectory-Lab/blob/4596ca485025da7022e219e76df366d043562b6a/image%20(1).jpeg


Tested: RDP as mylab\jane, confirmed login and password policy.

4. Create OU and GPO

In ADUC, right-clicked mylab.local > “New” > “Organizational Unit” > named Staff > “OK.”  
Right-clicked jane > “Move” > selected Staff > “OK.”  
In Group Policy Management, right-clicked Staff OU > “Create a GPO” > named Staff Password Policy.  
Right-clicked Staff Password Policy > “Edit.”  
Navigated: Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy.  
Set “Minimum password length” to 10 characters > “OK.”  
Closed editor, selected Staff Password Policy, ensured “Authenticated Users” in Security Filtering.  
Ran gpupdate /force in Command Prompt (admin).  
Tested: Logged in as mylab\jane, confirmed policy applied.

Added screenshot: https://github.com/djexterjay/ActiveDirectory-Lab/blob/7ec4dede35091bd26ff00969b7a8f3d0713bffbf/Screenshot%202025-04-17%20155146.png

5. Create Group, Share, and GPO

In ADUC, right-clicked Staff OU > “New” > “Group.”  
Named StaffTeam, set Scope: Global, Type: Security > “OK.”  
Double-clicked StaffTeam > “Members” tab > “Add” > typed jane > “Check Names” > “OK” > “Apply.”  
In File Explorer, right-clicked C: > “New” > “Folder” > named StaffFiles.  
Right-clicked StaffFiles > “Properties” > “Sharing” tab > “Share.”  
Typed StaffTeam > “Check Names” > “Add” > set Permission Level to “Read/Write.”  
Removed “Everyone” > “Share” > copied path (\\localhost\StaffFiles).  
In Group Policy Management, right-clicked Staff OU > “Create a GPO” > named Staff Desktop Policy.  
Right-clicked Staff Desktop Policy > “Edit.”  
Navigated: User Configuration > Policies > Administrative Templates > Desktop.  
Double-clicked “Hide and disable all items on the desktop” > set to “Enabled” > “OK.”  
Closed editor, selected Staff Desktop Policy, ensured “Authenticated Users” in Security Filtering.

Add screenshot: https://github.com/djexterjay/ActiveDirectory-Lab/blob/24d455bc0001a0411ec8019f609a79f93d298bb8/image%20(3).jpeg

FIXED GPO FILTERING:  
Issue: gpresult /r as jane showed “filtered out.”  
Confirmed Staff Desktop Policy linked to Staff OU.  
Reopened editor, verified “Hide and disable all items” was Enabled.  
In Group Policy Management, selected Staff Desktop Policy > “Security Filtering” > “Add” > typed Authenticated Users > “Check Names” > “OK.”  
Ran gpupdate /force.


Tested: RDP as mylab\jane, confirmed empty desktop, accessed \\localhost\StaffFiles, created file.

6. Add User

In ADUC, right-clicked Staff OU > “New” > “User.”  
Entered:  
Full name: Mike Brown  
User logon name: mike  
Password:****** 
Clicked “Finish.”


Double-clicked StaffTeam > “Members” tab > “Add” > typed mike > “Check Names” > “OK.”  
In Computer Management, added mike to “Remote Desktop Users.”  
Tested: RDP as mylab\mike, accessed \\localhost\StaffFiles, created file, confirmed empty desktop.







### *****CONFIGURE DRIVE MAPPING GPO
**Purpose and Usefulness**: Maps the shared folder `\\localhost\StaffFiles` to drive `Z:` for `jane` and `mike`, making it easily accessible in File Explorer for efficient file access and management.

1. Verified share path: In File Explorer, right-clicked `C:\StaffFiles` > “Properties” > “Sharing” tab, noted path `\\localhost\StaffFiles`.  
2. In Group Policy Management, right-clicked `Staff` OU > “Create a GPO” > named `Staff Drive Mapping`.  
3. Right-clicked `Staff Drive Mapping` > “Edit.”  
4. Navigated: User Configuration > Preferences > Windows Settings > Drive Maps.  
5. Right-clicked > “New” > “Mapped Drive.”  
6. Set:  
   - Action: Update  
   - Location: `\\localhost\StaffFiles`  
   - Label: Staff Files (optional)
   - Drive Letter: `Z:`  
9. Closed editor, confirmed “Authenticated Users” in Security Filtering.  
10. Ran `gpupdate /force` in Command Prompt (admin).  
11. Tested: RDP as `mylab\jane`, verified `Staff Files (Z:)` in File Explorer, created file. Repeated for `mike`.
