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
   
```
   Rename-Computer -NewName "New-ComputerName" -Force -Restart
```

### II. Step 2: Install ADDS Role on DC1
1. **Install the Active Directory Domain Services (ADDS) role**:
   - In the Server Manager, add the ADDS role and follow the instructions to promote the server to a domain controller.
2. **Domain Configuration**: During the promotion, set up a domain name (e.g., `lgds.local`).

### III. Step 3: Prepare DC2 Server for Redundancy
1. **Change the hostname** of the second server to `DC2`.
2. **Check connectivity** with DC1 to ensure DC2 can communicate with it.

### IV. Step 4: Install ADDS Role on DC2
1. **Add the ADDS role** on DC2.
2. **Promote DC2 as a secondary domain controller** by joining the existing domain `lgds.local`.

### V. Step 5: Test the Redundancy
1. **Test redundancy from a client**: Set up a Windows client (e.g., `W10-Client`) to use DC1 and DC2 as DNS servers and test domain access.
2. **Verify service continuity**: Disable DC1 and check that DC2 takes over authentication for users.

## Notes

> ⚠️ **Important**: Windows Server does not currently support failover clustering for domain controllers. It is therefore essential to configure each DNS client with at least two DNS servers (DC1 and DC2 in this example).
