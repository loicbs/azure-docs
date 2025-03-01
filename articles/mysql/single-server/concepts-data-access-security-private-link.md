---
title: Private Link - Azure Database for MySQL
description: Learn how Private link works for Azure Database for MySQL.
ms.service: mysql
ms.subservice: single-server
ms.topic: conceptual
author: mksuni
ms.author: sumuth
ms.date: 06/20/2022
---

# Private Link for Azure Database for MySQL

[!INCLUDE[applies-to-mysql-single-server](../includes/applies-to-mysql-single-server.md)]

[!INCLUDE[azure-database-for-mysql-single-server-deprecation](../includes/azure-database-for-mysql-single-server-deprecation.md)]

Private Link allows you to connect to various PaaS services in Azure via a private endpoint. Azure Private Link essentially brings Azure services inside your private Virtual Network (VNet). The PaaS resources can be accessed using the private IP address just like any other resource in the VNet.

For a list to PaaS services that support Private Link functionality, review the Private Link [documentation](../../private-link/index.yml). A private endpoint is a private IP address within a specific [VNet](../../virtual-network/virtual-networks-overview.md) and Subnet.

> [!NOTE]
> The private link feature is only available for Azure Database for MySQL servers in the General Purpose or Memory Optimized pricing tiers. Ensure the database server is in one of these pricing tiers.

## Data exfiltration prevention

Data ex-filtration in Azure Database for MySQL is when an authorized user, such as a database admin, is able to extract data from one system and move it to another location or system outside the organization. For example, the user moves the data to a storage account owned by a third party.

Consider a scenario with a user running MySQL Workbench inside an Azure Virtual Machine (VM) that is connecting to an Azure Database for MySQL server provisioned in West US. The example below shows how to limit access with public endpoints on Azure Database for MySQL using network access controls.

* Disable all Azure service traffic to Azure Database for MySQL via the public endpoint by setting *Allow Azure Services* to OFF. Ensure no IP addresses or ranges are allowed to access the server either via [firewall rules](./concepts-firewall-rules.md) or [virtual network service endpoints](./concepts-data-access-and-security-vnet.md).

* Only allow traffic to the Azure Database for MySQL using the Private IP address of the VM. For more information, see the articles on [Service Endpoint](concepts-data-access-and-security-vnet.md) and [VNet firewall rules](how-to-manage-vnet-using-portal.md).

* On the Azure VM, narrow down the scope of outgoing connection by using Network Security Groups (NSGs) and Service Tags as follows

    * Specify an NSG rule to allow traffic for *Service Tag = SQL.WestUs* - only allowing connection to Azure Database for MySQL in West US
    * Specify an NSG rule (with a higher priority) to deny traffic for *Service Tag = SQL* - denying connections to Update to Azure Database for MySQL in all regions</br></br>

At the end of this setup, the Azure VM can connect only to Azure Database for MySQL in the West US region. However, the connectivity isn't restricted to a single Azure Database for MySQL. The VM can still connect to any Azure Database for MySQL in the West US region, including the databases that aren't part of the subscription. While we've reduced the scope of data exfiltration in the above scenario to a specific region, we haven't eliminated it altogether.</br>

With Private Link, you can now set up network access controls like NSGs to restrict access to the private endpoint. Individual Azure PaaS resources are then mapped to specific private endpoints. A malicious insider can only access the mapped PaaS resource (for example an Azure Database for MySQL) and no other resource.

## On-premises connectivity over private peering

When you connect to the public endpoint from on-premises machines, your IP address needs to be added to the IP-based firewall using a server-level firewall rule. While this model works well for allowing access to individual machines for dev or test workloads, it's difficult to manage in a production environment.

