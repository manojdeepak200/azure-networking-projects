# Failure Testing Plan — Multi-Region Azure Network

## 1. Purpose

This document defines the failure scenarios, testing procedures, expected behavior, investigation methods, recovery procedures, and measurements for the Multi-Region Azure Hub-and-Spoke Network with Firewall Resiliency project.

The testing plan covers:

- Normal network connectivity
- Security failures
- Routing failures
- Network connectivity failures
- DNS failures
- Firewall failures
- Regional workload failures
- Asymmetric routing
- Manual failover
- Automated failover
- Monitoring and troubleshooting

All testing will be performed in a controlled lab environment.

Test evidence, actual results, recovery times, screenshots, logs, and root causes will be recorded as scenarios are executed.

---

## 2. Testing Principles

The project follows these testing principles:

- Establish a working baseline before introducing failures.
- Change only one component at a time.
- Record the original configuration.
- Document expected behavior before executing a test.
- Capture evidence of the failure.
- Use Azure diagnostic tools to investigate.
- Restore the original configuration after each test.
- Confirm the environment is healthy before proceeding.
- Document the actual outcome of every executed scenario.
- Measure recovery time wherever applicable.
- Perform environment cleanup after testing.

---

## 3. Baseline Connectivity Tests

Before introducing failures, establish normal network behavior.

| Test ID | Test | Expected Result |
|---|---|---|
| BASE-01 | VM-to-VM connectivity within the same subnet | Successful connectivity |
| BASE-02 | VM-to-VM connectivity across approved networks | Successful connectivity |
| BASE-03 | Internet connectivity through regional Azure Firewall | Successful connectivity |
| BASE-04 | DNS resolution | Successful name resolution |
| BASE-05 | Application access | Application accessible |
| BASE-06 | Azure Firewall logging | Traffic logs generated |
| BASE-07 | Effective routes | Expected routes displayed |
| BASE-08 | Network Watcher connectivity tools | Network path information available |

---

# 4. Security Failure Tests

## SEC-01 — Azure Firewall Denies Traffic

### Objective

Understand how Azure Firewall blocks traffic and how to investigate a firewall denial.

### Failure Injection

Create or modify an Azure Firewall rule to deny a required connection.

### Expected Symptoms

- Application connection fails.
- TCP connection cannot be established.
- Azure Firewall logs show denied traffic.

### Investigation

- Review Firewall Policy.
- Review rule collections.
- Check rule priority.
- Review Azure Firewall logs.
- Identify source and destination.
- Confirm the correct policy is associated.
- Confirm the protocol and destination port.

### Resolution

Correct or remove the incorrect firewall rule.

### Recovery Procedure

1. Identify the firewall rule responsible for the denial.
2. Review the rule configuration.
3. Correct the destination, protocol, port, or action.
4. Confirm the required traffic is permitted.
5. Test connectivity again.
6. Review firewall logs.

### Expected Outcome

The required connection is restored and the traffic is recorded in the firewall logs.

---

## SEC-02 — NSG Blocks Traffic

### Objective

Investigate a connectivity failure caused by a Network Security Group.

### Failure Injection

Create an NSG rule denying required traffic.

### Expected Symptoms

- Connection fails.
- Traffic may not reach Azure Firewall depending on the location of the NSG and the traffic path.
- The connection may be blocked at the subnet or network interface level.

### Investigation

- Review inbound rules.
- Review outbound rules.
- Check rule priority.
- Use IP Flow Verify.
- Check effective security rules.
- Identify the subnet or NIC associated with the NSG.

### Resolution

Correct the NSG rule.

### Recovery Procedure

1. Identify the blocking NSG.
2. Review inbound and outbound rules.
3. Identify the matching rule.
4. Correct its priority or configuration.
5. Retest connectivity.
6. Confirm the expected traffic is permitted.

### Expected Outcome

The required traffic is permitted while other security restrictions remain in place.

---

# 5. Routing Failure Tests

## ROUTE-01 — Incorrect UDR Next Hop

### Objective

Understand the impact of an incorrect next-hop configuration.

### Failure Injection

Configure a route with an incorrect Virtual Appliance IP.

### Expected Symptoms

