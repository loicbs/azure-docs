---
title: Scenarios to use a virtual network
description: Scenarios, resources, and limitations to deploy container groups to an Azure virtual network.
ms.topic: conceptual
ms.author: tomcassidy
author: tomvcassidy
ms.service: container-instances
services: container-instances
ms.date: 06/17/2022
---

# Virtual network scenarios and resources

[Azure Virtual Network](../virtual-network/virtual-networks-overview.md) provides secure, private networking for your Azure and on-premises resources. By deploying container groups into an Azure virtual network, your containers can communicate securely with other resources in the virtual network. 

This article provides background about virtual network scenarios, limitations, and resources. For deployment examples using the Azure CLI, see [Deploy container instances into an Azure virtual network](container-instances-vnet.md).

> [!IMPORTANT]
> Container group deployment to a virtual network is generally available for Linux and Windows containers, in most regions where Azure Container Instances is available. For details, see [Regions and resource availability](container-instances-region-availability.md). 

## Scenarios

Container groups deployed into an Azure virtual network enable scenarios like:

* Direct communication between container groups in the same subnet
* Send [task-based](container-instances-restart-policy.md) workload output from container instances to a database in the virtual network
* Retrieve content for container instances from a [service endpoint](../virtual-network/virtual-network-service-endpoints-overview.md) in the virtual network
* Enable container communication with on-premises resources through a [VPN gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md) or [ExpressRoute](../expressroute/expressroute-introduction.md)
* Integrate with [Azure Firewall](../firewall/overview.md) to identify outbound traffic originating from the container 
* Resolve names via the internal Azure DNS for communication with Azure resources in the virtual network, such as virtual machines
* Use NSG rules to control container access to subnets or other network resources

## Unsupported networking scenarios 

* **Azure Load Balancer** - Placing an Azure Load Balancer in front of container instances in a networked container group is not supported
* **Global virtual network peering** - Global peering (connecting virtual networks across Azure regions) is not supported
* **Public IP or DNS label** - Container groups deployed to a virtual network don't currently support exposing containers directly to the internet with a public IP address or a fully qualified domain name
* **Managed Identity with Virtual Network in Azure Government Regions** - Managed Identity with virtual networking capabilities is not supported in Azure Government Regions

## Other limitations

* To deploy container groups to a subnet, the subnet can't contain other resource types. Remove all existing resources from an existing subnet prior to deploying container groups to it, or create a new subnet.
* To deploy container groups to a subnet, the subnet and the container group must be on the same Azure subscription.
* You can't enable a [liveness probe](container-instances-liveness-probe.md) or [readiness probe](container-instances-readiness-probe.md) in a container group deployed to a virtual network.
* Due to the additional networking resources involved, deployments to a virtual network are typically slower than deploying a standard container instance.
* Outbound connections to port 25 and 19390 are not supported at this time. Port 19390 needs to be opened in your Firewall for connecting to ACI from Azure ortal when container groups are deployed in virtual networks.
* For inbound connections, the firewall should also allow all ip addresses within the virtual network.
* If you are connecting your container group to an Azure Storage Account, you must add a [service endpoint](../virtual-network/virtual-network-service-endpoints-overview.md) to that resource.
* [IPv6 addresses](../virtual-network/ip-services/ipv6-overview.md) are not supported at this time.
* Depending on your subscription type, [certain ports may be blocked](../virtual-network/network-security-groups-overview.md#azure-platform-considerations).

## Required network resources

There are three Azure Virtual Network resources required for deploying container groups to a virtual network: the [virtual network](#virtual-network) itself, a [delegated subnet](#subnet-delegated) within the virtual network, and a [network profile](#network-profile). 

### Virtual network

A virtual network defines the address space in which you create one or more subnets. You then deploy Azure resources (like container groups) into the subnets in your virtual network.

### Subnet (delegated)

Subnets segment the virtual network into separate address spaces usable by the Azure resources you place in them. You create one or several subnets within a virtual network.

The subnet that you use for container groups may contain only container groups. When you first deploy a container group to a subnet, Azure delegates that subnet to Azure Container Instances. Once delegated, the subnet can be used only for container groups. If you attempt to deploy resources other than container groups to a delegated subnet, the operation fails.

### Network profile

[!INCLUDE [network profile callout](./includes/network-profile/network-profile-callout.md)]

A network profile is a network configuration template for Azure resources. It specifies certain network properties for the resource, for example, the subnet into which it should be deployed. When you first use the [az container create][az-container-create] command to deploy a container group to a subnet (and thus a virtual network), Azure creates a network profile for you. You can then use that network profile for future deployments to the subnet. 

To use a Resource Manager template, YAML file, or a programmatic method to deploy a container group to a subnet, you need to provide the full Resource Manager resource ID of a network profile. You can use a profile previously created using [az container create][az-container-create], or create a profile using a Resource Manager template (see [template example](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-vnet) and [reference](/azure/templates/microsoft.network/networkprofiles)). To get the ID of a previously created profile, use the [az network profile list][az-network-profile-list] command. 

In the following diagram, several container groups have been deployed to a subnet delegated to Azure Container Instances. Once you've deployed one container group to a subnet, you can deploy additional container groups to it by specifying the same network profile.

![Container groups within a virtual network][aci-vnet-01]

## Next steps

* For deployment examples with the Azure CLI, see [Deploy container instances into an Azure virtual network](container-instances-vnet.md).
* To deploy a new virtual network, subnet, network profile, and container group using a Resource Manager template, see [Create an Azure container group with VNet](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-vnet
).
* When using the [Azure portal](container-instances-quickstart-portal.md) to create a container instance, you can also provide settings for a new or exsting virtual network on the **Networking** tab.


<!-- IMAGES -->
[aci-vnet-01]: ./media/container-instances-virtual-network-concepts/aci-vnet-01.png

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az_container_create
[az-network-profile-list]: /cli/azure/network/profile#az_network_profile_list
