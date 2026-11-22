# Traffic Flow Matrix — Multi-Region Azure Network

## 1. Purpose

This document defines the network communication requirements within the multi-region Azure Hub-and-Spoke architecture.

The traffic flow matrix identifies:

- Source and destination networks
- Required protocols and ports
- Purpose of network communication
- Expected traffic paths
- Security controls
- Traffic inspection requirements
- Communication that should be permitted
- Communication that should be denied
- Methods used to troubleshoot network connectivity

The matrix serves as the foundation for:

- Network Security Group rules
- Azure Firewall rules
- User Defined Routes
- Network segmentation
- Connectivity testing
- Network troubleshooting
- Security policy enforcement
- Traffic monitoring

The objective is to ensure that network communication is explicitly defined rather than allowing unrestricted connectivity between workloads.

---

## 2. Network Environments

### Region 1

| Network | Address Space | Purpose |
|---|---|---|
| Hub VNet | `10.0.0.0/16` | Central networking |
| Application Spoke | `10.1.0.0/16` | Application workloads |
| Shared Services Spoke | `10.2.0.0/16` | Shared infrastructure |

### Region 2

| Network | Address Space | Purpose |
|---|---|---|
| Hub VNet | `10.10.0.0/16` | Central networking |
| Application Spoke | `10.11.0.0/16` | Application workloads |
| Shared Services Spoke | `10.12.0.0/16` | Shared infrastructure |

Each region contains a centralized Azure Firewall deployed in the Hub VNet.

Regional Hubs are connected through Global VNet Peering.

---

## 3. Traffic Flow Requirements

## Flow 1 — Internet Users to Application

| Attribute | Requirement |
|---|---|
| Source | Internet users |
| Destination | Application workloads in Region 1 or Region 2 |
| Protocol | HTTPS |
| Port | TCP 443 |
| Purpose | Access the application |
| Traffic Direction | Inbound |
| Security Control | Global application entry point, WAF, and backend security controls |
| Expected Behavior | Traffic reaches a healthy application endpoint |
| Monitoring | Application health, access logs, and security logs |

### Expected Logical Architecture

```text
Internet Users
      |
      v
Azure Front Door
      |
      v
Global Application Entry Point
      |
      v
Healthy Regional Backend
      |
      v
Application Workload
```

### Design Considerations

Azure Front Door is the global application entry point for the architecture.

The design must account for:

- Application hosting model
- Backend connectivity
- Health probes
- TLS termination
- WAF policies
- Public versus private backend connectivity
- Backend access restrictions
- Regional application health
- Application-level failover

The backend architecture must ensure that application workloads are not exposed directly to unrestricted Internet traffic.

Where supported by the selected architecture, backend access should be restricted to the approved application entry point.

---

## Flow 2 — Application Spoke to Internet

| Attribute | Requirement |
|---|---|
| Source | Application Spoke |
| Destination | Internet |
| Protocol | HTTP / HTTPS |
| Ports | TCP 80 / TCP 443 |
| Purpose | Application outbound connectivity |
| Traffic Direction | Outbound |
| Expected Path | Spoke → Regional Hub → Azure Firewall → Internet |
| Security Control | Azure Firewall |
| Expected Behavior | Only approved outbound traffic is permitted |
| Monitoring | Firewall logs and external connectivity tests |

### Design Requirement

Application workloads should not have uncontrolled direct Internet access.

Outbound traffic should be routed through Azure Firewall wherever centralized inspection is required.

### Security Considerations

Outbound Internet access should be restricted based on application requirements.

Depending on the workload, outbound access may be controlled through:

- Azure Firewall network rules
- Azure Firewall application rules
- Destination FQDN restrictions
- NSGs
- Route tables
- Application-specific security policies

Unrestricted outbound connectivity should not be granted unless a workload has a documented requirement.

---

## Flow 3 — Application Spoke to Shared Services Spoke (Same Region)

| Attribute | Requirement |
|---|---|
| Source | Application Spoke Region 1 |
| Destination | Shared Services Spoke Region 1 |
| Protocol | Service-specific |
| Ports | Application-specific |
| Purpose | Access shared infrastructure |
| Traffic Direction | Internal |
| Expected Path | Application Spoke → Regional Hub / Firewall → Shared Services Spoke |
| Security Control | Azure Firewall and NSG |
| Expected Behavior | Only approved service communication is permitted |
| Monitoring | Connectivity tests and Firewall logs |

### Shared Services Examples

