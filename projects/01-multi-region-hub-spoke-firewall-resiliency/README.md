# Project 01 — Multi-Region Azure Hub-and-Spoke Network with Firewall Resiliency

## Project Overview

This project explores the design and implementation of a resilient, multi-region Azure network architecture using the Hub-and-Spoke topology.

The environment consists of two Azure regions, each containing a central Hub Virtual Network, Azure Firewall, and multiple Spoke Virtual Networks hosting workloads.

The two regional hubs are interconnected using Global VNet Peering. Network traffic is routed through centralized security infrastructure, with routing controls designed to support connectivity, traffic inspection, resiliency, and failover scenarios.

The project will also explore global application traffic distribution, network monitoring, failure simulation, and troubleshooting of complex network connectivity issues.

> **Project Status:** Architecture and documentation phase. Infrastructure deployment, testing, and validation will be documented as they are completed.

---

## Problem Statement

Organizations running workloads across multiple Azure regions require secure and resilient network connectivity between applications, shared services, and regional environments.

A poorly designed multi-region network can introduce several challenges:

- Uncontrolled network connectivity between workloads
- Inconsistent security enforcement across regions
- Routing complexity during regional or network failures
- Traffic flowing through unintended paths
- Asymmetric routing through stateful network appliances
- Difficulty identifying connectivity failures
- Lack of visibility into network traffic and firewall decisions
- Application availability issues during regional failures

This project addresses these challenges by designing a centralized, segmented, and resilient multi-region Azure network architecture.

---

## Project Objectives

- Design a multi-region Hub-and-Spoke network architecture.
- Implement network segmentation using Virtual Networks and subnets.
- Establish connectivity between regional hubs using Global VNet Peering.
- Deploy Azure Firewall in both regions.
- Implement centralized traffic inspection and network security controls.
- Configure User Defined Routes (UDRs) to control network traffic paths.
- Investigate routing behavior and route selection.
- Design and test network failover scenarios.
- Implement global application traffic distribution.
- Monitor network connectivity and firewall activity.
- Troubleshoot real-world network failure scenarios.
- Document architecture decisions, implementation steps, testing results, and lessons learned.

---

## Proposed Architecture

The environment will consist of two Azure regions.

### Region 1

- Hub Virtual Network
- Azure Firewall
- Workload Spoke Virtual Network
- Shared Services Spoke Virtual Network
- Route Tables
- Network Security Groups

### Region 2

- Hub Virtual Network
- Azure Firewall
- Workload Spoke Virtual Network
- Shared Services Spoke Virtual Network
- Route Tables
- Network Security Groups

### Inter-Region Connectivity

- Global VNet Peering between regional Hub VNets
- Controlled routing between regions
- Centralized firewall inspection
- Defined primary and backup traffic paths

### Global Application Connectivity

A global traffic management service will be evaluated to distribute application traffic across workloads in both regions.

Potential services:

- Azure Front Door
- Azure Traffic Manager

The final service selection will depend on the application architecture, traffic requirements, health monitoring capabilities, and routing behavior.

---

## Architecture Components

| Component | Purpose |
|---|---|
| Azure Virtual Network | Provides network isolation and connectivity |
| Hub VNet | Central network infrastructure for each region |
| Spoke VNet | Hosts application and shared services workloads |
| Azure Firewall | Centralized network traffic inspection and filtering |
| Global VNet Peering | Connects regional Hub VNets |
| User Defined Routes | Controls traffic forwarding and routing paths |
| Network Security Groups | Provides subnet and network interface-level traffic filtering |
| Virtual Machines | Simulates application and infrastructure workloads |
| Azure Front Door / Traffic Manager | Provides global application traffic distribution |
| Azure Monitor | Provides monitoring and alerting capabilities |
| Log Analytics Workspace | Centralizes network and firewall logs |
| Network Watcher | Provides network diagnostics and troubleshooting capabilities |

---

## High-Level Architecture

The architecture will contain two regional network environments.

Each region will have a central Hub VNet connected to multiple Spoke VNets. Azure Firewall will provide centralized traffic inspection and security enforcement.

The regional Hub VNets will be interconnected using Global VNet Peering.

Routing controls will be designed to support controlled inter-region communication and provide alternative traffic paths during selected failure scenarios.

Application workloads will be deployed in both regions to demonstrate regional availability and global traffic distribution.

> The architecture diagram is currently conceptual. Traffic paths, route propagation behavior, peering configuration, and failover mechanisms will be validated during the design and implementation phases.

---

## Project Phases

