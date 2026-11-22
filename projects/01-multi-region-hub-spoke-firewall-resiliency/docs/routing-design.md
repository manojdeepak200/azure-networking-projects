# Routing Design — Multi-Region Azure Hub-and-Spoke Network

## 1. Purpose

This document defines the routing architecture for the multi-region Azure Hub-and-Spoke network.

The routing design controls traffic flow between:

- Application Spokes
- Shared Services Spokes
- Regional Hub VNets
- Azure Firewalls
- Internet destinations
- Cross-region workloads

The architecture is designed to support centralized traffic inspection, network segmentation, connectivity control, resiliency, and network troubleshooting.

The routing model uses Azure system routes, VNet Peering, User Defined Routes (UDRs), and Azure Firewall to control traffic paths across the environment.

---

## 2. Network Topology

The architecture contains two Azure regions, each containing a Hub VNet, an Application Spoke, and a Shared Services Spoke.

### Region 1

| Network | Address Space |
|---|---|
| Hub VNet | `10.0.0.0/16` |
| Application Spoke | `10.1.0.0/16` |
| Shared Services Spoke | `10.2.0.0/16` |

### Region 2

| Network | Address Space |
|---|---|
| Hub VNet | `10.10.0.0/16` |
| Application Spoke | `10.11.0.0/16` |
| Shared Services Spoke | `10.12.0.0/16` |

### Regional Network Architecture

Each region follows the same network architecture:

- Spokes connect to their regional Hub VNet.
- Azure Firewall is deployed in the regional Hub.
- Spoke traffic requiring centralized inspection is routed through the regional Firewall.
- Regional Hubs are connected using Global VNet Peering.
- Inter-region communication is controlled through explicit routing and security policies.

---

## 3. Routing Components

The architecture uses the following routing components.

| Component | Purpose |
|---|---|
| Azure System Routes | Provide default platform routing behavior |
| VNet Peering | Connects directly peered Virtual Networks |
| User Defined Routes | Overrides default routing behavior for selected destinations |
| Azure Firewall | Inspects and filters network traffic |
| Route Tables | Define custom routing behavior |
| Effective Routes | Shows routes available to a network interface |
| Network Watcher | Provides network diagnostics and troubleshooting tools |

---

## 4. Routing Fundamentals

### 4.1 Azure System Routes

Azure automatically creates system routes for destinations such as:

- Virtual Network address spaces
- Peered Virtual Networks
- Internet
- Virtual Network Gateway
- Azure platform infrastructure

System routes provide the default routing behavior for Azure Virtual Networks.

However, system routes alone do not provide the centralized inspection architecture required by this project.

User Defined Routes will be used where explicit traffic steering is required.

### 4.2 User Defined Routes

User Defined Routes allow administrators to override Azure's default routing behavior for specific destination prefixes.

A UDR consists of:

- Destination address prefix
- Next-hop type
- Next-hop IP address, when applicable

Example:

| Destination | Next Hop Type | Next Hop |
|---|---|---|
| `0.0.0.0/0` | Virtual Appliance | Regional Firewall Private IP |
| `10.11.0.0/16` | Virtual Appliance | Regional Firewall Private IP |
| `10.12.0.0/16` | Virtual Appliance | Regional Firewall Private IP |

UDRs are used to direct selected traffic through Azure Firewall for inspection and security enforcement.

Route selection depends on the available system routes, user-defined routes, BGP routes, and longest-prefix matching.

### 4.3 Azure Firewall

Azure Firewall acts as a centralized network security and traffic inspection component.

Its role includes:

- Filtering network traffic
- Enforcing network and application rules
- Controlling Internet egress
- Inspecting traffic between selected network segments
- Logging permitted and denied traffic
- Supporting centralized security policies

Azure Firewall must have a valid route to the destination network, and the return path must be designed to maintain stateful traffic flow.

---

## 5. Routing Design Principles

### Principle 1 — Centralized Inspection

Traffic requiring centralized inspection should pass through the designated Azure Firewall.

Spoke subnet route tables will direct selected traffic to the regional Firewall.