- Connectivity failure.
- Traffic may be directed to an invalid next hop.
- Traffic may not reach the intended Azure Firewall.
- Application connectivity may time out.

### Investigation

- Review the route table.
- Review effective routes.
- Use Network Watcher Next Hop.
- Use Connection Troubleshoot.
- Verify the Azure Firewall private IP.
- Confirm the route table association.

### Resolution

Correct the next-hop address.

### Recovery Procedure

1. Identify the incorrect route.
2. Compare the configured next hop with the intended next hop.
3. Confirm the Azure Firewall private IP.
4. Correct the route.
5. Review effective routes.
6. Test connectivity.

### Expected Outcome

Traffic follows the intended route and reaches the appropriate network security component.

---

## ROUTE-02 — Missing Route

### Objective

Investigate connectivity when a required route is missing.

### Failure Injection

Remove a required route from a route table.

### Expected Symptoms

- Traffic follows an alternate system route or fails.
- Traffic may bypass intended inspection depending on the architecture.
- Connectivity may become inconsistent.

### Investigation

- Review route table configuration.
- Review effective routes.
- Identify the selected route.
- Compare expected and actual paths.
- Review route table associations.

### Resolution

Restore the required route.

### Recovery Procedure

1. Identify the missing route.
2. Confirm the destination prefix.
3. Confirm the intended next hop.
4. Add the route.
5. Confirm route table association.
6. Review effective routes.
7. Test connectivity.

### Expected Outcome

Traffic follows the intended routing path.

---

## ROUTE-03 — Incorrect Route Prefix

### Objective

Understand Longest Prefix Match and route selection.

### Failure Injection

Configure an incorrect or overly specific route.

### Expected Symptoms

Traffic may use an unexpected next hop.

Connectivity may fail or follow an unintended network path.

### Investigation

- Compare route prefixes.
- Review effective routes.
- Identify the most specific matching route.
- Validate the selected next hop.
- Compare the destination IP with configured prefixes.

### Resolution

Correct the destination prefix.

### Recovery Procedure

1. Identify the incorrect prefix.
2. Review all matching routes.
3. Determine which route takes precedence.
4. Correct the prefix.
5. Review effective routes.
6. Test connectivity.

### Expected Outcome

The correct route is selected based on the destination address and route precedence.

---

# 6. Connectivity Failure Tests

## CONN-01 — VNet Peering Failure

### Objective

Investigate connectivity when a VNet Peering connection is unavailable.

### Failure Injection

Disconnect or disable a selected peering connection in the lab.

### Expected Symptoms

- Connectivity between affected networks fails.
- Routes associated with the peering may become unavailable.
- Cross-network communication may be interrupted.

### Investigation

- Review peering status.
- Check both sides of the peering.
- Review effective routes.
- Confirm address spaces do not overlap.
- Review peering configuration.
- Use Network Watcher.

### Resolution

Restore the peering configuration.

### Recovery Procedure

1. Identify the affected peering connection.
2. Check the status of both peering sides.
3. Review the peering configuration.
4. Restore the connection.
5. Review effective routes.
6. Test connectivity between the affected networks.

### Expected Outcome

Connectivity between the peered VNets is restored.

---

## CONN-02 — DNS Failure

### Objective

Understand how DNS failures affect application connectivity.

### Failure Injection

Configure an incorrect DNS server or introduce a controlled DNS resolution failure.

### Expected Symptoms

- Application access using hostname fails.
- Access using IP may still work.
- DNS requests may time out.
- Applications may report name resolution errors.

### Investigation

- Run `nslookup`.
- Run `dig` where available.
- Review DNS configuration.
- Verify DNS connectivity.
- Review Private DNS configuration.
- Confirm DNS records.
- Confirm the DNS server is reachable.

### Resolution

Correct the DNS configuration.

### Recovery Procedure

1. Identify the DNS server configured on the workload.
2. Confirm the expected DNS server address.
3. Review DNS records and zones.
4. Correct the DNS configuration.
5. Test name resolution.
6. Test application connectivity using the hostname.

### Expected Outcome

DNS resolution succeeds and the application is accessible using its hostname.

---