| Phase | Description | Status |
|---|---|---|
| Phase 0 | Requirements and Architecture Design | In Progress |
| Phase 1 | Network Foundation | Not Started |
| Phase 2 | Azure Firewall Deployment | Not Started |
| Phase 3 | Routing and Traffic Inspection | Not Started |
| Phase 4 | Network Resiliency and Failover | Not Started |
| Phase 5 | Global Application Traffic Routing | Not Started |
| Phase 6 | Monitoring and Logging | Not Started |
| Phase 7 | Troubleshooting and Failure Simulation | Not Started |
| Phase 8 | Documentation and Portfolio Preparation | In Progress |
| Phase 9 | Video Demonstration | Not Started |

---

## Planned Failure and Troubleshooting Scenarios

The following scenarios will be investigated during the implementation and testing phases.

### Network Connectivity

- Spoke-to-spoke connectivity failure
- Inter-region connectivity failure
- Global VNet Peering failure
- Incorrect subnet configuration

### Routing

- Incorrect User Defined Route
- Incorrect next-hop configuration
- Missing route
- Unexpected route selection
- Traffic bypassing Azure Firewall
- Routing blackhole scenarios

### Security

- Azure Firewall denying traffic
- Network Security Group denying traffic
- Incorrect firewall rule configuration
- Unintended public network exposure

### Resiliency

- Regional firewall failure
- Regional workload failure
- Network path failure
- Backup route failure
- Failover configuration failure

### Advanced Troubleshooting

- Asymmetric routing
- Stateful firewall session failures
- DNS resolution issues
- Incorrect return traffic path
- Application connectivity failure despite successful network connectivity

Each scenario will include:

- Problem description
- Architecture context
- Symptoms
- Investigation steps
- Diagnostic tools
- Root cause
- Resolution
- Validation results
- Lessons learned

---

## Learning Outcomes

By completing this project, I aim to develop practical knowledge and troubleshooting experience in:

- Azure networking architecture
- Hub-and-Spoke network design
- Multi-region architecture
- VNet peering
- Global VNet Peering
- Azure Firewall
- User Defined Routes
- Route selection and effective routes
- Network Security Groups
- Centralized traffic inspection
- Network resiliency
- Failover design
- Azure Front Door and Traffic Manager
- Network monitoring
- Log Analytics and KQL
- Network Watcher
- Asymmetric routing
- Network troubleshooting
- Infrastructure architecture and design decisions

---

## Repository Structure

```text
01-multi-region-hub-spoke-firewall-resiliency/
│
├── README.md
│
├── architecture/
│   └── architecture-diagram.md
│
├── docs/
│   ├── project-plan.md
│   ├── architecture-design.md
│   ├── ip-address-plan.md
│   ├── routing-design.md
│   ├── failure-testing-plan.md
│   └── learning-guide.md
│
├── implementation/
│   └── implementation-status.md
│
├── testing/
│   └── test-plan.md
│
├── troubleshooting/
│   └── troubleshooting-plan.md
│
└── video/
    └── video-plan.md
```

Additional directories and files will be created as the project progresses.

---

## Implementation Status

| Deliverable | Status |
|---|---|
| Project concept | Completed |
| Initial requirements | Completed |
| High-level architecture | In Progress |
| Architecture diagram | Pending |
| IP address plan | Pending |
| Routing design | Pending |
| Infrastructure deployment | Not Started |
| Firewall configuration | Not Started |
| Routing validation | Not Started |
| Failover testing | Not Started |
| Troubleshooting documentation | Not Started |
| Monitoring implementation | Not Started |
| Video demonstration | Not Started |

---

## Important Design Considerations

This project will explicitly investigate and validate the following architectural considerations:

- Azure Firewall availability does not automatically guarantee network path failover.
- Static User Defined Routes do not dynamically learn or withdraw routes.
- Global VNet Peering does not provide automatic transitive routing.
- Application-level traffic failover and private network routing failover are separate mechanisms.
- Active-active infrastructure does not automatically mean active-active application architecture.
- Stateful firewalls can introduce asymmetric routing problems.
- Failover behavior must be tested rather than assumed.
- Network architecture must consider security, availability, operational complexity, and cost.

Advanced extensions may include Azure Route Server, BGP-based routing, automated route changes, Terraform deployment, and CI/CD pipelines.

---

## Cost Management

Azure resources used in this project may incur costs, particularly:

- Azure Firewall
- Public IP addresses
- Virtual Machines
- Azure Front Door
- Log Analytics data ingestion
- Cross-region network traffic
- Advanced networking services

Resources will be deployed in stages to minimize unnecessary cost.

Expensive resources will be stopped, removed, or cleaned up when they are not required for testing.

---

## Project Disclaimer

This is a personal Azure learning and portfolio project designed to simulate enterprise networking scenarios.

Architecture designs, implementation configurations, and test results will be documented separately.

Features will only be marked as implemented or validated after they have been deployed and tested.

---

## Author

Azure Platform Engineer | Azure Networking | Infrastructure | Cloud Security | Terraform

This project is part of my practical cloud engineering portfolio focused on Azure infrastructure, networking, security, architecture, and troubleshooting.