Shared services may include:

- DNS
- Monitoring
- Management
- Automation
- Internal application services
- Shared infrastructure components

### Security Requirements

Communication between Application and Shared Services networks should be restricted to required services.

The architecture should not allow unrestricted access from application workloads to all shared-services resources.

Rules should be based on:

- Source subnet
- Destination subnet
- Application requirement
- Protocol
- Port
- Security policy

---

## Flow 4 — Application Spoke Region 1 to Application Spoke Region 2

| Attribute | Requirement |
|---|---|
| Source | Application Spoke Region 1 |
| Destination | Application Spoke Region 2 |
| Protocol | Application-specific |
| Ports | Application-specific |
| Purpose | Cross-region application communication |
| Traffic Direction | Inter-region |
| Expected Path | Regional Network → Hub → Inter-region Connectivity → Regional Hub → Destination |
| Security Control | Regional Azure Firewalls |
| Expected Behavior | Cross-region communication is controlled and inspected |
| Monitoring | Route diagnostics, connectivity testing, and Firewall logs |

### Design Considerations

This flow is used to investigate:

- Global VNet Peering
- Route tables
- Cross-region traffic
- Firewall inspection
- Return traffic paths
- Asymmetric routing
- Network forwarding behavior

VNet Peering does not provide automatic transitive routing.

The required cross-region traffic path must be explicitly supported through the complete peering, routing, forwarding, Firewall, and return-path configuration.

Connectivity must not be assumed simply because the regional Hubs are peered.

---

## Flow 5 — Shared Services Region 1 to Shared Services Region 2

| Attribute | Requirement |
|---|---|
| Source | Shared Services Spoke Region 1 |
| Destination | Shared Services Spoke Region 2 |
| Protocol | Service-specific |
| Ports | Service-specific |
| Purpose | Cross-region shared-services communication |
| Traffic Direction | Inter-region |
| Expected Path | Regional Hub → Inter-region Connectivity → Regional Hub |
| Security Control | Azure Firewall |
| Expected Behavior | Controlled cross-region connectivity |
| Monitoring | Network connectivity and route diagnostics |

### Design Considerations

Cross-region shared-services communication should be limited to services that require regional synchronization, replication, administration, or inter-region dependencies.

The traffic must follow the defined inter-region routing architecture.

Security rules should explicitly identify the required source, destination, protocol, and port.

---

## Flow 6 — Management Access to Virtual Machines

| Attribute | Requirement |
|---|---|
| Source | Authorized management environment |
| Destination | Azure Virtual Machines |
| Protocol | SSH / RDP |
| Ports | TCP 22 / TCP 3389 |
| Purpose | VM administration |
| Traffic Direction | Management |
| Security Control | Azure Bastion, NSG, Firewall, or private connectivity |
| Expected Behavior | Only authorized administrators can access workloads |
| Monitoring | Access logs and connectivity tests |

### Security Requirements

Direct public IP access to workload Virtual Machines should be avoided wherever possible.

Management access should use controlled administrative connectivity.

Potential access methods include:

- Azure Bastion
- Private connectivity
- VPN connectivity
- Controlled administrative networks
- Just-in-time access mechanisms where applicable

Management access should be restricted to authorized sources.

### Recommended Security Policy

| Source | Destination | Port | Policy |
|---|---|---|---|
| Authorized management network | Management VM | TCP 22 | Allow when required |
| Authorized management network | Windows VM | TCP 3389 | Allow when required |
| Internet | Workload VM management ports | TCP 22 / 3389 | Deny |
| Untrusted workload | Management network | Management ports | Deny |

---

## Flow 7 — Application Spoke to DNS Services

| Attribute | Requirement |
|---|---|
| Source | Application Spokes |
| Destination | DNS Server / DNS Service |
| Protocol | UDP / TCP |
| Port | 53 |
| Purpose | Name resolution |
| Traffic Direction | Internal or platform DNS |
| Security Control | NSG and Firewall where applicable |
| Expected Behavior | Workloads resolve required internal and external names |
| Monitoring | DNS query testing and connectivity diagnostics |

### DNS Design Considerations

DNS is a critical dependency for application connectivity.

DNS failures can cause applications to appear unavailable even when the underlying network connectivity is working.

The architecture must consider:

- Azure-provided DNS
- Custom DNS servers
- Azure Private DNS Zones
- DNS forwarding
- DNS Private Resolver
- Cross-region name resolution
- Private Endpoint name resolution
- Conditional forwarding

