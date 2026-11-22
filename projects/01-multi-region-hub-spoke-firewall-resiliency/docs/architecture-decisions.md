# Architecture Decision Records (ADR)

## Project

Multi-Region Azure Hub-and-Spoke Network with Firewall Resiliency

---

# ADR-001 — Multi-Region Network Topology

## Status

**Accepted**


---

## 1. Context

The project requires a network architecture that supports workloads distributed across two Azure regions.

The architecture must provide:

- Regional workload isolation
- Centralized network security
- Controlled inter-region connectivity
- Internet egress inspection
- Network segmentation
- Resiliency
- Routing control
- Troubleshooting capabilities
- Future scalability

Several network topology options are available.

The architecture must balance security, scalability, operational complexity, cost, and ease of troubleshooting.

---

## 2. Decision Drivers

The following factors influenced the architecture selection.

| Decision Driver | Importance |
|---|---|
| Security | High |
| Centralized traffic inspection | High |
| Network segmentation | High |
| Regional isolation | High |
| Scalability | High |
| Resiliency | High |
| Troubleshooting simplicity | High |
| Operational complexity | Medium |
| Cost | High |
| Learning value | High |

---

## 3. Options Considered

### Option 1 — Flat Network Architecture

All workloads are deployed into a small number of directly connected VNets with minimal centralized network infrastructure.

#### Advantages

- Simple to deploy
- Easy to understand initially
- Fewer network components
- Lower infrastructure cost

#### Disadvantages

- Limited network segmentation
- Difficult to enforce centralized security
- Poor scalability for large environments
- Less suitable for enterprise network architecture
- Troubleshooting becomes difficult as the environment grows

#### Decision

**Rejected.**

This architecture does not adequately demonstrate enterprise network segmentation, centralized inspection, or advanced routing.

---

### Option 2 — Single Hub-and-Spoke Architecture

One central Hub VNet connects multiple Spoke VNets.

The Hub provides shared network services and centralized traffic inspection.

#### Advantages

- Centralized security
- Clear network segmentation
- Easier network administration
- Scalable compared with a flat network
- Suitable for enterprise environments

#### Disadvantages

- Central Hub may become a dependency
- Regional resiliency is limited
- A single regional Hub does not address multi-region requirements
- Cross-region architecture is not covered

#### Decision

**Rejected as the final architecture.**

This topology forms the foundation of the regional architecture.

---

### Option 3 — Multi-Region Hub-and-Spoke

Each Azure region contains:

- One Hub VNet
- One Azure Firewall
- Multiple Spoke VNets
- Regional routing infrastructure

The regional Hubs are connected using Global VNet Peering.

#### Advantages

- Regional network isolation
- Centralized security in each region
- Regional traffic inspection
- Improved scalability
- Supports multi-region workload deployment
- Enables investigation of inter-region connectivity
- Provides a foundation for resiliency

#### Disadvantages

- More infrastructure
- Higher cost
- Increased routing complexity
- Cross-region routing requires careful design
- Failover is not automatically provided by the topology
- Potential asymmetric routing

#### Decision

**Selected.**

The multi-region Hub-and-Spoke architecture is the foundation of this project.

---

## 4. Selected Architecture

The project will use a multi-region Hub-and-Spoke network topology.

Each region will contain a dedicated Hub VNet and Azure Firewall. Application and shared-services workloads will be deployed into separate Spoke VNets.

The regional Hubs will be connected using Global VNet Peering.

This architecture provides regional isolation, centralized security, controlled connectivity, and a foundation for network resiliency experiments.

---

# ADR-002 — Regional Spoke Connectivity

## Status

**Accepted**

---

## 1. Context

The project contains multiple Spoke VNets across two regions.

Several connectivity models are possible. The selected model must provide clear regional boundaries, centralized traffic inspection, manageable routing, and controlled inter-region connectivity.

---

## 2. Options Considered

### Option A — Each Spoke Connects Only to Its Local Hub