### Principle 2 — Explicit Routing

Traffic paths must be explicitly designed.

The architecture will not rely on assumptions about VNet Peering, transit routing, or automatic failover.

### Principle 3 — Regional Preference

Workloads should use their local regional network infrastructure wherever appropriate.

Local routing reduces unnecessary cross-region traffic and simplifies regional traffic management.

### Principle 4 — Controlled Inter-Region Communication

Cross-region communication must be explicitly permitted and routed.

Inter-region traffic will follow defined network paths and security policies.

### Principle 5 — Return Path Awareness

Forward and return paths must be analyzed together.

Stateful network appliances such as Azure Firewall may experience connectivity problems when traffic follows inconsistent paths.

### Principle 6 — Failover Must Be Explicit

A secondary Firewall or an alternative route does not automatically guarantee failover.

The mechanism responsible for detecting failures and changing traffic paths must be explicitly designed.

---

## 6. Routing Architecture

### 6.1 Selected Baseline Architecture — Regional Hub-and-Spoke

The baseline architecture uses local Hub-and-Spoke routing within each region.

#### Region 1

```text
Application Spoke
       |
       v
    Hub VNet
       |
       v
 Azure Firewall 1
```

```text
Shared Services Spoke
       |
       v
    Hub VNet
       |
       v
 Azure Firewall 1
```

#### Region 2

```text
Application Spoke
       |
       v
    Hub VNet
       |
       v
 Azure Firewall 2
```

```text
Shared Services Spoke
       |
       v
    Hub VNet
       |
       v
 Azure Firewall 2
```

The regional Hub VNets are connected using Global VNet Peering.

### Advantages

- Simple regional architecture
- Clear regional security boundaries
- Centralized security enforcement
- Local traffic uses regional infrastructure
- Consistent architecture across both regions
- Easier troubleshooting and operational management

### Limitations

Global VNet Peering does not provide automatic transit routing.

A Spoke connected to Hub 1 cannot automatically use Hub 1 as a transit network to reach a network connected to Hub 2.

Additional routing configuration and traffic forwarding behavior are required for cross-region Spoke communication.

---

## 7. Alternative Routing Architectures

### 7.1 Spokes Connected to Both Regional Hubs

Each Spoke could establish peering with both regional Hub VNets.

#### Potential Topology

Application Spoke Region 1:

- Peering with Hub 1
- Peering with Hub 2

Application Spoke Region 2:

- Peering with Hub 1
- Peering with Hub 2

### Advantages

- Additional connectivity options
- Potential alternative paths
- More routing flexibility

### Risks

- Increased peering complexity
- More complicated route management
- Potential routing ambiguity
- Possible asymmetric routing
- Firewall inspection paths become more complex
- Increased operational overhead

This architecture is not the baseline design because it increases complexity and requires careful routing and security planning.

### 7.2 Direct Spoke-to-Spoke Connectivity

Selected Spokes may be directly peered where specific connectivity requirements justify it.

### Advantages

- Direct connectivity
- Fewer intermediate routing dependencies
- Potentially simpler paths for selected workloads

### Disadvantages

- Reduced centralized network control
- Increased peering complexity as the number of Spokes grows
- Potential bypass of centralized security inspection
- More difficult management of security boundaries

Direct Spoke-to-Spoke Peering is not the default architecture.

### 7.3 Dynamic Routing with Azure Route Server

An advanced architecture may use Azure Route Server with BGP-capable Network Virtual Appliances.

Potential components:

- Azure Route Server
- BGP-capable Network Virtual Appliance
- BGP peers
- Advertised prefixes
- Route withdrawal
- Dynamic route convergence

### Advantages

- Dynamic route exchange
- Advanced routing capabilities
- Potential automated path changes
- More realistic enterprise routing scenarios

### Limitations

- Increased complexity
- Additional infrastructure cost
- Requires BGP knowledge
- Requires an appropriate NVA architecture

Dynamic routing is an advanced extension to the baseline architecture.

---

## 8. Baseline Routing Behavior

### 8.1 Application Spoke Region 1 — Internet Egress

