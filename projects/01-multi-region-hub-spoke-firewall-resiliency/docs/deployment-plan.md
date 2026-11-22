# Deployment Plan — Multi-Region Hub-and-Spoke Network

## 1. Purpose

This document defines the deployment sequence for implementing the multi-region Azure Hub-and-Spoke network architecture.

The deployment follows a structured approach to ensure network infrastructure, security, connectivity, workloads, monitoring, and testing are implemented in the correct order.

---

## 2. Deployment Phases

### Phase 1 — Network Foundation

Deploy the foundational network infrastructure in both Azure regions.

Resources:

- Resource Groups
- Hub Virtual Networks
- Application Spoke Virtual Networks
- Shared Services Spoke Virtual Networks
- Subnets
- Network Security Groups

Objectives:

- Establish regional network address spaces
- Create the required subnet structure
- Apply initial network security rules
- Ensure IP address spaces do not overlap

---

### Phase 2 — Hub Network Connectivity

Configure connectivity between Hub and Spoke Virtual Networks.

Tasks:

- Configure Hub-to-Spoke VNet Peering
- Configure Spoke-to-Hub VNet Peering
- Enable required peering settings
- Verify basic network connectivity
- Validate peering status

---

### Phase 3 — Centralized Security

Deploy Azure Firewall in each regional Hub.

Tasks:

- Create Azure Firewall subnets
- Deploy Azure Firewall
- Configure Firewall Policies
- Create required Network Rules
- Create required Application Rules
- Configure Firewall diagnostic settings

Objectives:

- Establish centralized security enforcement
- Control outbound Internet connectivity
- Prepare traffic inspection capabilities

---

### Phase 4 — Routing Configuration

Configure routing to control traffic paths.

Tasks:

- Create User Defined Route tables
- Configure routes for workload subnets
- Configure Firewall next-hop routing
- Associate route tables with required subnets
- Configure inter-region routing where required
- Validate effective routes
- Verify next-hop selection

Objectives:

- Route required traffic through Azure Firewall
- Establish controlled network communication
- Verify regional and inter-region routing behavior

---

### Phase 5 — Inter-Region Connectivity

Connect the regional Hub Virtual Networks.

Tasks:

- Configure Global VNet Peering
- Verify peering connectivity
- Configure required routing
- Validate cross-region communication
- Verify traffic inspection requirements
- Investigate return-path routing

Objectives:

- Establish private inter-region connectivity
- Validate controlled communication between regions

---

### Phase 6 — Workload Deployment

Deploy workload Virtual Machines into the Application and Shared Services Spokes.

Tasks:

- Deploy Windows and/or Linux Virtual Machines
- Configure VM network interfaces
- Apply Network Security Groups
- Configure management access
- Validate subnet connectivity

Objectives:

- Establish test workloads
- Provide endpoints for network testing
- Validate workload-level network security

---

### Phase 7 — Global Application Connectivity

Deploy and configure Azure Front Door.

Tasks:

- Configure Front Door profile
- Configure endpoints
- Configure backend origins
- Configure health probes
- Configure routing rules
- Configure backend security
- Validate application connectivity
- Test regional application failover

Objectives:

- Establish global application connectivity
- Validate backend health monitoring
- Investigate application failover behavior

---

### Phase 8 — Monitoring and Diagnostics

Configure network monitoring and troubleshooting capabilities.

Tasks:

- Create Log Analytics Workspace
- Configure Azure Firewall diagnostic settings
- Configure Azure Monitor
- Enable Network Watcher
- Review Firewall logs
- Validate diagnostic data collection

Objectives:

- Establish centralized network monitoring
- Provide visibility into network traffic
- Support troubleshooting and failure investigation

---

### Phase 9 — Network Validation and Failure Testing

Perform connectivity, security, routing, and failure tests.

Testing includes:

- Hub-to-Spoke connectivity
- Spoke-to-Spoke connectivity
- Internet connectivity
- Firewall traffic inspection
- NSG enforcement
- Inter-region connectivity
- DNS resolution
- Application connectivity
- Incorrect routing scenarios
- Network failure scenarios
- Regional application failover

Test results are documented in the Failure Testing Plan.

---

### Phase 10 — Cleanup and Cost Management

Review deployed resources and remove unnecessary infrastructure.

Tasks:

- Stop Virtual Machines when not required
- Review Azure Firewall deployment costs
- Review Front Door costs
- Remove temporary test resources
- Delete unused resources
- Verify resource cleanup

Objectives:

- Minimize unnecessary Azure costs
- Maintain a clean lab environment
- Ensure resources are deployed only when required

---

## 3. Deployment Order

The overall deployment sequence is:

```text
Resource Groups
      |
      v
Virtual Networks and Subnets
      |
      v
Network Security Groups
      |
      v
VNet Peering
      |
      v
Azure Firewall
      |
      v
Firewall Policies
      |
      v
User Defined Routes
      |
      v
Inter-Region Connectivity
      |
      v
Workload Virtual Machines
      |
      v
Azure Front Door
      |
      v
Monitoring and Logging
      |
      v
Connectivity and Failure Testing
      |
      v
Cleanup
```

---

## 4. Deployment Approach

The implementation will be performed progressively.

The initial deployment focuses on establishing the network foundation and validating basic connectivity.

Centralized Firewall inspection and routing will be introduced after the basic network connectivity is established.

Advanced routing, resiliency, failure testing, monitoring, and global application connectivity will be implemented after the foundational network is operational.

This approach allows each component to be tested independently before introducing additional complexity.

---

## 5. Validation Principles

Each deployment phase should be validated before moving to the next phase.

Validation includes:

- Resource deployment verification
- Network configuration verification
- Peering status verification
- Routing verification
- Security rule verification
- Connectivity testing
- Firewall log analysis
- Troubleshooting

Deployment issues and their resolutions should be documented as part of the project implementation.