Example:

```text
Region 1

Application Spoke 1 ── Hub 1
Shared Services 1 ──── Hub 1


Region 2

Application Spoke 2 ── Hub 2
Shared Services 2 ──── Hub 2
```

Hub 1 and Hub 2 are connected through Global VNet Peering.

#### Advantages

- Simple regional topology
- Easier to manage
- Clear regional boundaries
- Reduced peering complexity
- Easier to troubleshoot local connectivity

#### Disadvantages

- Cross-region spoke connectivity requires additional routing design
- VNet Peering does not provide automatic transitive routing
- Additional routing and forwarding configuration is required

---

### Option B — Each Spoke Connects to Both Hubs

Example:

```text
Application Spoke 1

       |       |
       |       |
     Hub 1   Hub 2


Application Spoke 2

       |       |
       |       |
     Hub 1   Hub 2
```

#### Advantages

- Multiple connectivity paths
- More flexibility in routing
- Additional possible paths during failures

#### Disadvantages

- Increased peering complexity
- More complicated route management
- Increased risk of asymmetric routing
- More difficult centralized traffic inspection
- More complex troubleshooting

---

### Option C — Direct Spoke-to-Spoke Peering

Selected Spokes establish direct peering connections.

#### Advantages

- Direct communication
- Potentially simpler path for selected traffic
- Fewer routing dependencies for some workloads

#### Disadvantages

- Peering complexity increases
- Centralized inspection may be bypassed
- Difficult to scale as the number of Spokes grows
- Less consistent security architecture

---

## 3. Decision

The project uses the following connectivity model:

- Each Spoke connects to its regional Hub.
- Hub 1 and Hub 2 connect using Global VNet Peering.
- Direct Spoke-to-Spoke peering is not used as the default connectivity model.
- Cross-region Spoke communication uses explicitly designed routing and network forwarding.
- Centralized traffic inspection is maintained through the regional network security architecture.

---

## 4. Connectivity Design Principles

The architecture follows these principles:

- Regional Spokes connect to their local Hub.
- Regional traffic is routed through the regional network security infrastructure when inspection is required.
- Cross-region traffic uses explicitly defined routing paths.
- VNet Peering is not treated as a transitive routing mechanism.
- Forwarding behavior and routing decisions are explicitly configured.
- Return traffic follows a compatible path to avoid asymmetric routing.
- Network paths are designed to maintain security inspection requirements.

---

# ADR-003 — Centralized Network Security

## Status

**Accepted**

---

## 1. Context

Workloads require Internet access and communication with other internal networks.

Direct connectivity without centralized inspection may result in inconsistent security enforcement.

The architecture requires centralized security controls and network-level visibility.

---

## 2. Options Considered

### Option A — NSGs Only

Use Network Security Groups to control traffic.

#### Advantages

- Lower cost
- Easy to configure
- Native Azure functionality
- Provides subnet and NIC-level controls

#### Disadvantages

- Limited centralized inspection
- Does not provide full network firewall capabilities
- Difficult to manage centralized outbound policies
- Less visibility into application-layer traffic

---

### Option B — Azure Firewall in Each Region

Deploy Azure Firewall in each regional Hub.

#### Advantages

- Centralized regional security
- Network and application rules
- Firewall logging
- Threat intelligence capabilities
- Controlled outbound connectivity
- Supports Hub-and-Spoke architecture
- Suitable for enterprise network security

#### Disadvantages

- Additional cost
- Routing complexity
- Firewall policy management
- Stateful traffic requires correct routing
- Firewall deployment alone does not provide network failover

---

## 3. Decision

Azure Firewall will be deployed in both regional Hubs.

Network Security Groups will provide additional subnet-level security.

The architecture incorporates the following Azure Firewall capabilities:

- Network rules
- Application rules
- Firewall policies
- Logging
- Outbound traffic inspection
- Inter-region traffic inspection
- Firewall failure scenarios