Expected traffic path:

```text
Application VM Region 1
          |
          v
Application Spoke
          |
          v
     Route Table
          |
          v
 Azure Firewall Region 1
          |
          v
       Internet
```

The default route is configured to direct Internet-bound traffic through the regional Azure Firewall.

| Property | Value |
|---|---|
| Destination Prefix | `0.0.0.0/0` |
| Next Hop Type | Virtual Appliance |
| Next Hop | Azure Firewall Region 1 Private IP |

This route provides centralized Internet egress inspection.

### 8.2 Application Spoke Region 2 — Internet Egress

Expected traffic path:

```text
Application VM Region 2
          |
          v
Application Spoke
          |
          v
     Route Table
          |
          v
 Azure Firewall Region 2
          |
          v
       Internet
```

The same regional routing principle applies.

| Property | Value |
|---|---|
| Destination Prefix | `0.0.0.0/0` |
| Next Hop Type | Virtual Appliance |
| Next Hop | Azure Firewall Region 2 Private IP |

Each region uses its own Firewall for local Internet egress.

### 8.3 Application Spoke to Shared Services Spoke

Expected logical traffic path:

```text
Application Spoke
       |
       v
Regional Firewall
       |
       v
Shared Services Spoke
```

Traffic between Application and Shared Services networks is routed through the regional Firewall when centralized inspection is required.

The route configuration, peering settings, and Firewall forwarding behavior must support the intended path.

### 8.4 Region 1 to Region 2

The desired logical traffic path is:

```text
Region 1 Workload
       |
       v
Regional Firewall 1
       |
       v
Inter-Region Connectivity
       |
       v
Regional Firewall 2
       |
       v
Region 2 Workload
```

This represents the intended logical inspection path.

However, Azure VNet Peering is not transitive.

The following peering relationships alone do not automatically establish Spoke-to-Spoke transit connectivity:

- Spoke 1 is peered with Hub 1
- Hub 1 is peered with Hub 2
- Hub 2 is peered with Spoke 2

The architecture must explicitly configure and support traffic forwarding through the regional network appliances.

This includes appropriate peering forwarding settings, routing configuration, Firewall routes, and return-path design.

---

## 9. Route Table Design

Route tables are associated with workload subnets to control traffic flow.

The baseline route table design uses regional Azure Firewall private IP addresses as the next hop for traffic requiring inspection.

### 9.1 Region 1 Application Spoke

| Route Name | Destination Prefix | Next Hop Type | Next Hop |
|---|---|---|---|
| DefaultRoute | `0.0.0.0/0` | Virtual Appliance | Firewall 1 IP |
| Region2-App | `10.11.0.0/16` | Virtual Appliance | Firewall 1 IP |
| Region2-Shared | `10.12.0.0/16` | Virtual Appliance | Firewall 1 IP |

### 9.2 Region 1 Shared Services Spoke

| Route Name | Destination Prefix | Next Hop Type | Next Hop |
|---|---|---|---|
| DefaultRoute | `0.0.0.0/0` | Virtual Appliance | Firewall 1 IP |
| Region2-App | `10.11.0.0/16` | Virtual Appliance | Firewall 1 IP |
| Region2-Shared | `10.12.0.0/16` | Virtual Appliance | Firewall 1 IP |

### 9.3 Region 2 Application Spoke

| Route Name | Destination Prefix | Next Hop Type | Next Hop |
|---|---|---|---|
| DefaultRoute | `0.0.0.0/0` | Virtual Appliance | Firewall 2 IP |
| Region1-App | `10.1.0.0/16` | Virtual Appliance | Firewall 2 IP |
| Region1-Shared | `10.2.0.0/16` | Virtual Appliance | Firewall 2 IP |

### 9.4 Region 2 Shared Services Spoke

| Route Name | Destination Prefix | Next Hop Type | Next Hop |
|---|---|---|---|
| DefaultRoute | `0.0.0.0/0` | Virtual Appliance | Firewall 2 IP |
| Region1-App | `10.1.0.0/16` | Virtual Appliance | Firewall 2 IP |
| Region1-Shared | `10.2.0.0/16` | Virtual Appliance | Firewall 2 IP |