The required DNS architecture depends on the workload and connectivity design.

### Troubleshooting Methods

Common DNS troubleshooting tools include:

```bash
nslookup
```

```bash
dig
```

Additional investigation may include:

- DNS server reachability
- DNS configuration on the VM
- DNS forwarding configuration
- Private DNS Zone links
- DNS records
- Conditional forwarding
- Firewall rules
- Network routing

---

## Flow 8 — Network Workloads to Monitoring Services

| Attribute | Requirement |
|---|---|
| Source | Azure workloads and network resources |
| Destination | Azure Monitor / Log Analytics |
| Protocol | HTTPS |
| Port | TCP 443 |
| Purpose | Monitoring and logging |
| Traffic Direction | Outbound |
| Security Control | Azure service connectivity and Firewall rules where applicable |
| Expected Behavior | Monitoring and diagnostic data reaches the monitoring platform |
| Monitoring | Log ingestion and telemetry availability |

### Design Considerations

Monitoring and logging are critical operational requirements.

Workloads and network resources must be able to send required telemetry to their monitoring destinations.

The architecture must account for:

- Azure Monitor
- Log Analytics Workspace
- Azure Monitor Agent
- Diagnostic Settings
- Azure Firewall logs
- VM monitoring
- Network monitoring
- Required Azure service endpoints
- Firewall restrictions
- Private connectivity requirements where applicable

Where Internet egress is restricted, required monitoring destinations must be explicitly allowed through the network security architecture.

---

## Flow 9 — Cross-Region Management Connectivity

| Attribute | Requirement |
|---|---|
| Source | Management Spoke Region 1 |
| Destination | Management Spoke Region 2 |
| Protocol | SSH / RDP / HTTPS |
| Ports | Service-specific |
| Purpose | Cross-region administration |
| Traffic Direction | Inter-region |
| Expected Path | Regional Hub → Inter-region Network → Regional Hub |
| Security Control | Azure Firewall and NSG |
| Expected Behavior | Controlled administrative connectivity |
| Monitoring | Connectivity testing and route diagnostics |

### Security Requirements

Cross-region management connectivity should be limited to authorized management systems and required administrative services.

Management traffic must not be treated as unrestricted inter-region communication.

---

## 4. Traffic That Should Be Denied

The architecture should not permit unrestricted communication between all networks.

The following traffic should be denied unless explicitly required by an approved workload or architecture requirement.

| Source | Destination | Reason |
|---|---|---|
| Internet | Private workload IPs | Prevent unauthorized inbound access |
| Application Spoke | Unapproved Shared Services | Restrict lateral movement |
| Application Spoke | Unapproved Application Spoke | Enforce segmentation |
| Untrusted workload | Management network | Protect management infrastructure |
| Any workload | Unapproved Internet destinations | Control outbound access |
| Unauthorized source | VM management ports | Prevent unauthorized administration |
| Unapproved region | Regional management services | Restrict cross-region administrative access |

### Deny-by-Default Principle

Network communication should be permitted only when a business or technical requirement exists.

The architecture should follow a least-privilege approach:

- Permit required communication.
- Deny unnecessary communication.
- Restrict source and destination.
- Restrict protocol and port.
- Log important denied traffic.
- Review security rules periodically.

---

## 5. Traffic Inspection Requirements

The following traffic categories require centralized Firewall inspection where mandated by the network architecture.

| Traffic Type | Inspection Requirement |
|---|---|
| Internet outbound | Required |
| Cross-region application traffic | Required |
| Cross-region shared-services traffic | Required |
| Spoke-to-Spoke traffic | Required where centralized inspection is mandated |
| Management traffic | Based on security architecture |
| DNS traffic | Based on DNS architecture |
| Monitoring traffic | Based on service connectivity requirements |

### Inspection Objectives

Firewall inspection is intended to provide:

- Centralized security enforcement
- Network traffic filtering
- Logging
- Threat detection capabilities where configured
- Control over Internet egress
- Control over selected internal traffic
- Visibility into permitted and denied flows

Not every Azure service communication path necessarily passes through Azure Firewall.

The routing architecture determines whether traffic is inspected.

---

## 6. Routing Expectations

The routing architecture must support the intended traffic paths.