Azure Firewall provides centralized security enforcement, while NSGs provide additional workload and subnet-level access controls.

---

## 4. Security Architecture Principles

- Workload traffic is restricted based on required communication.
- Internet egress is centrally controlled.
- Firewall policies define permitted network and application traffic.
- NSGs provide additional segmentation.
- Firewall logs provide visibility into inspected traffic.
- Routing is designed to maintain firewall inspection.
- Security controls are applied consistently across both regions.

---

# ADR-004 — Internet Egress Architecture

## Status

**Accepted**

---

## 1. Context

Application workloads require outbound Internet connectivity.

Uncontrolled direct outbound access is not desirable.

The architecture requires centralized outbound traffic control, inspection, logging, and policy enforcement.

---

## 2. Options Considered

### Option A — Direct Internet Access

Workloads access the Internet directly.

#### Advantages

- Simple
- Minimal routing configuration
- Lower network complexity

#### Disadvantages

- Limited centralized control
- Difficult security enforcement
- Limited visibility

---

### Option B — Azure Firewall-Based Egress

Application traffic is routed through Azure Firewall.

#### Advantages

- Centralized security
- Outbound policy enforcement
- Traffic logging
- Application and network filtering
- Centralized management

#### Disadvantages

- Additional cost
- Requires UDR configuration
- Routing errors may cause connectivity failures
- Requires firewall rules

---

## 3. Decision

Azure Firewall will be used for centralized outbound Internet traffic inspection.

Application workload subnets will use User Defined Routes to direct default Internet traffic toward the regional Azure Firewall.

The routing architecture uses the Azure Firewall as the centralized egress security boundary.

---

## 4. Egress Validation Methods

The implementation uses the following methods to inspect and troubleshoot outbound traffic:

- Effective Routes
- Next Hop
- Network connectivity tests
- Azure Firewall logs
- Firewall rule evaluation
- Application connectivity testing
- Route table inspection

---

# ADR-005 — Global Application Traffic Management

## Status

**Accepted**

---

## 1. Context

Application workloads will be deployed in both Azure regions.

The project requires an entry point that can direct users to healthy application endpoints and support application-level regional failover.

Application availability and network availability are treated as separate architectural concerns.

---

## 2. Options Considered

### Azure Traffic Manager

DNS-based global traffic distribution.

#### Advantages

- DNS-based routing
- Multiple routing methods
- Health monitoring
- Regional failover

#### Disadvantages

- DNS caching affects failover behavior
- Does not proxy application traffic
- Does not provide Layer 7 reverse proxy functionality

---

### Azure Front Door

Global Layer 7 application delivery service.

#### Advantages

- Global entry point
- HTTP and HTTPS routing
- Health probes
- Application-layer traffic distribution
- TLS termination capabilities
- Supports application-level failover
- Provides additional application security capabilities

#### Disadvantages

- Additional cost
- Requires application architecture compatibility
- Backend connectivity requirements must be considered
- Does not automatically provide private network routing failover

---

## 3. Decision

Azure Front Door is selected as the global application traffic management component.

The application architecture uses Azure Front Door to provide a global entry point for application workloads deployed across both regions.

---

## 4. Application Traffic Management Design

The architecture incorporates:

- Backend connectivity
- Health probes
- Regional backend health
- Traffic distribution
- Regional application failover
- HTTPS connectivity

Application failover is treated separately from private network failover.

Azure Front Door manages application traffic distribution and backend health. Private network routing and Firewall resiliency are handled by the network architecture.

---

# ADR-006 — Network Failover Strategy

## Status

**Accepted**

---

## 1. Context

The architecture contains two regional Azure Firewalls.

A Firewall failure may interrupt connectivity for workloads that depend on that Firewall.

However, the existence of a second Firewall does not automatically provide failover.

The project explores different network failover approaches and their operational behavior.

---

## 2. Options Considered

### Option A — Manual Route Changes

An administrator modifies User Defined Routes during a failure.

