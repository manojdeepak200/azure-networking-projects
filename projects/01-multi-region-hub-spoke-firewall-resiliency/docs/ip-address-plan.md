# IP Address Plan — Multi-Region Hub-and-Spoke Network

## 1. Design Objectives

The IP addressing design supports the following network architecture requirements:

- Two Azure regions
- Hub-and-Spoke architecture
- Multiple spokes per region
- Regional connectivity
- Global VNet Peering
- Centralized Azure Firewall
- Future network expansion
- Workload segmentation
- Network troubleshooting and route analysis

All Virtual Networks use non-overlapping address spaces to support VNet peering, routing, and inter-region connectivity without IP address conflicts.

---

## 2. Addressing Strategy

Private IPv4 addressing is used throughout the environment.

The selected private address range is:

`10.0.0.0/8`

Dedicated address ranges are allocated to each region to simplify network identification, routing, troubleshooting, and future expansion.

Each region contains:

- One Hub VNet
- One Application Spoke VNet
- One Shared Services Spoke VNet

Additional address ranges are reserved for future spokes, environments, and workload expansion.

The addressing strategy follows a hierarchical allocation model:

- Region 1 uses the `10.0.0.0/8` address range beginning with `10.0.0.0/16`.
- Region 2 uses a separate allocation beginning with `10.10.0.0/16`.
- Each VNet receives a dedicated `/16` address space.
- Subnets are allocated within their respective VNet address spaces.

This structure simplifies identification of regional resources and allows future network expansion without redesigning the existing address allocation.

---

## 3. Regional Address Allocation

| Region | Purpose | Address Space |
|---|---|---|
| Region 1 | Hub VNet | `10.0.0.0/16` |
| Region 1 | Application Spoke | `10.1.0.0/16` |
| Region 1 | Shared Services Spoke | `10.2.0.0/16` |
| Region 1 | Reserved | `10.3.0.0/16` – `10.9.0.0/16` |
| Region 2 | Hub VNet | `10.10.0.0/16` |
| Region 2 | Application Spoke | `10.11.0.0/16` |
| Region 2 | Shared Services Spoke | `10.12.0.0/16` |
| Region 2 | Reserved | `10.13.0.0/16` – `10.19.0.0/16` |

All allocated VNet address spaces are unique and non-overlapping.

---

## 4. Region 1 — Hub VNet

**VNet Name:** `vnet-hub-region1`

**Address Space:** `10.0.0.0/16`

| Subnet | Address Range | Purpose |
|---|---|---|
| AzureFirewallSubnet | `10.0.0.0/26` | Azure Firewall deployment |
| AzureFirewallManagementSubnet | `10.0.0.64/26` | Dedicated Firewall management subnet |
| GatewaySubnet | `10.0.1.0/27` | Reserved for future VPN Gateway |
| snet-management | `10.0.2.0/24` | Management infrastructure |
| snet-shared-network | `10.0.3.0/24` | Shared network services |
| Reserved | Remaining address space | Future expansion |

### Design Notes

- `AzureFirewallSubnet` is the required subnet name for Azure Firewall.
- Azure Firewall requires a dedicated subnet with a minimum supported size of `/26`.
- `AzureFirewallManagementSubnet` is reserved for firewall management configurations that require a dedicated management IP address.
- `GatewaySubnet` is reserved for potential VPN Gateway deployment.
- Management and shared network service subnets are separated from firewall infrastructure.
- Remaining address space is reserved for future network appliances and shared infrastructure.

---

## 5. Region 1 — Application Spoke

**VNet Name:** `vnet-spoke-app-region1`

**Address Space:** `10.1.0.0/16`

| Subnet | Address Range | Purpose |
|---|---|---|
| snet-app | `10.1.1.0/24` | Application workloads |
| snet-web | `10.1.2.0/24` | Web tier |
| snet-app-private | `10.1.3.0/24` | Private application services |
| snet-private-endpoints | `10.1.4.0/24` | Private Endpoint resources |
| Reserved | Remaining address space | Future expansion |

### Design Notes

The Application Spoke separates application components into dedicated subnets to support workload segmentation, security controls, routing policies, and troubleshooting.

The subnet structure allows future deployment of:

- Web servers
- Application servers
- Private application services
- Private Endpoints
- Additional application tiers

---

## 6. Region 1 — Shared Services Spoke

**VNet Name:** `vnet-spoke-shared-region1`

**Address Space:** `10.2.0.0/16`

| Subnet | Address Range | Purpose |
|---|---|---|
| snet-management | `10.2.1.0/24` | Management workloads |
| snet-monitoring | `10.2.2.0/24` | Monitoring infrastructure |
| snet-automation | `10.2.3.0/24` | Automation workloads |
| snet-private-endpoints | `10.2.4.0/24` | Private Endpoint resources |
| Reserved | Remaining address space | Future expansion |

### Design Notes

The Shared Services Spoke hosts infrastructure used by multiple workloads or application environments.

Potential services include:

- Monitoring and logging infrastructure
- Automation workloads
- Management services
- Shared infrastructure components
- Private Endpoint connectivity

The separation of shared services from application workloads simplifies security policy management and network administration.

---

## 7. Region 2 — Hub VNet

**VNet Name:** `vnet-hub-region2`

**Address Space:** `10.10.0.0/16`

| Subnet | Address Range | Purpose |
|---|---|---|
| AzureFirewallSubnet | `10.10.0.0/26` | Azure Firewall deployment |
| AzureFirewallManagementSubnet | `10.10.0.64/26` | Dedicated Firewall management subnet |
| GatewaySubnet | `10.10.1.0/27` | Reserved for future VPN Gateway |
| snet-management | `10.10.2.0/24` | Management infrastructure |
| snet-shared-network | `10.10.3.0/24` | Shared network services |
| Reserved | Remaining address space | Future expansion |

