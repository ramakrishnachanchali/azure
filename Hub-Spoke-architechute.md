# Azure Hub-and-Spoke Architecture Deployment Script

This script automates the deployment of a **Hub-and-Spoke** architecture on **Azure**. It includes the creation of **Resource Groups**, **Virtual Networks (VNets)**, **Subnets**, **Network Security Groups (NSGs)**, and **Virtual Machines (VMs)**. This architecture consists of one **Hub** and two **Spokes**.

---


## 1. HUB Creation

```bash
# Define variables
RG=AZ-HUB-RG
IMAGE=Canonical:0001-com-ubuntu-server-focal-daily:20_04-daily-lts-gen2:latest

# Create Azure Resource Group
echo "Creating Azure Resource Group"
az group create --location eastus -n ${RG}

# Create Azure Virtual Network (VNet)
echo "Creating Azure Virtual Network"
az network vnet create -g ${RG} -n ${RG}-vNET1 --address-prefix 10.34.0.0/16 --subnet-name Jump-Svr-Subnet-1 --subnet-prefix 10.34.1.0/24 -l eastus

# Create additional subnets in the existing VNET
echo "Creating other subnets in existing VNET"
az network vnet subnet create -g ${RG} --vnet-name ${RG}-vNET1 -n AzureFirewallSubnet \
    --address-prefixes 10.34.10.0/24
az network vnet subnet create -g ${RG} --vnet-name ${RG}-vNET1 -n GatewaySubnet \
    --address-prefixes 10.34.20.0/24
az network vnet subnet create -g ${RG} --vnet-name ${RG}-vNET1 -n AzureBastionSubnet \
    --address-prefixes 10.34.30.0/24

# Create NSG (Network Security Group)
echo "Creating NSG"
az network nsg create -g ${RG} -n ${RG}_NSG1 --location eastus

# Create NSG rules
echo "Creating NSG Rule 1: Allow all TCP traffic"
az network nsg rule create -g ${RG} --nsg-name ${RG}_NSG1 -n ${RG}_NSG1_RULE1 --priority 100 \
  --source-address-prefixes '*' --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges '*' \
  --access Allow --protocol Tcp --description "Allowing All Traffic For Now"

echo "Creating NSG Rule 2: Allow ICMP traffic"
az network nsg rule create -g ${RG} --nsg-name ${RG}_NSG1 -n ${RG}_NSG1_RULE2 --priority 101 \
  --source-address-prefixes '*' --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges '*' \
  --access Allow --protocol Icmp --description "Allowing ICMP Traffic For Now"

# Create a Virtual Machine (VM)
echo "Creating Virtual Machine"
az vm create --resource-group ${RG} --name JUMPLINUXVM1 --image $IMAGE --vnet-name ${RG}-vNET1 --subnet Jump-Svr-Subnet-1 --admin-username adminsree --admin-password "India@123456" --size Standard_B1s --nsg ${RG}_NSG1 --storage-sku StandardSSD_LRS --private-ip-address 10.34.1.10 --zone 1 --custom-data ./clouddrive/cloud-init3.txt

[make --custom-data ./clouddrive/cloud-init3.txt cloud-init3.text](--custom-data ./clouddrive/cloud-init3.txt)

```


## 2. SPOKE 1 Setup Script

This script automates the process of setting up **SPOKE 1** on Azure. It includes the creation of a **Resource Group**, **Virtual Network**, **Network Security Group (NSG)**, **NSG Rules**, and a **Virtual Machine (VM)**.