#### Advantages

- Simple
- Easy to understand
- Useful for demonstrating routing behavior
- No additional automation infrastructure required

#### Disadvantages

- Requires manual intervention
- Recovery time depends on administrator response
- Not suitable for fully automated production failover

---

### Option B — Automated UDR Modification

A monitoring system detects a failure and modifies route tables.

Potential components include:

- Azure Monitor
- Alert rules
- Azure Functions
- Azure Automation
- PowerShell
- Azure CLI
- Azure REST APIs

#### Advantages

- Automated response
- Reduces manual intervention
- Simulates production-style operational workflows

#### Disadvantages

- Requires automation design
- Detection and action delays
- Automation itself may fail
- Must prevent unnecessary route changes

---

### Option C — Dynamic Routing Using BGP

Use Azure Route Server and compatible Network Virtual Appliances to exchange routes dynamically.

#### Advantages

- Dynamic route exchange
- Route advertisement and withdrawal
- Advanced routing behavior
- Can support dynamic failover architecture

#### Disadvantages

- Additional complexity
- Additional cost
- Requires BGP knowledge
- Requires compatible network appliances

---

## 3. Decision

The project implements network failover progressively.

### Stage 1 — Manual Route Modification

Manual route changes are used to understand routing behavior and demonstrate failover scenarios.

This stage includes:

- Simulating Firewall failure
- Identifying affected traffic
- Examining effective routes
- Modifying UDRs
- Observing connectivity recovery
- Investigating return traffic paths
- Documenting recovery behavior

### Stage 2 — Automated Route Modification

Automated route modification is explored using Azure monitoring and automation components.

The automation architecture may incorporate:

- Azure Monitor
- Alert rules
- Azure Functions
- Automation
- PowerShell
- Azure CLI
- Azure REST APIs

### Stage 3 — Advanced Dynamic Routing

Dynamic routing using BGP and Azure Route Server is included as an advanced extension.

This stage explores:

- Route advertisement
- Route withdrawal
- Dynamic route selection
- BGP-based routing
- Network Virtual Appliance integration
- Dynamic failover behavior

---

## 4. Failover Design Principles

- Firewall failure does not automatically imply network failover.
- Failover behavior must be explicitly designed.
- Routing changes must account for stateful traffic.
- Return traffic must follow a compatible path.
- Application failover and network failover are separate.
- Failover recovery behavior must be documented.
- Automated route modification must include safeguards against unnecessary route changes.

---

# ADR-007 — Infrastructure Deployment Method

## Status

**Accepted**

---

## 1. Context

The project infrastructure can be deployed manually or using Infrastructure as Code.

The deployment approach must provide learning value, repeatability, troubleshooting capabilities, and a foundation for automation.

---

## 2. Options Considered

### Manual Deployment Using Azure Portal

#### Advantages

- Easy for beginners
- Provides understanding of Azure configuration
- Useful for initial troubleshooting
- Allows direct observation of resource configuration

#### Disadvantages

- Time-consuming
- Difficult to reproduce
- Configuration drift
- Less suitable for repeatable deployments

---

### Azure CLI

#### Advantages

- Repeatable commands
- Useful for automation
- Faster than manual deployment
- Supports scripting and operational tasks

#### Disadvantages

- Requires scripting
- Can become difficult to maintain for large environments

---

### Terraform

#### Advantages

- Infrastructure as Code
- Repeatable deployments
- Version control
- State management
- Reusable modules
- Supports automation pipelines

#### Disadvantages

- Requires Terraform knowledge
- State management complexity
- Requires additional learning

---

## 3. Decision

The project uses a progressive deployment approach.

### Stage 1 — Azure Portal

Azure Portal is used to understand the infrastructure configuration and resource relationships.

Initial infrastructure deployment includes understanding:

- Resource groups
- VNets
- Subnets
- VNet Peering
- Route Tables
- NSGs
- Azure Firewall
- Firewall Policies
- Virtual Machines
- Monitoring resources

