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
- AZ203 is valid to take until Aug 31st 2020, it is valid for two years after successfully taking the exam

### Virtual Machines

- IaaS (Infrastructure as a Service)
- use the pricing calculator based on VM size / model to understand your needs
- turn on / off whenever you wish, configure auto-shutdowns, configure `runbooks` to get them working in intervals
- get discount for reserving 1 or 3 years upfront or not including the Windows OS license
- create a VM with Azure Portal wizard, the basic setup
    - subscription
    - resource group
    - VM name
    - (geo) region, **does affect cost (check in calculator)**
    - availability options (separate VMs per zones)
    - Azure Spot Instance (it is a shortlived VM with a lower price)
    - OS image file (e.g. Windows Server for Datacenter)
    - sizing (e.g. Standard DS1 v2, from FREE to premium tiers)
    - admin account details (username and password)
    - inbound network port control (none, allow selected)
- select disk options
    - OS disk type, standard HDD, standard SSD and premium SSD (premium is suggested)
        - larger disk sizes gives more IOPS and throughput
    - encryption at rest is DEFAULT, Azure manages by default, you can use __Azure data vault__ to use your encryption key
    - you can attach additional Disks/LUNs you customize as the main disk
- select networking options
    - create a virtual network with a new subnet and public IP
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
    - ARM template scripts can be in: **Powershell, Ruby, Bash, and C#**
- a finished VM consists of multiple Azure resources; e.g. disk, netw. sec. group, IP, vnet, nic, VM, shutdown scheme, each of them have properties linked to them such as time of creation
- each VM resource can be inspected and reconfigured
- you can connect to a Windows VM through RDP, SSH or BASTION (does not need the 3389 RDP protocol) service
- to enable RDP -> netw. sec. group. -> open 3389 port with new inbound security rule
- VM size can be resized through VM resource options

> EXAM NOTE: download the ARM templates of a VM and examine them in detail and understand how they work
>
> - both parameters.json and template.json document
> - you can store the template docs into a library (by default the parameters are empty and you need to apply them)
>
> EXERCISE: test saving an ARM template and applying a parameters file to deploy another VM based on a library template, the SETTINGS section should have values applied

## Using Powershell Az modules

- start with installing Powershell Az module: <https://docs.microsoft.com/en-us/powershell/azure/azurerm/install-azurerm-ps?view=azurermps-6.13.0>
- reopen the PS terminal and you will have the option to login with the Az module cmdlet: `Connect-AzAccount`

## Encrypting a VM

- create an azure key vault service (**in the same region as the VM** to make this work)

> EXERCISE: create a key vault service (same region as VM which is standard or above size) with a key (this can be done from PS or Azure CLI also)

- can store (encryption) keys, secrets and certificates
- for encryption of a VM create a new Key value, give it a meaningful name, RSA-2048 is default algorithm, without expiry options
- encrypting a VM from PS via Az PS modules:

```powershell
> $keyVault = Get-AzKeyVault -name "your_kv_name"
> $kekurl = (Get-AzKeyVaultKey -VaultName $keyVault.VaultName -Name "your_key_name").Key.kid

# 'kek' stands for key encryption key

# encrypt the VMs drive:
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
# create a new VM from Az module cmdlets:
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

### Azure Batch and AKS

- Azure Batch and Azure Kubernetes Service is out of scope for AZ-204
- opening the Azure Kubernetes web dashboard will not work through cloud shell, you must install it locally on the machine with `az aks install-cli` and then merge locally the AKS credentials of the target cluster and browse with `az aks browse --resource-group "rg_name" --name aksClusterName`

> EXERCISE: `git clone https://github.com/Azure-Samples/azure-voting-app-redis` and run `docker-compose up -d` to start the services locally. Create a new ACR on Azure and tag your frontend project then push it upstream to your newly created ACR instance.

- when pushing to ACR you are required to login from CLI first with: `az acr login --name "your_acr_name"`

## Day 3

- app services resource (PaaS - Platform as a Service)

> IMPORTANT: spend time on comparing prices between BASIC, STANDARD and PREMIUM app service plans

- test creating a new webapp resource with a STANDARD app service plan, test hosting a real web application to it (depending on which application type you selected)
- app services can scale up (vertically) or out (horizontally), BASIC plans don't allow automatic out-scaling, you need a STANDARD+ plan
- besides web apps we can create `web jobs` which are attached to a webapp (with its app service plan) which represent a background running job that can take care of some work
- deployment slots allow multiple deployments per a webppp where one can seamlessly replace between deployment without any down-time
- additional slots will create a postfix url, e.g. if your base domain is <https://myapp.azurewebsites.com>, adding a new slot with name staging will create a new subdomain site on <https://myapp-staging.azurewebsites.com>
- deployment slots can be used to swap between slots and to easily execute A/B testing by directing 50% to each of e.g. two slots
- app service webapp Filesystem logging enables you to show a live log stream in the azure portal from your service standard output

> EXERCISE: test creating resources such as `group`, `vm`, `appservice`, `webapp`, `storage`, `functionapp`, `batch`, `aks`, `container`, `acr`. Using `az webapp up` is a super easy utility to bring up a whole web app with an app service plan (can use existing one). You can use the `--html` option which will try to auto determine which type of app it is.

## Day 4

### Function apps

- lighter than web apps in terms of consumption plans; you can run it on a app service plan or serverless (you are charged per function/CPU usage run).
- 1st million runs is for FREE
- you can choose a Windows or Linux OS, Windows allows Java and Linux Python
- they required a storage account
- functions can be created from inside the Azure portal, ie. you edit a .csx file (if you chose .NET/C#)
- functions are executed on triggers; two most common ones are Webhook HTTP request and Timer based
- possible triggers:
    - HTTP request (Webhook)
    - Timer (CRON syntax template based)
    - Queue Storage
    - Service Bus Queue
    - Service Bus Topic
    - Event Hub
    - Blob Storage
    - Cosmos DB
    - IoT Hub (Event hub)
    - SendGrid (email)
    - Event Grid

> EXERCISE: try creating a function app with a storage V2 LRS attached with `New-AzFunctionApp` PS cmdlet.

## Day 5

### CosmosDB

- CosmosDB, a NoSQL azure service, choose between different DB providers/engines like SQL, MongoDB, Cassandra, (Azure) Tables, or Gremlin
- Azure Cosmos DB is Microsoft's globally distributed, multi-model database service
- sub 10ms latency, faster than Azure Table Storage (a service within the Azure storage account)
- enables you to elastically and independently scale throughput and storage across any number of Azure regions worldwide
- has active multi-master replication for reads/writes of databases, ie. offers a high SLA to demanding clients
- no manual index or schema management, all of the data is auto-indexed all the time
- data is encrypted in rest and motion all the time
- core feature besides blazing low latency is it allows **multiple levels of data consistency**:
    - strong (ASAP on all regions, but can take time due to copying, multi-master)
    - bounded staleness (you control the delay of data being written to other regions)
    - session (default, the same app has all region access, but other apps/session might have a delay)
    - consistent prefix (ensures the order of data being written and has delays due to that ordering)
    - eventual (likes, comments, no order is guaranteed but the data gets written after a delay)
- when compared to Azure Table Storage (within a storage account) **you can not control consistency like this** with the storage account

> EXERCISE: go through V4 tutorial for SQL: <https://docs.microsoft.com/en-us/azure/cosmos-db/create-sql-api-dotnet-v4>