# 7. Firewall and Regional Failure Tests

## FAIL-01 — Regional Firewall Failure

### Objective

Investigate the impact of a regional Azure Firewall failure.

### Failure Injection

Simulate an outage or failure condition using a controlled method.

### Expected Symptoms

The impact depends on the routing architecture.

Possible symptoms include:

- Connectivity may fail.
- Internet egress may fail.
- Inter-region communication may fail.
- Alternate paths may or may not exist.
- Stateful connections may be interrupted.
- Applications may experience timeouts.

### Investigation

- Review route tables.
- Review effective routes.
- Identify next hops.
- Check Firewall health.
- Review connectivity diagnostics.
- Review Azure Firewall logs.
- Investigate the forward and return traffic paths.

### Recovery Procedure

Execute the designed recovery or failover procedure.

### Measurements

Record:

- Failure time
- Detection time
- Recovery initiation time
- Connectivity recovery time
- Packet loss
- Application impact
- Traffic path before failure
- Traffic path after recovery

### Expected Outcome

The impact of the Firewall failure is documented, and the configured recovery procedure restores the required connectivity where applicable.

---

## FAIL-02 — Regional Workload Failure

### Objective

Evaluate application availability during a workload failure.

### Failure Injection

Stop or make a regional application endpoint unavailable.

### Expected Symptoms

- Application endpoint becomes unhealthy.
- Users may experience connection failures.
- Global traffic management may redirect traffic if configured correctly.

### Investigation

- Review application health.
- Review health probes.
- Review backend status.
- Validate regional traffic distribution.
- Review Azure Front Door configuration.
- Confirm health probe behavior.

### Resolution

Restore workload availability or allow the global routing mechanism to direct traffic to a healthy backend.

### Recovery Procedure

1. Identify the unhealthy application endpoint.
2. Review application health.
3. Review backend health.
4. Confirm the global traffic management response.
5. Restore the application if required.
6. Confirm traffic is reaching a healthy endpoint.

### Expected Outcome

Application traffic is directed to a healthy backend according to the configured global traffic management behavior.

---

# 8. Asymmetric Routing Test

## ASYM-01 — Different Forward and Return Paths

### Objective

Investigate stateful network behavior when traffic follows different paths in each direction.

### Failure Injection

Create a controlled routing configuration where forward and return traffic use different paths.

### Expected Symptoms

Possible symptoms include:

- TCP connection failures
- Intermittent connectivity
- Firewall session drops
- Unexpected connection resets
- Connections timing out

### Investigation

- Trace the forward path.
- Trace the return path.
- Review effective routes on both sides.
- Review Azure Firewall logs.
- Use Network Watcher.
- Capture packets where appropriate.
- Compare source and destination routes.

### Resolution

Modify routing to ensure compatible forward and return paths.

### Recovery Procedure

1. Identify the source and destination.
2. Map the forward traffic path.
3. Map the return traffic path.
4. Identify the routing inconsistency.
5. Correct the route configuration.
6. Review effective routes.
7. Test connectivity again.

### Expected Outcome

Forward and return traffic follow compatible paths, allowing stateful network components to process the connection correctly.

---

# 9. Failover Testing

## FAILOVER-01 — Manual Route Failover

### Objective

Demonstrate how a route can be manually changed during a failure.

### Procedure

1. Establish normal connectivity.
2. Record the current route.
3. Introduce a controlled failure.
4. Confirm connectivity is interrupted.
5. Modify the route according to the failover design.
6. Review the effective route.
7. Test connectivity.
8. Record recovery time.
9. Restore the original configuration.

### Measurements

| Metric | Recorded Value |
|---|---|
| Failure time | |
| Detection time | |
| Failover initiation | |
| Recovery time | |
| Packet loss | |
| Application impact | |
| Original traffic path | |
| Recovery traffic path | |

### Expected Outcome

The manual route modification changes the traffic path according to the failover design and restores the required connectivity where an alternate path is available.

---

## FAILOVER-02 — Automated Failover

### Objective

Investigate automated route modification during a network failure.

### Potential Components

- Azure Monitor
- Alert Rules
- Azure Functions
- Azure Automation
- PowerShell
- Azure CLI