### Stage 2 — Azure CLI

Azure CLI is used for operational tasks, configuration, troubleshooting, and repeatable resource management.

### Stage 3 — Terraform

Terraform is used to implement the infrastructure as code.

The Terraform implementation incorporates:

- Resource groups
- VNets
- Subnets
- VNet Peering
- Route Tables
- NSGs
- Azure Firewall
- Firewall Policies
- Virtual Machines
- Monitoring resources

The project progressively transitions from portal-based deployment to Infrastructure as Code.

---

# ADR-008 — Monitoring and Troubleshooting

## Status

**Accepted**

---

## 1. Context

Network connectivity failures can occur because of:

- Routing
- Firewall rules
- NSGs
- DNS
- Peering
- Incorrect next hops
- Workload configuration
- Application failures

The architecture requires diagnostic visibility to identify, investigate, and resolve network issues.

---

## 2. Decision

The project uses Azure-native monitoring and diagnostic tools.

The monitoring and troubleshooting architecture incorporates:

- Azure Monitor
- Log Analytics
- Network Watcher
- Effective Routes
- Next Hop
- IP Flow Verify
- Connection Troubleshoot
- Packet Capture
- Azure Firewall Logs

---

## 3. Troubleshooting Methodology

Network troubleshooting follows a structured approach.

### Step 1 — Identify the Symptom

Determine the affected workload, source, destination, protocol, port, and time of failure.

### Step 2 — Verify Workload Configuration

Inspect:

- VM status
- Operating system network configuration
- IP address
- DNS configuration
- Application status
- Local firewall configuration

### Step 3 — Inspect Network Security

Review:

- NSG rules
- Azure Firewall rules
- Firewall policy
- Network rule collections
- Application rule collections

### Step 4 — Inspect Routing

Review:

- Effective Routes
- Route table associations
- User Defined Routes
- Next-hop behavior
- VNet Peering configuration
- Return traffic routing

### Step 5 — Investigate Connectivity

Use Network Watcher tools:

- IP Flow Verify
- Next Hop
- Connection Troubleshoot
- Packet Capture

### Step 6 — Review Logs

Inspect:

- Azure Firewall logs
- Network diagnostic logs
- Log Analytics queries
- Azure Monitor metrics
- Alerts

### Step 7 — Identify Root Cause

Document the root cause and the configuration responsible for the connectivity failure.

### Step 8 — Apply Resolution

Implement the required configuration change and verify connectivity.

---

## 4. Troubleshooting Documentation

Each troubleshooting scenario is documented using the following structure:

- Scenario
- Symptoms
- Affected resources
- Investigation steps
- Root cause
- Resolution
- Verification
- Lessons learned

---

# Summary of Architecture Decisions

| ADR | Decision | Status |
|---|---|---|
| ADR-001 | Multi-Region Hub-and-Spoke Network Topology | Accepted |
| ADR-002 | Regional Spoke Connectivity | Accepted |
| ADR-003 | Centralized Network Security Using Azure Firewall | Accepted |
| ADR-004 | Centralized Internet Egress Through Azure Firewall | Accepted |
| ADR-005 | Global Application Traffic Management Using Azure Front Door | Accepted |
| ADR-006 | Progressive Network Failover Strategy | Accepted |
| ADR-007 | Progressive Infrastructure Deployment Using Portal, CLI, and Terraform | Accepted |
| ADR-008 | Azure-Native Monitoring and Troubleshooting | Accepted |

---

# Architecture Principles

The project follows these architectural principles:

1. Regional isolation
2. Centralized network security
3. Controlled Internet egress
4. Explicit network routing
5. Segmented workloads
6. Centralized monitoring
7. Controlled inter-region connectivity
8. Consistent security enforcement
9. Structured troubleshooting
10. Progressive implementation
11. Infrastructure as Code
12. Cost awareness
13. Separation of application failover and network failover
14. Documented architecture decisions
