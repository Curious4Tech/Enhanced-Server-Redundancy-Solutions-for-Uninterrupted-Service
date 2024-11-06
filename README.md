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
| **DC1**       | 192.168.130.101/24  | 192.168.130.2  | 127.0.0.1        | 192.168.130.102  |
| **DC2**       | 192.168.130.102/24  | 192.168.130.2  | 192.168.130.101  | 127.0.0.1        |
| **W10-Client**| 192.168.130.200/24  | 192.168.130.2  | 192.168.130.101  | 192.168.130.102  |

## Configuration Steps

### I. Step 1: Set the IP and Hostname for DC1
1. **Set a static IP** for DC1:
    - IP: `192.168.130.101/24`
    - Gateway: `192.168.130.2`
    - DNS1: `127.0.0.1`
    - DNS2: `1.1.1.1`
  
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
2. **Check connectivity** with DC1 to ensure DC2 can communicate with it.

### IV. Step 4: Install ADDS Role on DC2
1. **Add the ADDS role** on DC2.
2. **Promote DC2 as a secondary domain controller** by joining the existing domain `nextechiq.local`.

### V. Step 5: Test the Redundancy
1. **Test redundancy from a client**: Set up a Windows client (e.g., `W10-Client`) to use DC1 and DC2 as DNS servers and test domain access.
2. **Verify service continuity**: Disable DC1 and check that DC2 takes over authentication for users.

## Notes

> ⚠️ **Important**: Windows Server does not currently support failover clustering for domain controllers. It is therefore essential to configure each DNS client with at least two DNS servers (DC1 and DC2 in this example).