With Private Link, you can enable cross-premises access to the private endpoint using [Express Route](https://azure.microsoft.com/services/expressroute/) (ER), private peering or [VPN tunnel](../../vpn-gateway/index.yml). They can subsequently disable all access via public endpoint and not use the IP-based firewall.

> [!NOTE]
> In some cases the Azure Database for MySQL and the VNet-subnet are in different subscriptions. In these cases you must ensure the following configurations:
> - Make sure that both subscriptions have the **Microsoft.DBforMySQL** resource provider registered. For more information refer [resource-manager-registration][resource-manager-portal]

## Configure Private Link for Azure Database for MySQL

### Creation Process

Private endpoints are required to enable Private Link. This can be done using the following how-to guides.

* [Azure portal](./how-to-configure-private-link-portal.md)
* [CLI](./how-to-configure-private-link-cli.md)

### Approval Process
Once the network admin creates the private endpoint (PE), the MySQL admin can manage the private endpoint Connection (PEC) to Azure Database for MySQL. This separation of duties between the network admin and the DBA is helpful for management of the Azure Database for MySQL connectivity. 

* Navigate to the Azure Database for MySQL server resource in the Azure portal. 
    * Select the private endpoint connections in the left pane
    * Shows a list of all private endpoint Connections (PECs)
    * Corresponding private endpoint (PE) created

:::image type="content" source="media/concepts-data-access-and-security-private-link/select-private-link-portal.png" alt-text="select the private endpoint portal":::

* Select an individual PEC from the list by selecting it.

:::image type="content" source="media/concepts-data-access-and-security-private-link/select-private-link.png" alt-text="select the private endpoint pending approval":::

* The MySQL server admin can choose to approve or reject a PEC and optionally add a short text response.

:::image type="content" source="media/concepts-data-access-and-security-private-link/select-private-link-message.png" alt-text="select the private endpoint message":::

* After approval or rejection, the list will reflect the appropriate state along with the response text

:::image type="content" source="media/concepts-data-access-and-security-private-link/show-private-link-approved-connection.png" alt-text="select the private endpoint final state":::

## Use cases of Private Link for Azure Database for MySQL

Clients can connect to the private endpoint from the same VNet, [peered VNet](../../virtual-network/virtual-network-peering-overview.md) in same region or across regions, or via [VNet-to-VNet connection](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) across regions. Additionally, clients can connect from on-premises using ExpressRoute, private peering, or VPN tunneling. Below is a simplified diagram showing the common use cases.

:::image type="content" source="media/concepts-data-access-and-security-private-link/show-private-link-overview-1.png" alt-text="select the private endpoint overview":::

### Connecting from an Azure VM in Peered Virtual Network (VNet)
Configure [VNet peering](../../virtual-network/tutorial-connect-virtual-networks-powershell.md) to establish connectivity to the Azure Database for MySQL from an Azure VM in a peered VNet.

### Connecting from an Azure VM in VNet-to-VNet environment
Configure [VNet-to-VNet VPN gateway connection](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) to establish connectivity to a Azure Database for MySQL from an Azure VM in a different region or subscription.

### Connecting from an on-premises environment over VPN
To establish connectivity from an on-premises environment to the Azure Database for MySQL, choose and implement one of the options:

* [Point-to-Site connection](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)
* [Site-to-Site VPN connection](../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)
* [ExpressRoute circuit](../../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)

## Private Link combined with firewall rules

The following situations and outcomes are possible when you use Private Link in combination with firewall rules:

* If you don't configure any firewall rules, then by default, no traffic will be able to access the Azure Database for MySQL.

* If you configure public traffic or a service endpoint and you create private endpoints, then different types of incoming traffic are authorized by the corresponding type of firewall rule.

* If you don't configure any public traffic or service endpoint and you create private endpoints, then the Azure Database for MySQL is accessible only through the private endpoints. If you don't configure public traffic or a service endpoint, after all approved private endpoints are rejected or deleted, no traffic will be able to access the Azure Database for MySQL.

## Deny public access for Azure Database for MySQL

If you want to rely only on private endpoints for accessing their Azure Database for MySQL, you can disable setting all public endpoints (i.e. [firewall rules](concepts-firewall-rules.md) and [VNet service endpoints](concepts-data-access-and-security-vnet.md)) by setting the **Deny Public Network Access** configuration on the database server. 

When this setting is set to *YES*, only connections via private endpoints are allowed to your Azure Database for MySQL. When this setting is set to *NO*, clients can connect to your Azure Database for MySQL based on your firewall or VNet service endpoint settings. Additionally, once the value of the Private network access is set, customers cannot add and/or update existing 'Firewall rules' and 'VNet service endpoint rules'.

> [!NOTE]
> This feature is available in all Azure regions where Azure Database for MySQL - Single Server supports General Purpose and Memory Optimized pricing tiers.
>
> This setting does not have any impact on the SSL and TLS configurations for your Azure Database for MySQL.

To learn how to set the **Deny Public Network Access** for your Azure Database for MySQL from Azure portal, refer to [How to configure Deny Public Network Access](how-to-deny-public-network-access.md).

## Next steps

To learn more about Azure Database for MySQL security features, see the following articles:

* To configure a firewall for Azure Database for MySQL, see [Firewall support](./concepts-firewall-rules.md).

* To learn how to configure a virtual network service endpoint for your Azure Database for MySQL, see [Configure access from virtual networks](./concepts-data-access-and-security-vnet.md).

* For an overview of Azure Database for MySQL connectivity, see [Azure Database for MySQL Connectivity Architecture](./concepts-connectivity-architecture.md)

<!-- Link references, to text, Within this same GitHub repo. -->
[resource-manager-portal]: ../../azure-resource-manager/management/resource-providers-and-types.md
