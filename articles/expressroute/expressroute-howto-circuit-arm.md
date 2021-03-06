<properties
   pageTitle="Create and modify an ExpressRoute circuit by using Resource Manager and PowerShell | Microsoft Azure"
   description="This article describes how to create and provision an ExpressRoute circuit. It also shows you how to check the circuit, update it, or delete and deprovision it."
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carolz"
   editor=""
   tags="azure-resource-manager"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="03/03/2016"
   ms.author="cherylmc"/>

# Create and modify an ExpressRoute circuit by using Resource Manager and PowerShell

   > [AZURE.SELECTOR]
   [PowerShell - Classic](expressroute-howto-circuit-classic.md)
   [PowerShell - Resource Manager](expressroute-howto-circuit-arm.md)

This article describes how to create an Azure ExpressRoute circuit by using Windows PowerShell cmdlets and the Azure Resource Manager deployment model. The following steps also show you how to check the status of the circuit, update it, or delete and deprovision it.

   [AZURE.INCLUDE [vpn-gateway-sm-rm](../../includes/vpn-gateway-sm-rm-include.md)]

## Configuration prerequisites

To create an ExpressRoute circuit, you need to:

- Obtain the latest version of the Azure PowerShell modules, version 1.0 or later. For step-by-step guidance on how to configure your computer to use the PowerShell modules, follow the instructions on the [How to install and configure Azure PowerShell](../powershell-install-configure.md) page.
- Review the [Prerequisites](expressroute-prerequisites.md) page and the [Workflows](expressroute-workflows.md) page before you begin configuration.

## Create and provision an ExpressRoute circuit

**Step 1. Import the PowerShell module for ExpressRoute.**

To begin using the ExpressRoute cmdlets, you must install the latest PowerShell installer from [PowerShell gallery](http://www.powershellgallery.com/) and import the Resource Manager modules into the PowerShell session. You will run PowerShell as an administrator.

```
Install-Module AzureRM

Install-AzureRM
```

Import all the AzureRM modules within the known semantic version range:

```
Import-AzureRM
```

You can also import a select module within the known semantic version range:

```
Import-Module AzureRM.Network
```

Sign in to your account:

```
Login-AzureRmAccount
```

Select the subscription that you want to create an ExpressRoute circuit:

```
Select-AzureRmSubscription -SubscriptionId "<subscription ID>"   			
```

**Step 2. Get the list of supported providers, locations, and bandwidths.**

Before creating an ExpressRoute circuit, you need the list of connectivity providers, supported locations, and bandwidth options. The PowerShell *Get-AzureRmExpressRouteServiceProvider* cmdlet returns this information, which you’ll use in later steps.

```
PS C:\> Get-AzureRmExpressRouteServiceProvider
```

Check to see if your connectivity provider is listed there. Make a note of the following because you will need them later when you create a circuit:

- Name
- PeeringLocations
- BandwidthsOffered

You are now ready to create an ExpressRoute circuit.   

**Step 3.  Create an ExpressRoute circuit.**

If you don't already have a resource group, you must create one before you create your ExpressRoute circuit. You can do so by running the following command:

```
New-AzureRmResourceGroup -Name "ExpressRouteResourceGroup" -Location "West US"
```

The following example shows how to create a 200-Mbps ExpressRoute circuit through Equinix in Silicon Valley. If you are using a different provider and different settings, substitute that information when you make your request. The following is an example request for a new service key:

```
New-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup" -Location "West US" -SkuTier Standard -SkuFamily MeteredData -ServiceProviderName "Equinix" -PeeringLocation "Silicon Valley" -BandwidthInMbps 200
```

Make sure that you specify the correct SKU tier and SKU family:

- SKU tier determines whether an ExpressRoute standard or an ExpressRoute premium add-on is enabled. You can specify *standard* to get the standard SKU or *premium* for the premium add-on.
- SKU family determines the billing type. You can select *metereddata* for a metered data plan and *unlimiteddata* for an unlimited data plan. **Note:** After you've created a circuit, you will not be able to change the billing type.

The response contains the service key. You can get detailed descriptions of all parameters by running the following:

```
get-help New-AzureRmExpressRouteCircuit -detailed
```

**Step 4.  List all ExpressRoute circuits.**

To get a list of all ExpressRoute circuits you created, run the *Get-AzureRmExpressRouteCircuit* command:

```
#Getting service key
Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

The response will look similar to the following example:

```
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
   		                           }