### Route Design Considerations

These route entries represent the intended traffic steering behavior.

The complete routing path requires:

- Spoke-to-Hub VNet Peering
- Appropriate peering forwarding settings
- Hub-to-Hub Global VNet Peering
- Firewall forwarding capability
- Valid routes from the Firewall to destination networks
- Correct return-path routing
- Security rules allowing the intended traffic

A UDR pointing to a Firewall does not independently establish connectivity to the destination.

The Firewall must also be able to forward the traffic toward the destination network.

---

## 10. Route Selection

Azure route selection is based on the available routes and their matching prefixes.

Important concepts include:

- Longest Prefix Match
- Azure System Routes
- User Defined Routes
- BGP Routes
- Route Priority
- Effective Routes
- Next Hop

### Example

| Route | Prefix |
|---|---|
| Route A | `10.0.0.0/8` |
| Route B | `10.1.0.0/16` |
| Route C | `10.1.1.0/24` |

For traffic destined to:

`10.1.1.10`

Azure selects the most specific matching route.

In this example, `10.1.1.0/24` is the most specific prefix.

This is known as Longest Prefix Match.

The project includes practical demonstrations of route selection, route overrides, and effective route analysis.

---

## 11. Failover Design

### 11.1 Failure Scenario

The preferred regional Firewall or network path becomes unavailable.

The architecture must determine:

- How the failure is detected
- Which traffic is affected
- Whether an alternative route exists
- How the traffic path changes
- Whether existing sessions are interrupted
- How traffic recovers
- Whether the recovery mechanism is automatic or manual

### 11.2 Static UDR Limitations

UDRs are static routing configurations.

They do not automatically:

- Detect Firewall failure
- Withdraw a route
- Select an alternate Firewall
- Perform dynamic routing convergence
- Guarantee network path failover

A failover mechanism must be explicitly implemented.

### 11.3 Manual Failover

An administrator modifies route table entries to direct traffic through an alternative path.

Manual failover is the baseline demonstration because the route change and its effect can be observed directly.

### 11.4 Automated Route Modification

A monitoring mechanism detects a failure and updates route tables through automation.

Potential technologies include:

- Azure Monitor Alerts
- Azure Functions
- Azure Automation
- PowerShell
- Azure CLI
- Azure REST API

The automation must include failure detection, decision logic, route modification, and recovery handling.

### 11.5 Dynamic Routing

An advanced design may use BGP and Azure Route Server with compatible Network Virtual Appliances.

This can support dynamic route advertisement and withdrawal, subject to the selected network architecture.

Dynamic routing is an advanced extension of the project.

---

## 12. Asymmetric Routing

Asymmetric routing occurs when forward and return traffic follow different network paths.

### Forward Path

```text
VM Region 1
     |
     v
 Firewall 1
     |
     v
 Firewall 2
     |
     v
VM Region 2
```

### Return Path

```text
VM Region 2
     |
     v
 Firewall 2
     |
     v
Alternative Network Path
     |
     v
VM Region 1
```

### Potential Symptoms

- TCP connections fail
- Connections establish and then terminate
- Firewall session state does not match
- Intermittent connectivity
- Different behavior depending on traffic direction

### Investigation Tools

- Effective Routes
- Network Watcher Next Hop
- Connection Troubleshoot
- Azure Firewall logs
- Packet capture
- Network topology analysis

Forward and return paths must be analyzed together when investigating stateful firewall connectivity.

---

## 13. Routing Troubleshooting Methodology

Routing issues will be investigated using a structured troubleshooting process.

### Step 1 — Confirm the Source and Destination

Identify:

- Source VM
- Source subnet
- Source IP
- Destination IP
- Destination subnet
- Expected network path

### Step 2 — Review the Route Table

Confirm:

- Correct route table
- Correct subnet association
- Destination prefix
- Next-hop type
- Next-hop IP address

### Step 3 — Review Effective Routes

Use the VM's network interface to inspect effective routes.

