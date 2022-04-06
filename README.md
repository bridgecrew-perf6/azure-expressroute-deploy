# Bicep and ARM Template to deploy an ExpressRoute

The goal of this repo is to create ExpressRoute circuits with private peering config pre-populated before initiating the provisioning process with the service provider. Note that over the Azure Portal you can only add private peering configuration after the circuit gets fully provisioned with the service provider.

## Prerequisites

- It assumes you have already an Resource Group created if not create one via portal or command. Here is an example over CLI:

```bash
az group create -n er-circuits --location southcentralus
```

### Deploy using CLI

Note: use Linux shell with Azure CLI or Bash [Cloud Shell](https://shell.azure.com)

```bash
#List default Subscription being used
az account list --query "[?isDefault == \`true\`].{Name:name, IsDefault:isDefault}" -o table
#Select your target subscription name (Only needed if you want to use a different subscription)
az account set --subscription <subscription name>  #Select proper Subscription to create the ER Circuit.

# Define variables
rg=er-circuits # Specify the resource group
ercircuitname='ER-Chicago-Circuit'
provider='Megaport' #Specify you service provider. You can use command "az network express-route list-service-providers -o table" to get a full list of the ExpressRoute Service Providers
asn=65001 #Specify an ASN
primaryPeerAddressPrefix='172.100.112.0/30' #Set the primary connection /30 range
secondaryPeerAddressPrefix='172.100.112.4/30' #Set the secondary connection /30 range
peeringlocation=Chicago #For a list of all locations see https://aka.ms/erlocations per provider.
bandwidthInMbps=50 #Set Circuit bandwidth between 50 and 10000 (10Gbps).
sku=Standard # You can set Local (over 1Gbps), Standard and Premium SKUs.

# Create ER Circuit
az deployment group create \
--resource-group $rg \
--template-uri https://raw.githubusercontent.com/dmauser/azure-expressroute-deploy/main/ercircuit.json \
--parameters ercircuitname=$ercircuitname asn=$asn primaryPeerAddressPrefix=$primaryPeerAddressPrefix secondaryPeerAddressPrefix=$secondaryPeerAddressPrefix provider=$provider peeringlocation=$peeringlocation bandwidthInMbps=$bandwidthInMbps sku=$sku

#Validate if the Private peering configuration has been populated
az network express-route show -n $ercircuitname -g $rg --query 'peerings'
```