CircuitProvisioningState          : Enabled
ServiceProviderProvisioningState  : NotProvisioned
ServiceProviderNotes              :
ServiceProviderProperties         : {
                                      "ServiceProviderName": "Equinix",
                                      "PeeringLocation": "Silicon Valley",
                                      "BandwidthInMbps": 200
                                    }
ServiceKey                        : **************************************
Peerings                          : []
```

You can retrieve this information at any time by using the *Get-AzureRmExpressRouteCircuit* cmdlet. Making the call with no parameters will list all circuits. Your service key will be listed in the *ServiceKey* field.

```
Get-AzureRmExpressRouteCircuit
```

The response will look similar to the following example:

```
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
   		                           }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                     "ServiceProviderName": "Equinix",
                                     "PeeringLocation": "Silicon Valley",
                                     "BandwidthInMbps": 200
   		                           }
ServiceKey                       : **************************************
Peerings                         : []
```

You can get detailed descriptions of all parameters by running the following:

```
get-help Get-AzureRmExpressRouteCircuit -detailed
```

**Step 5.  Send the service key to your connectivity provider for provisioning.**

When you create a new ExpressRoute circuit, the circuit will be in the following state:

```
ServiceProviderProvisioningState : NotProvisioned

CircuitProvisioningState         : Enabled
```

*ServiceProviderProvisioningState* provides information on the current state of provisioning on the service provider side, and Status provides the state on the Microsoft side. For you to be able to use an ExpressRoute circuit, it must be in the following state:

```
ServiceProviderProvisioningState : Provisioned

CircuitProvisioningState         : Enabled
```

The circuit will change to the following state when the connectivity provider is in the process of enabling it for you:

```
ServiceProviderProvisioningState : Provisioned

Status                           : Enabled
```

**Step 6.  Periodically check the status and the state of the circuit key.**

Checking the status and the state of the circuit key informs you when your provider has enabled your circuit. After the circuit has been configured, *ServiceProviderProvisioningState* appears as *Provisioned*, as shown in the following example:

```
Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

The response will look similar to the following example:

```
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                     "ServiceProviderName": "Equinix",
                                     "PeeringLocation": "Silicon Valley",
                                     "BandwidthInMbps": 200
   	                            }
ServiceKey                       : **************************************
Peerings                         : []

**Step 7.  Create your routing configuration.**

For step-by-step instructions, refer to the [ExpressRoute circuit routing configuration (create and modify circuit peerings)](expressroute-howto-routing-arm.md).

**Step 8.  Link a virtual network to an ExpressRoute circuit.**

Next, link a virtual network to your ExpressRoute circuit. You can use [this template](https://github.com/Azure/azure-quickstart-templates/tree/ecad62c231848ace2fbdc36cbe3dc04a96edd58c/301-expressroute-circuit-vnet-connection) when you work with the Resource Manager deployment mode. We're currently working on steps to accomplish this by using PowerShell.

## Get the status of an ExpressRoute circuit

You can retrieve this information at any time by using the *Get-AzureRmExpressRouteCircuit* cmdlet. Making the call with no parameters will list all circuits.

```
Get-AzureRmExpressRouteCircuit
```

The response will be similar to the following example:

```
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
   		                             "ServiceProviderName": "Equinix",
   		                             "PeeringLocation": "Silicon Valley",
   		                             "BandwidthInMbps": 200
   		                           }
ServiceKey                       : **************************************
Peerings                         : []
```

You can get information on a specific ExpressRoute circuit by passing the resource group name and circuit name as a parameter to the call:

```
Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
```

The response will look similar to the following example:

```
Name                             : ExpressRouteARMCircuit
ResourceGroupName                : ExpressRouteResourceGroup
Location                         : westus
Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
   		                             "Tier": "Standard",
   		                             "Family": "MeteredData"
   		                           }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             :