### Procedure

1. Establish baseline connectivity.
2. Configure failure detection.
3. Configure automated response.
4. Introduce a controlled failure.
5. Observe failure detection.
6. Observe route modification.
7. Review effective routes.
8. Test connectivity.
9. Record recovery time.
10. Investigate automation failures.
11. Restore the original configuration.

### Expected Outcome

The automation detects the configured failure condition and performs the intended routing action.

The resulting traffic path and recovery behavior are documented.

---

# 10. Test Result Documentation

Every executed test will use the following template.

## Test Information

| Attribute | Value |
|---|---|
| Test ID | |
| Test Name | |
| Execution Date | |
| Environment | |
| Tester | |
| Result | |

## Objective

Describe what the test is intended to demonstrate.

## Initial Configuration

Document the working configuration before introducing the failure.

## Failure Injection

Describe exactly what was changed.

## Expected Result

Describe the expected behavior.

## Actual Result

Document what happened.

## Symptoms

Document the observed failure.

## Investigation

List the diagnostic tools and commands used.

## Root Cause

Describe the underlying issue.

## Resolution

Describe the corrective action.

## Recovery Verification

Confirm whether the issue was resolved and whether the expected traffic path was restored.

## Evidence

Include:

- Screenshots
- Logs
- Commands
- Diagnostic results
- Effective routes
- Network Watcher results
- Architecture diagrams

## Lessons Learned

Document the technical lessons.

---

# 11. Test Scenario Catalogue

| Test ID | Scenario | Test Objective |
|---|---|---|
| BASE-01 | Baseline connectivity within subnet | Confirm local connectivity |
| BASE-02 | Approved inter-network connectivity | Confirm permitted network communication |
| BASE-03 | Internet egress | Confirm outbound traffic through Azure Firewall |
| BASE-04 | DNS resolution | Confirm name resolution |
| BASE-05 | Application access | Confirm application availability |
| BASE-06 | Firewall logging | Confirm firewall diagnostic logging |
| BASE-07 | Effective routes | Confirm expected routing behavior |
| BASE-08 | Network Watcher tools | Confirm diagnostic visibility |
| SEC-01 | Firewall denial | Investigate Azure Firewall blocking traffic |
| SEC-02 | NSG denial | Investigate NSG blocking traffic |
| ROUTE-01 | Incorrect next hop | Investigate incorrect routing |
| ROUTE-02 | Missing route | Investigate route absence |
| ROUTE-03 | Incorrect prefix | Investigate route selection |
| CONN-01 | VNet Peering failure | Investigate peering connectivity failure |
| CONN-02 | DNS failure | Investigate name resolution failure |
| FAIL-01 | Regional Firewall failure | Investigate Firewall outage impact |
| FAIL-02 | Regional workload failure | Investigate application availability |
| ASYM-01 | Asymmetric routing | Investigate forward and return path differences |
| FAILOVER-01 | Manual route failover | Demonstrate manual recovery |
| FAILOVER-02 | Automated failover | Demonstrate automated route modification |

---

# 12. Completion Criteria

The failure testing exercise is complete when:

- Normal connectivity scenarios have been demonstrated.
- Security failures have been reproduced and investigated.
- Routing failures have been reproduced and investigated.
- Connectivity failures have been investigated.
- DNS failure behavior has been documented.
- Regional Firewall failure scenarios have been evaluated.
- Regional workload failure scenarios have been evaluated.
- Asymmetric routing behavior has been investigated.
- Manual failover behavior has been demonstrated.
- Automated failover behavior has been demonstrated where implemented.
- Actual test results have been documented.
- Evidence has been collected.
- Recovery procedures have been documented.
- Environment cleanup has been completed.
- Lessons learned have been documented.

---

# 13. Project Testing Principles

This project distinguishes between application availability, network availability, and network security.

A successful application failover does not necessarily indicate private network failover.

A successful route change does not necessarily indicate that stateful connections were preserved.

A second Azure Firewall does not automatically provide failover.

Every failure scenario must be investigated by examining the actual traffic path, effective routes, security controls, and return traffic.

The final test documentation will describe the observed behavior rather than assuming that the architecture behaves as expected.