```bash
### SPOKE 1 Creation ###

# Define Resource Group and Image
RG=AZ-SP1-RG
IMAGE=Canonical:0001-com-ubuntu-server-focal-daily:20_04-daily-lts-gen2:latest

# Step 1: Create Resource Group
echo "Creating Azure Resource Group for SPOKE 1"
az group create --location eastus -n ${RG}

# Step 2: Create Virtual Network
echo "Creating Azure Virtual Network for SPOKE 1"
az network vnet create -g ${RG} -n ${RG}-vNET1 --address-prefix 172.16.0.0/16 \
  --subnet-name ${RG}-Subnet-1 --subnet-prefix 172.16.1.0/24 -l eastus

# Step 3: Create Network Security Group (NSG)
echo "Creating NSG for SPOKE 1"
az network nsg create -g ${RG} -n ${RG}_NSG1 --location eastus

# Step 4: Create NSG Rules
echo "Creating NSG Rule 1: Allow TCP traffic"
az network nsg rule create -g ${RG} --nsg-name ${RG}_NSG1 -n ${RG}_NSG1_RULE1 \
  --priority 100 --source-address-prefixes '*' --source-port-ranges '*' \
  --destination-address-prefixes '*' --destination-port-ranges '*' --access Allow --protocol Tcp \
  --description "Allowing All Traffic For Now"

echo "Creating NSG Rule 2: Allow ICMP traffic"
az network nsg rule create -g ${RG} --nsg-name ${RG}_NSG1 -n ${RG}_NSG1_RULE2 \
  --priority 101 --source-address-prefixes '*' --source-port-ranges '*' \
  --destination-address-prefixes '*' --destination-port-ranges '*' --access Allow --protocol Icmp \
  --description "Allowing ICMP Traffic For Now"

# Step 5: Create Virtual Machine (VM)
echo "Creating a Virtual Machine for SPOKE 1"
az vm create --resource-group ${RG} --name TestServer01 --image $IMAGE --vnet-name ${RG}-vNET1 \
  --subnet ${RG}-Subnet-1 --admin-username adminram --admin-password "India@123456" \
  --size Standard_B1s --storage-sku StandardSSD_LRS --nsg ${RG}_NSG1
```

## 3. SPOKE 2 Setup Script

This script automates the process of setting up **SPOKE 2** on Azure. It includes the creation of a **Resource Group**, **Virtual Network**, **Network Security Group (NSG)**, **NSG Rules**, and a **Virtual Machine (VM)**.

```bash
### SPOKE 2 Creation ###

# Define Resource Group and Image
RG=AZ-SP2-RG
IMAGE=Canonical:0001-com-ubuntu-server-focal-daily:20_04-daily-lts-gen2:latest

# Step 1: Create Resource Group
echo "Creating Azure Resource Group for SPOKE 2"
az group create --location eastus -n ${RG}

# Step 2: Create Virtual Network
echo "Creating Azure Virtual Network for SPOKE 2"
az network vnet create -g ${RG} -n ${RG}-vNET1 --address-prefix 172.17.0.0/16 \
  --subnet-name ${RG}-Subnet-1 --subnet-prefix 172.17.1.0/24 -l eastus

# Step 3: Create Network Security Group (NSG)
echo "Creating NSG for SPOKE 2"
az network nsg create -g ${RG} -n ${RG}_NSG1 --location westus

# Step 4: Create NSG Rules
echo "Creating NSG Rule 1: Allow TCP traffic"
az network nsg rule create -g ${RG} --nsg-name ${RG}_NSG1 -n ${RG}_NSG1_RULE1 \
  --priority 100 --source-address-prefixes '*' --source-port-ranges '*' \
  --destination-address-prefixes '*' --destination-port-ranges '*' --access Allow --protocol Tcp \
  --description "Allowing All Traffic For Now"

echo "Creating NSG Rule 2: Allow ICMP traffic"
az network nsg rule create -g ${RG} --nsg-name ${RG}_NSG1 -n ${RG}_NSG1_RULE2 \
  --priority 101 --source-address-prefixes '*' --source-port-ranges '*' \
  --destination-address-prefixes '*' --destination-port-ranges '*' --access Allow --protocol Icmp \
  --description "Allowing ICMP Traffic For Now"

# Step 5: Create Virtual Machine (VM)
echo "Creating a Virtual Machine for SPOKE 2"
az vm create --resource-group ${RG} --name TestServer01 --image $IMAGE --vnet-name ${RG}-vNET1 \
  --subnet ${RG}-Subnet-1 --admin-username adminram --admin-password "India@123456" \
  --size Standard_B1s --storage-sku StandardSSD_LRS --nsg ${RG}_NSG1
```

## Cloud-init3.txt

```yaml
# cloud-config
package_upgrade: true
packages:
  - nginx
  - stress
  - unzip
  - jq
  - net-tools
  - curl

runcmd:
  - service nginx restart
  - systemctl enable nginx
  - echo "<h1>$(cat /etc/hostname)</h1>"  >> /var/www/html/index.nginx-debian.html
```