ServiceProviderProperties        : {
                                     "ServiceProviderName": "Equinix",
                                     "PeeringLocation": "Silicon Valley",
                                     "BandwidthInMbps": 200
   		                           }
ServiceKey                       : **************************************
Peerings                         : []
```

You can get detailed descriptions of all parameters by running the following:

```
get-help get-azurededicatedcircuit -detailed
```

## Modify an ExpressRoute circuit

You can modify certain properties of an ExpressRoute circuit without impacting connectivity.

You can do the following, with no downtime:

- Enable or disable an ExpressRoute premium add-on for your ExpressRoute circuit.
- Increase the bandwidth of your ExpressRoute circuit.

For more information on limits and limitations, refer to the [ExpressRoute FAQ](expressroute-faqs.md) page.

### Enable the ExpressRoute premium add-on

You can enable the ExpressRoute premium add-on for your existing circuit by using the following PowerShell snippet:

```
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Tier = "Premium"
$ckt.sku.Name = "Premium_MeteredData"

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

The circuit will now have the ExpressRoute premium add-on features enabled. Note that Microsoft will begin billing you for the premium add-on capability as soon as the command has successfully run.

### Disable the ExpressRoute premium add-on

You can disable the ExpressRoute premium add-on for the existing circuit by using the following PowerShell cmdlet:

```
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.Sku.Tier = "Standard"
$ckt.sku.Name = "Standard_MeteredData"

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

The premium add-on is now disabled for the circuit.

Note that this operation can fail if you are using resources greater than what is permitted for the standard circuit.

- Before you downgrade from premium to standard, you must ensure that the number of virtual networks linked to the circuit is less than 10. If you don't do so, your update request fails and Microsoft will bill you at premium rates.
- You must unlink all virtual networks in other geopolitical regions. If you don't do so, your update request will fail and Microsoft will bill you at premium rates.
- Your route table must be less than 4,000 routes for private peering. If your route table size is greater than 4,000 routes, the BGP session drops and won't be reenabled until the number of advertised prefixes goes below 4,000.

### Update the ExpressRoute circuit bandwidth

For supported bandwidth options for your provider, check the [ExpressRoute FAQ](expressroute-faqs.md) page. You can pick any size greater than the size of your existing circuit. After you decide what size you need, use the following command to resize your circuit:

```
$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

$ckt.ServiceProviderProperties.BandwidthInMbps = 1000

Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
```

Your circuit will be sized up on the Microsoft side. Then you must contact your connectivity provider to update configurations on their side to match this change. After you make this notification, Microsoft will begin billing you for the updated bandwidth option.

**Important**: You cannot reduce the bandwidth of an ExpressRoute circuit without disruption. Downgrading bandwidth requires you to deprovision the ExpressRoute circuit and then reprovision a new ExpressRoute circuit.

## Delete and deprovision an ExpressRoute circuit

You can delete your ExpressRoute circuit by running the following command:

```
Remove-AzureRmExpressRouteCircuit -ResourceGroupName "ExpressRouteResourceGroup" -Name "ExpressRouteARMCircuit"
```

Note that for this operation to succeed, you must unlink all virtual networks from the ExpressRoute circuit. If this operation fails, check whether any virtual networks are linked to the circuit.

If the ExpressRoute circuit service provider provisioning state is enabled, the status moves to *disabling* from an enabled state. You must work with your service provider to deprovision the circuit on their side. Microsoft will continue to reserve resources and bill you until the service provider completes deprovisioning the circuit and notifies us.

If the service provider has deprovisioned the circuit (the service provider provisioning state is set to *not provisioned*) before you run the above cmdlet, Microsoft will deprovision the circuit and stop billing you.

## Next steps

After you create your circuit, make sure that you do the following:

- [Create and modify routing for your ExpressRoute circuit](expressroute-howto-routing-arm.md)
- [Link your virtual network to your ExpressRoute circuit](expressroute-howto-linkvnet-arm.md)
