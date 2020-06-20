# Microsoft AZ-204 (beta) exam learning questions, notes and prep exercises

## Learning guides and labs

- learning guide: <https://www.udemy.com/course/70532-azure>
- MCT practice materials: <https://github.com/MicrosoftLearning/AZ-204-DevelopingSolutionsforMicrosoftAzure>
- Microsoft Learn labs: <https://docs.microsoft.com/en-us/learn/roles/developer>

## Prerequisites for guides and labs

- Working Azure account (allows access to portal.azure.com)
    - free registration, 12 months of free services, 25 forever free, 1 month 200$ to spend for free
- Visual Studio 2019 (Community Edition will do)
- vscode
- powershell / bash

## DAY 1

- beta stands for that the results are not immediate (at the point of writing this June 2020)
- AZ203 is valid to take until Aug 31st 2020, it is valid for to years after successfuly taking the exam

### Virtual Machines

- use the pricing calculator based on VM size / model
- turn on / off whenever you wish
- get discount for reserving 1 or 3 years upfront or not including the Windows OS license
- create a VM with Azure Portal wizard, the basic setup
    - subscription
    - resource group
    - VM name
    - (geo) region, **does affect cost (check in calculator)**
    - availability options (separate VMs per zones)
    - Azure Spot Instance (shortlived VM with a lower price)
    - OS image file (e.g. Windows Server for Datacenter)
    - sizing (e.g. Standard DS1 v2, from FREE tier to premium)
    - admin account details (username, password)
    - inbound network port control (none, allow selected)
- select disk options
    - OS disk type, standard HDD, standard SSD and premium SSD (premium is suggested)
        - larger disk sizes gives more IOPS and throughput
    - encryption at rest is DEFAULT, Azure manages by default, you can use __Azure data vault__ to use your encryption key
    - you can attach additional Disks/LUNs you customize as the main disk
- select networking options
    - create a virtual network with a new subnet and public IOPS
    - select NIC security group
    - allow public inbound ports (same as before)
    - create a new (network) load balancer or use an existing
- select management (logging) options
    - boot, OS guest diagnostics on / off
    - configure time when the VM shuts down, sends email notification if needed
    - enable VM backup
- select advanced options (startup init logic)
    - startup PS scripts, install extensions, for Linux is "Cloud init", runs scripts
    - configure a dedicated Host Machines
    - configure VM proximity placement group (to have lower latency between VMs)
    - configure VM gen 1 / 2 (gen 2 does not yet include Disk encryption)
- configure tags (name, value and total resources with that tag)

> EXERCISE: create a VM from Azure CLI and Powershell Az module

## VM connection through RDP and ARM templates

- review VM creation tab, size, price and options, download the VM creation JSON (ARM) template and parameters files
    - ARM template scripts can be in: Powershell, Ruby, Bash, and C#
- a finished VM consists of multiple Azure resources; e.g. disk, netw. sec. group, IP, vnet, nic, VM, shutdown scheme, each of them have properties linked to them such as time of creation
- each VM resource can be inspected and reconfigured
- you can connect to a Windows VM through RDP, SSH or BASTION (does not need the 3389 RDP protocol) service
- to enable RDP -> netw. sec. group. -> open 3389 port with new inbound security rule
- VM size can be resized through VM resource options
- EXAM NOTE: download the ARM templates of a VM and examine them in detail and understand how they work
    - both parameters.json and template.json document
    - you can store the template docs into a library (by default the parameters are empty and you need to apply them)

> EXERCISE: test saving an ARM template and applying a parameters file to deploy another VM based on a library template, the SETTINGS section should have values applied

## Using Powershell Az modules

- start with installing Powershell Az module: <https://docs.microsoft.com/en-us/powershell/azure/azurerm/install-azurerm-ps?view=azurermps-6.13.0>
- reopen PS terminal and you will have the option to login wtih `Connect-AzAccount`

## Encrypting a VM

- create an azure key vault service (**in the same region as the VM** to make this work)

> EXERCISE: create a key vault service (same region as VM which is standard or above size) with a key (this can be done from PS or Azure CLI also)

- can store (encryption) keys, secrets and certificates
- for encryption of a VM create a new Key value, give it a meaningful name, RSA-2048 is default without expiry options
- encrypting a VM from PS via Az PS modules:

```powershell
> $keyVault = Get-AzKeyVault -name "your_kv_name"
> $kekurl = (Get-AzKeyVaultKey -VaultName $keyVault.VaultName -Name "your_key_name").Key.kid

>  Set-AzVMDiskEncryptionExtension -resourcegroupname "your_rg_name" `
>> -vmname "win10-01" -DiskEncryptionKeyVaultUrl $keyVault.VaultUri `
>> -DiskEncryptionKeyVaultId $keyVault.ResourceId `
>> -KeyEncryptionKeyUrl $kekurl `
>> -KeyEncryptionKeyVaultId $keyVault.ResourceId

# wait for it to finish, check in portal or CLI if VM disk is encrypted
```

- verify disk encryption from AZ CLI

```powershell
> az vm encryption show --resource-group "your_rg_name" --name "your_vm_name"
```

## DAY 2

### Creating a VM from PS Az

- use the `New-AzVm` command
- usually all commands are similar and follow a pattern such as `New-Az[some Azure resource] -name "resource_name"`
- creating a VM should be easy as:

```powershell
> New-AzVm -resourcegroupname "az204" `
>> -name "az204vm-01" `
>> -location NorthEU `
>> -virtualnetworkname "az204vnet-01" `
>> -subnetname "default" `
>> -securitygroupname "az204sg-01" `
>> -publicipaddressname "az204pip-01" `
>> -openports 80,3389

# you will be asked for username and password credentials also
# other options such as size will take a default value
```