Confirm:

- Expected route exists
- Route prefix is correct
- Next-hop type is correct
- Route selection matches the intended path

### Step 4 — Validate the Next Hop

Use Network Watcher Next Hop to determine where traffic is expected to be forwarded.

### Step 5 — Check Network Security

Review:

- Network Security Groups
- Azure Firewall rules
- Firewall network rules
- Firewall application rules
- Destination service configuration

### Step 6 — Check Peering Configuration

Review:

- Peering status
- Address space configuration
- Allow forwarded traffic
- Gateway transit settings, where applicable
- Remote gateway settings, where applicable

### Step 7 — Analyze the Return Path

Confirm that the destination can return traffic through a compatible network path.

### Step 8 — Review Logs

Review:

- Azure Firewall logs
- Network Watcher diagnostics
- VM operating system logs
- Application logs

---

## 14. Routing Experiments

### Experiment 1 — Default Route to Firewall

Configure a UDR for Internet traffic.

Investigate:

- Effective route
- Next Hop
- Internet connectivity
- Firewall logs
- Traffic inspection

### Experiment 2 — Firewall Bypass

Modify or remove a route.

Investigate whether traffic follows an alternative path and whether centralized inspection is bypassed.

### Experiment 3 — Incorrect Next Hop

Configure an incorrect next-hop IP.

Observe:

- Connectivity failure
- Effective route
- Next Hop diagnostics
- Troubleshooting results

### Experiment 4 — Cross-Region Routing

Configure routing toward the opposite region.

Investigate:

- VNet Peering connectivity
- Traffic forwarding
- Firewall path
- Destination reachability
- Return path

### Experiment 5 — Asymmetric Routing

Create a scenario where forward and return traffic follow different network paths.

Investigate:

- Firewall session behavior
- TCP connectivity
- Route selection
- Logs
- Packet capture

### Experiment 6 — Firewall Failure

Simulate a Firewall or network path failure.

Investigate:

- Failure detection
- Connectivity impact
- Route behavior
- Failover mechanism
- Recovery

---

## 15. Important Architecture Limitations

The architecture does not assume that:

- VNet Peering automatically provides transit routing.
- Azure Firewall automatically provides route failover.
- Two Firewalls automatically create an active-active routing architecture.
- UDRs dynamically change during failures.
- Application failover automatically changes private network routing.
- Connectivity automatically means traffic follows the intended path.
- Forward and return traffic automatically follow the same route.
- A route pointing to a Firewall automatically guarantees destination reachability.

Routing behavior depends on the complete network configuration, including route tables, peering configuration, traffic forwarding, Firewall routing, security policies, and return paths.

---

## 16. Routing Design Summary

The routing architecture provides a consistent regional Hub-and-Spoke design across two Azure regions.

The design includes:

- Regional Hub-and-Spoke routing
- Centralized Azure Firewall inspection
- Regional Internet egress
- Explicit User Defined Routes
- Global VNet Peering between regional Hubs
- Controlled cross-region traffic
- Route selection analysis
- Effective Route troubleshooting
- Next Hop investigation
- Asymmetric routing analysis
- Manual failover scenarios
- Automated route modification scenarios
- Advanced dynamic routing extensions

The routing architecture is designed to support centralized inspection, regional traffic control, network segmentation, and structured troubleshooting across the multi-region environment.

---

## 17. Implementation Approach

The routing architecture will be implemented progressively.

The implementation sequence is:

1. Deploy regional Hub VNets.
2. Deploy regional Spoke VNets.
3. Establish Hub-to-Spoke VNet Peering.
4. Establish Global VNet Peering between regional Hubs.
5. Deploy Azure Firewall in each Hub.
6. Configure route tables.
7. Associate route tables with selected Spoke subnets.
8. Configure Firewall policies.
9. Test regional Internet egress.
10. Test inter-Spoke traffic.
11. Test cross-region connectivity.
12. Investigate routing failures and asymmetric routing.
13. Implement manual failover.
14. Explore automated route modification.
15. Document observed routing behavior and troubleshooting results.