| Traffic | Intended Routing |
|---|---|
| Application Spoke → Internet | Regional Firewall |
| Shared Services → Internet | Regional Firewall |
| Application Spoke → Shared Services | Controlled regional Hub path |
| Region 1 → Region 2 | Controlled inter-region path |
| Management → Workloads | Controlled management path |
| Internet → Application | Global application entry point |
| Workloads → DNS | Defined DNS architecture |
| Workloads → Monitoring | Approved Azure service connectivity |
| Return Traffic | Compatible return path |

### Routing Requirements

- Route tables must be associated with the correct subnets.
- UDR destination prefixes must be correct.
- Next-hop IP addresses must be valid.
- Firewalls must have routes to destination networks.
- Peering must support the required forwarding behavior.
- Return traffic must follow a compatible route.
- Connectivity must not be confused with successful traffic inspection.

---

## 7. Asymmetric Routing Consideration

Asymmetric routing occurs when traffic follows different paths in each direction.

### Forward Path

```text
Application Region 1
10.1.1.10
      |
      v
Firewall Region 1
      |
      v
Firewall Region 2
      |
      v
Application Region 2
10.11.1.10
```

### Return Path

```text
Application Region 2
10.11.1.10
      |
      v
Firewall Region 2
      |
      v
Direct or Alternative Route
      |
      v
Application Region 1
10.1.1.10
```

Different forward and return paths can cause connectivity problems when stateful network appliances do not observe the expected traffic flow.

### Potential Symptoms

- TCP connections fail
- Connections establish and then terminate
- Firewall session state does not match
- Intermittent connectivity
- Different behavior depending on traffic direction

### Investigation Areas

- Route selection
- UDR configuration
- Firewall state
- Return traffic paths
- Effective Routes
- Network Watcher Next Hop
- Connection Troubleshoot
- Packet capture
- Firewall logs

---

## 8. Connectivity and Traffic Verification

Each traffic flow must be evaluated using appropriate network and application troubleshooting tools.

### Available Methods

- Ping where supported
- TCP connectivity testing
- SSH
- RDP
- HTTP / HTTPS requests
- `curl`
- `Test-NetConnection`
- `nslookup`
- `dig`
- Azure Network Watcher
- Effective Routes
- Next Hop
- IP Flow Verify
- Connection Troubleshoot
- Azure Firewall Logs
- Log Analytics queries

### Verification Requirements

Successful connectivity alone is insufficient.

Verification should confirm:

- The traffic followed the intended route.
- The expected security controls were applied.
- The correct Firewall inspected the traffic where required.
- The destination was reachable.
- The return path was compatible.
- Logs were generated where expected.
- Denied traffic was blocked according to policy.
- The observed behavior matched the defined traffic requirement.

---

## 9. Traffic Flow Summary

| Flow | Source | Destination | Protocol / Port | Security Control |
|---|---|---|---|---|
| Internet to Application | Internet | Application | HTTPS / TCP 443 | Front Door / WAF / Backend Security |
| Application to Internet | Application Spoke | Internet | HTTP / HTTPS | Azure Firewall |
| Application to Shared Services | Application Spoke | Shared Services Spoke | Service-specific | Firewall / NSG |
| Region 1 to Region 2 Application | Application Spoke R1 | Application Spoke R2 | Application-specific | Regional Firewalls |
| Shared Services Cross-Region | Shared Services R1 | Shared Services R2 | Service-specific | Azure Firewall |
| Management to VM | Management Network | VM | SSH 22 / RDP 3389 | Bastion / NSG / Firewall |
| Application to DNS | Application Spoke | DNS Service | UDP/TCP 53 | NSG / Firewall as applicable |
| Workloads to Monitoring | Workloads | Azure Monitor / Log Analytics | HTTPS 443 | Firewall / Service Connectivity |
| Cross-Region Management | Management R1 | Management R2 | Service-specific | Firewall / NSG |

---

## 10. Traffic Flow Design Summary

The traffic flow design establishes communication requirements across the multi-region Azure network.

The architecture includes:

- Controlled Internet access to applications
- Centralized outbound Internet inspection
- Regional Application-to-Shared Services communication
- Controlled cross-region application communication
- Cross-region shared-services connectivity
- Restricted management access
- DNS connectivity
- Monitoring and logging connectivity
- Network segmentation
- Deny-by-default security principles
- Explicit routing requirements
- Return-path awareness
- Firewall inspection requirements
- Network troubleshooting and traffic verification

The traffic flow matrix provides the foundation for implementing NSGs, Azure Firewall policies, route tables, connectivity tests, and network troubleshooting scenarios.

The implementation must ensure that permitted traffic follows the intended network path and that unauthorized communication is blocked.
