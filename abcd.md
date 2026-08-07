# Microsoft Service and Resource Reuse Guide for Lab 1

## Purpose

This guide identifies the Microsoft services and Azure resources that can be reused from the other two labs when building Lab 1. It is aligned with current Microsoft guidance for Azure SQL Managed Instance, Azure Migrate, and Azure Virtual Machines.

---

## 1. Exact Microsoft Service Names to Use

The following official Microsoft service names are relevant to the lab comparison and can be used in Lab 1:

- Azure SQL Managed Instance
- Azure Migrate
- Azure Virtual Machines
- Azure Database Migration Service
- Azure Storage Account
- Azure Resource Group
- Azure Virtual Network
- Azure Network Interface
- Azure Network Security Group
- Azure Public IP Address
- Azure Private IP Address
- Azure Managed Disks

---

## 2. Azure Resources That Can Be Reused

| Microsoft Service or Resource | Why It Is Relevant | Safe to Reuse in Lab 1 | Notes |
|---|---|---|---|
| Azure SQL Managed Instance | Target platform for migrated SQL workloads | Yes | Best fit for the database layer |
| Azure Virtual Machines | Target compute for application hosting | Yes | Best fit for the application-hosting layer |
| Azure Migrate | Migration planning and execution | Yes | Useful for the overall migration storyline |
| Azure Storage Account | Migration data staging and storage | Yes | Useful when backup or migration data needs to be staged |
| Azure Virtual Network | Network foundation for Azure resources | Yes | Required for secure communication between application and data tiers |
| Network Interface Card (NIC) | Connects the VM to the network | Yes | Required for Azure VM connectivity |
| Network Security Group (NSG) | Controls inbound and outbound traffic | Yes | Important for secure access |
| Public IP / Private IP | Provides external or internal connectivity | Yes | Use based on the connectivity design |
| Managed Disks | Provides persistent VM storage | Yes | Preferred storage model for Azure VMs |
| Resource Group | Logical container for Azure resources | Yes | Standard Azure deployment pattern |

---

## 3. Reusable Technical Patterns for Lab 1

The following technical patterns can be reused from the other two labs because they represent common Azure migration design patterns.

### A. Migration Lifecycle Pattern

Use the following sequence in Lab 1:

1. Prepare the Azure target environment.
2. Create the required Azure resources.
3. Configure networking and access.
4. Migrate the workload.
5. Validate the final state.

### B. Azure Target Setup Pattern

Lab 1 can reuse the following target setup pattern:

- Create a resource group.
- Create a virtual network.
- Create a subnet and supporting network components.
- Create a VM for application hosting.
- Create Azure SQL Managed Instance for the database tier.
- Configure connectivity between the application and database tiers.

### C. Security and Connectivity Pattern

Use the following Azure networking and security components:

- Virtual Network
- Subnet
- Network Interface Card (NIC)
- Network Security Group (NSG)
- Public IP or Private IP, as required
- Access control configuration

### D. Validation Pattern

End the lab with the following validation steps:

- Confirm that the VM is running.
- Confirm that the database service is available.
- Confirm that application-to-database connectivity works.
- Confirm that the migrated workload is running successfully.

---

## 4. Relevant Microsoft Guidance for Lab 1

### Azure SQL Managed Instance

Azure SQL Managed Instance is a fully managed PaaS database engine that provides:

- Near-complete compatibility with the SQL Server Database Engine
- Native virtual network support
- Automated patching and updates
- Automated backups
- High availability options
- Microsoft Entra authentication support

This makes Azure SQL Managed Instance a suitable choice for the database layer of Lab 1.

### Azure Migrate

Azure Migrate provides a unified platform for migration activities such as:

- Discovery
- Assessment
- Migration execution
- Support for servers, databases, web apps, and related workloads

This makes Azure Migrate suitable for the overall migration storyline of Lab 1.

### Azure Virtual Machines

Azure Virtual Machines provide on-demand, scalable compute resources. When configuring a VM, consider:

- Resource name
- Region
- VM size
- Operating system image
- Networking configuration
- Supporting resources such as NICs, NSGs, IP addresses, and managed disks

This makes Azure Virtual Machines suitable for the application-hosting layer of Lab 1.

---

## 5. Updates from the Older Lab Style

The following changes should be considered to keep the lab aligned with current Microsoft guidance.

### A. Use Current Official Service Names

Use the following official names:

- Azure SQL Managed Instance
- Azure Migrate
- Azure Virtual Machines

Avoid relying only on abbreviated or generic terms such as "SQL MI" or "migration and modernization."

### B. Prefer Managed Azure Resources for VMs

Azure Virtual Machines should be configured with the appropriate supporting resources, including:

- Managed Disks
- Network Interfaces
- Network Security Groups
- Virtual Network integration

### C. Prefer PaaS and Managed Services Where Appropriate

For the database layer, Azure SQL Managed Instance provides a more managed approach than a generic SQL Server deployment.

### D. Use Azure Migrate as the Migration Platform

Azure Migrate supports a broader migration workflow covering servers, databases, web apps, and migration planning. It can therefore serve as the umbrella service for the lab's migration storyline.

---

## 6. Copy-Ready Summary for Managers

The first lab can reuse the same Azure migration patterns from the other two labs while adopting current Microsoft service names and resource models. The core services include **Azure SQL Managed Instance** for the database layer, **Azure Virtual Machines** for application hosting, and **Azure Migrate** for the migration workflow. Supporting Azure resources such as **Virtual Network, Network Interface Card, Network Security Group, Public/Private IP addresses, Managed Disks, and Storage Account** can also be reused for connectivity, security, and storage. This approach provides a current Microsoft-aligned architecture while maintaining a consistent end-to-end migration experience.

---

## 7. Recommended Reuse List for Lab 1

For a single copy-ready list, use:

- Azure SQL Managed Instance
- Azure Virtual Machines
- Azure Migrate
- Azure Storage Account
- Azure Resource Group
- Azure Virtual Network
- Network Interface Card (NIC)
- Network Security Group (NSG)
- Public IP / Private IP
- Managed Disks