### Design Notes

Region 2 follows the same Hub VNet subnet structure as Region 1.

This consistency simplifies:

- Regional deployment
- Network troubleshooting
- Firewall configuration
- Routing design
- Infrastructure as Code implementation
- Future operational automation

---

## 8. Region 2 — Application Spoke

**VNet Name:** `vnet-spoke-app-region2`

**Address Space:** `10.11.0.0/16`

| Subnet | Address Range | Purpose |
|---|---|---|
| snet-app | `10.11.1.0/24` | Application workloads |
| snet-web | `10.11.2.0/24` | Web tier |
| snet-app-private | `10.11.3.0/24` | Private application services |
| snet-private-endpoints | `10.11.4.0/24` | Private Endpoint resources |
| Reserved | Remaining address space | Future expansion |

### Design Notes

The Region 2 Application Spoke mirrors the Region 1 application network structure.

This supports consistent application deployment across regions and allows the network architecture to accommodate regional workload expansion and application-level disaster recovery scenarios.

---

## 9. Region 2 — Shared Services Spoke

**VNet Name:** `vnet-spoke-shared-region2`

**Address Space:** `10.12.0.0/16`

| Subnet | Address Range | Purpose |
|---|---|---|
| snet-management | `10.12.1.0/24` | Management workloads |
| snet-monitoring | `10.12.2.0/24` | Monitoring infrastructure |
| snet-automation | `10.12.3.0/24` | Automation workloads |
| snet-private-endpoints | `10.12.4.0/24` | Private Endpoint resources |
| Reserved | Remaining address space | Future expansion |

### Design Notes

The Shared Services Spoke provides a dedicated network for common infrastructure components in Region 2.

The subnet allocation follows the same design pattern as Region 1 to maintain consistency across both regions.

---

## 10. Address Allocation Summary

| Network | Address Space |
|---|---|
| Hub Region 1 | `10.0.0.0/16` |
| Application Spoke Region 1 | `10.1.0.0/16` |
| Shared Services Spoke Region 1 | `10.2.0.0/16` |
| Hub Region 2 | `10.10.0.0/16` |
| Application Spoke Region 2 | `10.11.0.0/16` |
| Shared Services Spoke Region 2 | `10.12.0.0/16` |

All VNet address spaces are unique and non-overlapping.

---

## 11. Future Expansion

The following address ranges are reserved for future network expansion.

### Region 1

| Address Space | Intended Use |
|---|---|
| `10.3.0.0/16` | Additional Application Spoke |
| `10.4.0.0/16` | Data Spoke |
| `10.5.0.0/16` | Additional Shared Services |
| `10.6.0.0/16` | Development Environment |
| `10.7.0.0/16` | Test Environment |
| `10.8.0.0/16` | Reserved for expansion |
| `10.9.0.0/16` | Reserved for expansion |

### Region 2

| Address Space | Intended Use |
|---|---|
| `10.13.0.0/16` | Additional Application Spoke |
| `10.14.0.0/16` | Data Spoke |
| `10.15.0.0/16` | Additional Shared Services |
| `10.16.0.0/16` | Development Environment |
| `10.17.0.0/16` | Test Environment |
| `10.18.0.0/16` | Reserved for expansion |
| `10.19.0.0/16` | Reserved for expansion |

These allocations provide room for additional workloads, environments, network services, and regional infrastructure.

---

## 12. Design Considerations

### Non-Overlapping Address Spaces

All VNets use unique address spaces.

Overlapping IP ranges can prevent VNet peering and introduce routing conflicts when connecting networks.

The regional address allocation ensures that the current VNets and reserved future networks can be deployed without address overlap.

### Subnet Sizing

Subnet sizes are determined based on:

- Expected workload capacity
- Azure service requirements
- Private Endpoint requirements
- Network appliance requirements
- Network architecture
- Future expansion

Subnet sizing should account for Azure-reserved IP addresses and the number of usable addresses available within each subnet.

### Reserved Subnets

Certain Azure services require specific subnet names and configurations.

Examples include:

- `AzureFirewallSubnet`
- `AzureFirewallManagementSubnet`
- `GatewaySubnet`

These subnet names and address ranges are reserved for their corresponding Azure services.

### Private Endpoint Subnets

Private Endpoint resources are placed in dedicated subnets to simplify network organization and security management.

Private Endpoint subnet policies must be configured according to the routing and security requirements of the environment, particularly when Network Security Groups or User Defined Routes are applied to Private Endpoint traffic.

### Address Space Expansion

The `/16` VNet allocations provide significant flexibility for the lab environment.

In production environments, address allocation would typically be determined by:

- Enterprise IP Address Management standards
- Existing corporate network address ranges
- On-premises connectivity requirements
- Expected workload capacity
- Organizational network segmentation policies
- Long-term growth forecasts

The allocation structure is designed to make future expansion easier while maintaining a consistent regional addressing pattern.

---

## 13. IP Addressing Design Summary

The IP address allocation provides a consistent addressing structure across both Azure regions.

The design includes:

- Six primary VNets
- Dedicated Hub and Spoke address spaces
- Regional address allocation
- Non-overlapping VNet ranges
- Dedicated subnet allocation for Azure Firewall
- Reserved GatewaySubnet ranges
- Dedicated application and shared-services subnets
- Private Endpoint subnet allocation
- Reserved address ranges for future expansion

The addressing structure supports the multi-region Hub-and-Spoke architecture and provides a foundation for VNet peering, centralized firewall routing, workload segmentation, and future network expansion.
