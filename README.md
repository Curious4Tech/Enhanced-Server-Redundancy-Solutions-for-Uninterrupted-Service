# Win-server-redundancy

# Setting Up Domain Controller Redundancy – Windows Server

This guide explains how to configure redundancy for your Active Directory domain to ensure service continuity within your corporate network. By setting up a secondary domain controller, you can ensure that in case of a failure, another machine will take over the domain controller role, thereby maintaining the stability and longevity of your infrastructure.

## Prerequisites

Before starting, make sure to meet the following requirements:

- **Operating System**: Use the same version of Windows Server on all hosts (in this example, domain functional level is set to Windows Server 2016).
- **Shared Domain**: DC1 and DC2 should be joined to the same Active Directory domain.
- **Admin Accounts**: Have local administrator accounts with super-user rights on each server.
- **Hardware Requirements**: A computer capable of running 3 virtual machines simultaneously (e.g., VirtualBox, Hyper-V) .

## Network Configuration for Virtual Machines

| Machine       | Static IP           | Gateway        | DNS 1            | DNS 2            |
|---------------|---------------------|----------------|-------------------|-------------------|
| **DC1**       | 172.25.124.177/20  | 172.25.112.1  | 127.0.0.1        |     8.8.8.8
| **DC2**       | 172.25.120.231/20  | 172.25.112.1  | 172.25.124.177  | 127.0.0.1        |
| **W10-Client**| 172.25.122.176/20  | 172.25.112.1  | 172.25.124.177  | 172.25.120.231  |

## Configuration Steps

### I. Step 1: Set the IP and Hostname for DC1
1. **Set a static IP** for DC1:
    - IP: `172.25.124.177/20`
    - Gateway: `172.25.112.1`
    - DNS1: `127.0.0.1`
    - DNS2: `8.8.8.8`
  
![image](https://github.com/user-attachments/assets/bbae1002-1585-4c17-a79a-067d9f97997b)


2. **Change the hostname** of the server to `DC1`.

3. Use the `Rename-Computer` cmdlet: Run the following command to rename your computer. Replace `New-ComputerName` with the desired name for your computer or server, here `DC1`.
   
```
   Rename-Computer -NewName "New-ComputerName" -Force -Restart
```

The `-Force` parameter suppresses prompts, and the `-Restart` parameter will restart the computer automatically for the changes to take effect.

### II. Step 2: Install ADDS Role on DC1
1. **Install the Active Directory Domain Services (ADDS) role**:
   - In the Server Manager, add the ADDS role and follow the instructions to promote the server to a domain controller.
   - Or Use the `Install-WindowsFeature` cmdlet to install the AD DS role, by runing this powershell command with administrative privelege.
     
   ```
      Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
   ```

![image](https://github.com/user-attachments/assets/5df83843-6d6c-4154-8ec5-927e77a54472)


1. **Domain Configuration**: During the promotion, set up a domain name (e.g., `nextechiq.local`).

Use the `Install-ADDSForest` cmdlet to create a new forest and domain.Remember to run this command as an administrator in PowerShell.
```
Install-ADDSForest -DomainName "nextechiq.local" -DomainNetbiosName "NEXTECHIQ" -ForestMode "WinThreshold" -DomainMode "WinThreshold" -InstallDns -Confirm:$false -SafeModeAdministratorPassword (ConvertTo-SecureString -AsPlainText "YourSafeModePassword" -Force)
```

![image](https://github.com/user-attachments/assets/3ac9c12f-3752-4541-b583-6b9173e20744)

Replace:

   ` YourDomainName.com` with your desired domain name.
    `YourNetBIOSName` with the NetBIOS name for the domain.
    `WinThreshold` with the functional level you prefer (use WinThreshold for Windows Server 2016 or newer).
    `YourSafeModePassword` with the DSRM (Directory Services Restore Mode) password.
    
 The server will automatically restart to complete the Active Directory Domain Services installation. 

 Once the server restarts, you'll be able to connect to the new domain.
 
![image](https://github.com/user-attachments/assets/c1ac9e09-7c45-4d5c-9520-a062c453b0f0)

 
### III. Step 3: Prepare DC2 Server for Redundancy
1. **Change the hostname** of the second server to `DC2`.

      
```
   Rename-Computer -NewName "New-ComputerName" -Force -Restart
```

Replace `New-ComputerName` with the desired name for your computer or server, here `DC2`.

3. **Check connectivity** with DC1 to ensure DC2 can communicate with it by  just pinging the DC1 from DC2.

![image](https://github.com/user-attachments/assets/a52bd2fa-b2a3-4678-85ff-1327d3e3260b)


### IV. Step 4: Install ADDS Role on DC2
1. **Add the ADDS role** on DC2.

   
 ```
      Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
   ```
   
3. **Promote DC2 as a secondary domain controller** by joining the existing domain, in my case here `nextechiq.local`.

Use the `Install-ADDSDomainController` cmdlet to add your domain controller to your existing domain, here `nextechiq.local`. Remember to run this command as an administrator in PowerShell.
   
```
   Install-ADDSDomainController -DomainName "YourDomainName.com" -InstallDns -Credential (Get-Credential) -SafeModeAdministratorPassword (ConvertTo-SecureString -AsPlainText "YourSafeModePassword" -Force)
```
Replace:
        ` YourDomainName.com` with your existing domain name, here `nextechiq.local`.
         `YourSafeModePassword` with the DSRM (Directory Services Restore Mode) password.

If you see that you're being prompted for credentials while running the `Install-ADDSDomainController` command.


![image](https://github.com/user-attachments/assets/c68987cd-963e-445a-a051-dfe9be8768d1)


In this prompt:
1. For `User name`: Enter your domain admin account as `YOURNETBIOSNAME\Administrator`, in my case here `nextechiq\Administrator`.
2. For `Password`: Enter your domain admin password (not the DSRM password)

This credential is needed because you're adding this server as an additional domain controller to your existing domain, in my case here "nextechiq.local". The system needs domain administrator privileges to replicate AD data from your existing DC. 

### V. Step 5: Test the Redundancy
1. **Test redundancy from a client**: Set up a Windows client (e.g., `Win 10 Client`) to use DC1 and DC2 as DNS servers and test domain access.


![image](https://github.com/user-attachments/assets/f1f0a8f2-d2f6-450a-812c-a99bf9ea1ecc)

   
2. **Verify service continuity**: Disable DC1.

![image](https://github.com/user-attachments/assets/110a4699-359f-42f3-8691-318bc74fa0a5)

Now, check if  DC2 takes over for FQDN ping.

![image](https://github.com/user-attachments/assets/7bc37d34-5aa2-49bd-81e0-0476e1db270c)

 As you can see on the screenshot above, DC2 is responding to the `nextechiq.local` ping from `Win 10 Client`
## Notes

> ⚠️ **Important**: Windows Server does not currently support failover clustering for domain controllers. It is therefore essential to configure each DNS client with at least two DNS servers (DC1 and DC2 in this example).
