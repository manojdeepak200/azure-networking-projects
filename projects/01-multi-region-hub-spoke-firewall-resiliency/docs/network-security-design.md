# Network Security Design — Multi-Region Hub-and-Spoke Network

## 1. Purpose

This document defines the network security design for the multi-region Azure Hub-and-Spoke architecture.

The security design uses Azure Firewall, Network Security Groups, routing controls, and network segmentation to control traffic between workloads, regional networks, shared services, and the Internet.

The objective is to provide centralized security enforcement while maintaining controlled network connectivity across both Azure regions.

---

## 2. Security Architecture

The network security architecture consists of the following components:

- Azure Firewall — Centralized network traffic inspection and security enforcement
- Network Security Groups (NSGs) — Subnet-level traffic filtering
- User Defined Routes (UDRs) — Traffic path control
- Virtual Network segmentation — Isolation between application and shared services workloads
- Global VNet Peering — Controlled inter-region connectivity
- Azure Monitor and Log Analytics — Security monitoring and traffic logging

Each Azure region contains its own Azure Firewall within the regional Hub VNet.

Network traffic is controlled using a combination of routing configuration, Firewall policies, and Network Security Groups.

---

## 3. Azure Firewall

Azure Firewall is deployed in each regional Hub Virtual Network to provide centralized network security.

### Responsibilities

- Control outbound Internet connectivity
- Inspect traffic routed through the Firewall
- Control communication between network segments
- Enforce inter-region traffic policies
- Allow or deny traffic based on configured rules
- Provide network and application-level filtering
- Generate traffic logs for monitoring and troubleshooting

### Firewall Rule Types

#### Network Rules

Network rules control traffic based on:

- Source IP address
- Destination IP address
- Protocol
- Destination port

Examples:

- Allow Application Spoke to Shared Services on TCP 443
- Allow workload connectivity to approved DNS servers
- Allow required connectivity between regional workloads
- Deny unauthorized network communication

#### Application Rules

Application rules control outbound traffic based on:

- Source IP address
- Destination FQDN
- HTTP
- HTTPS

Examples:

- Allow workloads to approved application endpoints
- Allow access to required software repositories
- Restrict outbound access to approved websites and services

### Firewall Policies

Firewall rules are organized according to traffic requirements and security objectives.

Rules should follow the principle of least privilege.

Only required traffic should be permitted, and unnecessary connectivity should be restricted.

---

## 4. Network Security Groups

Network Security Groups provide additional traffic filtering at the subnet or network interface level.

NSGs are used to restrict traffic entering or leaving workload subnets.

### NSG Security Strategy

NSGs will be applied to workload subnets based on their role.

Examples include:

- Application subnet
- Web subnet
- Shared Services subnet
- Management subnet
- Private Endpoint subnet, where applicable

### Example Security Rules

| Source | Destination | Port | Protocol | Action |
|---|---|---|---|---|
| Approved management network | Management subnet | 22 / 3389 | TCP | Allow |
| Application subnet | Shared Services subnet | Required ports | TCP | Allow |
| Application subnet | Internet | Required traffic | TCP | Controlled |
| Unauthorized network | Workload subnet | Any | Any | Deny |
| Internet | Private workload subnet | Any | Any | Deny |

The exact NSG rules will depend on the workload requirements.

NSGs should not unnecessarily duplicate every Azure Firewall rule. They provide an additional security layer at the workload and subnet level.

---

## 5. Network Segmentation

The architecture separates workloads into dedicated Virtual Networks.

### Application Spoke

The Application Spoke hosts application workloads.

Connectivity is restricted based on application requirements.

The Application Spoke should not have unrestricted access to other regional networks or Shared Services.

### Shared Services Spoke

The Shared Services Spoke hosts common infrastructure and services used by workloads.

Access to Shared Services is controlled based on the specific service and required ports.

### Hub VNet

The Hub VNet provides centralized network infrastructure, including Azure Firewall and network connectivity services.

The Hub is used to control and inspect traffic routed through the centralized network security infrastructure.

---

## 6. Spoke-to-Spoke Security

Communication between Spokes is controlled through routing and security policies.

Application workloads should only communicate with Shared Services or other workloads when required.

Traffic between Spokes should follow the intended network path through the regional Hub and Azure Firewall where centralized inspection is required.

VNet Peering alone does not force traffic through Azure Firewall.

Therefore, User Defined Routes and the appropriate routing configuration must be used to direct traffic through the Firewall.

Network Security Groups provide additional restrictions to control the permitted communication between workloads.

---

## 7. Inter-Region Security

Communication between Region 1 and Region 2 is controlled through Global VNet Peering and routing configuration.

Inter-region traffic should be restricted to required communication flows.

Security controls include:

- Azure Firewall inspection
- Network Security Group restrictions
- Explicit routing configuration
- Restricted workload communication
- Return-path validation

Inter-region connectivity does not automatically provide transitive routing.

Traffic between Spokes located in different regions must follow an explicitly designed and validated routing path.

---

## 8. Internet Egress Security

Outbound Internet connectivity from workloads is controlled through the regional Azure Firewall.

User Defined Routes are configured where required to direct outbound traffic through Azure Firewall.

### Egress Security Principles

- Workloads should not have unrestricted Internet access.
- Outbound connectivity