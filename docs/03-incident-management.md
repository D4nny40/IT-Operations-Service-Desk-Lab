# 03 — Incident Management and Service Recovery

## 1. Overview

This document covers the incident management workflow implemented and tested within the IT Operations Service Desk Lab.

The objective was to simulate a service-affecting incident and demonstrate how an IT Operations team would respond, investigate, escalate, resolve and document the issue.

GLPI was used as the IT service management (ITSM) platform, providing a central location for incident tracking, technician assignments, investigation notes and resolution records.

A controlled outage involving an Nginx web server was used as the technical scenario.

The exercise combined service desk processes with practical Linux troubleshooting.

## 2. Objectives

The main objectives were to:

- Create and categorise an incident within GLPI.
- Assign urgency, impact and priority.
- Allocate the incident to first-line IT Operations support.
- Document initial investigation and troubleshooting requirements.
- Escalate the incident to second-line Infrastructure Support.
- Investigate a simulated Linux web service outage.
- Identify and document the cause of the failure.
- Restore service availability.
- Verify successful recovery.
- Record the resolution.
- Approve the solution and close the incident.

## 3. Incident Summary

| Field | Details |
|---|---|
| Incident ID | GLPI Ticket #1 |
| Title | Internal Web Service Unavailable |
| Incident type | Incident |
| Environment | Ubuntu Server VM |
| Affected service | Nginx web server |
| Service endpoint | `http://127.0.0.1/` |
| Urgency | High |
| Impact | High |
| Calculated priority | High |
| Initial support | IT Operations - L1 |
| Escalated support | Infrastructure - L2 |
| Root cause | Controlled shutdown of Nginx |
| Resolution | Restart Nginx and verify HTTP availability |
| Final ticket status | Closed |
| Exercise date | 10 November 2026 |

**Incident classification:** Controlled training simulation.

The incident was not a genuine production outage. The scenario was created to demonstrate operational procedures in a virtualised environment.

## 4. Incident Management Environment

The incident was managed using the GLPI service desk deployed earlier in the project.

The environment consisted of:

- VMware Workstation running Ubuntu Server.
- GLPI and MySQL deployed using Docker Compose.
- Dedicated first-line and second-line support accounts.
- Separate support groups configured in GLPI.
- Nginx installed directly on Ubuntu as a test web service.

The Nginx service was independent of the GLPI application containers.

This allowed the web service to be stopped and restarted without intentionally disrupting the incident-management platform.

### Support accounts

| Account | Responsibility |
|---|---|
| `ops.l1` | Initial incident response |
| `infra.l2` | Technical investigation and recovery |
| `ops.admin` | Operational administration |

The incident workflow used the separate support roles configured during the user-management phase.

## 5. Incident Creation

### 5.1 Creating the Ticket

The incident was created through the GLPI technician interface.

**Navigation:**

`Assistance → Tickets → Add`

The title was entered as:

`Internal Web Service Unavailable`

GLPI automatically assigned the ticket its unique identifier, Ticket #1.

The incident description outlined the simulated service outage and the effect it would have on internal users.

### 5.2 Initial Incident Description

The scenario assumed that an internal web application had become unavailable.

Users attempting to access the service would experience a connection error.

The initial investigation requirements included:

- Confirming the affected service and endpoint.
- Checking connectivity between the client and server.
- Inspecting the web service status.
- Reviewing operating system and application logs.
- Establishing whether the issue was isolated or widespread.

At this stage, these were proposed troubleshooting activities rather than completed technical checks.

### 5.3 Incident Priority

The incident was assigned High urgency and High impact.

GLPI calculated the overall priority as High.

**Urgency** represented the need to restore the affected service.

**Impact** represented the assumed disruption to users depending on the internal application.

For the simulated scenario, these values were chosen to demonstrate how service-affecting incidents could be prioritised.

They were not derived from real production user counts or business impact measurements.

## 6. First-Line Incident Response

### 6.1 Initial Assignment

The incident was initially handled by the First-line IT Operations Engineer using the `ops.l1` account.

After assignment, GLPI displayed the ticket status as:

**Processing (assigned)**

This indicated that the incident had been allocated for investigation.

### 6.2 Initial Investigation

An L1 investigation follow-up was recorded in the ticket.

The initial triage checklist covered:

1. Identify the affected application and reported symptoms.
2. Confirm the service endpoint.
3. Check basic network connectivity.
4. Review service availability.
5. Identify relevant system logs.
6. Determine whether escalation was necessary.

The investigation record clearly identified the work as a simulated exercise.

No unverified root cause was recorded at this stage.

### 6.3 Escalation Decision

The incident scenario required further investigation of the Linux web service.

The first-line support account therefore escalated the incident to Infrastructure Support.

This demonstrated an important operational principle: incidents should be transferred to the appropriate support function when additional technical investigation or specialist access is required.

## 7. Escalation to Infrastructure L2

### 7.1 Escalation Process

The incident was reassigned from the first-line engineer to the Infrastructure Support Engineer.

The intended ownership transfer was:

**IT Operations - L1 → Infrastructure - L2**

The `infra.l2` account was assigned to continue the investigation.

### 7.2 Escalation Documentation

A follow-up was recorded explaining why the incident required second-line support.

The escalation note identified:

- The reported web service outage.
- The need for additional server-level investigation.
- The requirement to inspect service configuration and logs.
- The absence of a confirmed technical root cause.
- The expected next actions for Infrastructure Support.

### 7.3 Why the Handover Matters

A documented escalation provides context to the receiving engineer.

Without a proper handover, L2 may need to repeat initial checks or investigate without understanding what has already been considered.

Recording the escalation in GLPI created a visible history of the incident's progression between support roles.

## 8. Technical Investigation

The technical investigation was performed on the Ubuntu Server VM.

The affected application was a local Nginx web service.

A controlled shutdown was used to reproduce the service-unavailable scenario.

### 8.1 Initial Service Verification

Nginx was installed using Ubuntu's package manager:

`sudo apt install -y nginx`

The service status was checked using:

`sudo systemctl status nginx`

The initial healthy service was also tested using:

`curl -I http://127.0.0.1`

The server returned:

`HTTP/1.1 200 OK`

This confirmed that Nginx was responding to HTTP requests before the controlled outage.

### 8.2 Simulating the Failure

The service was deliberately stopped using:

`sudo systemctl stop nginx`

The service status was then checked:

`sudo systemctl status nginx`

The output showed:

`Active: inactive (dead)`

This confirmed that Nginx was no longer running.

### 8.3 Inspecting System Logs

The Linux system journal was inspected using:

`sudo journalctl -u nginx --no-pager -n 30`

The recorded journal entries included:

- Stopping the Nginx service.
- Successful deactivation.
- Confirmation that the service had stopped.

The logs showed that Nginx had been stopped successfully.

They did not indicate an unexpected application crash.

### 8.4 Investigation Findings

The investigation identified the immediate cause of the simulated outage as the deliberate shutdown of the Nginx service.

This distinction was important.

An inactive service can result from several different causes, including an administrator action, configuration issue, failed startup or unexpected crash.

In this exercise, the recorded shutdown was intentional.

The evidence supported a controlled service stoppage rather than an unexpected Nginx failure.

## 9. Root Cause Analysis

### 9.1 Root Cause

**Root cause:** The Nginx service was deliberately stopped using `systemctl` during the incident simulation.

### 9.2 Supporting Evidence

The root cause was supported by:

- The executed service stop command.
- The resulting inactive service state.
- System journal entries documenting the shutdown.
- The successful service restart and HTTP recovery verification.

### 9.3 What the Investigation Did Not Establish

The exercise did not investigate an unexpected production failure.

There was no evidence of:

- A real application crash.
- Hardware failure.
- Network equipment failure.
- Malicious activity.
- Incorrect production deployment.

The purpose was to practise a controlled investigation and demonstrate how service status and logs can be used to support a technical conclusion.

### 9.4 Preventive Recommendations

Although the service was intentionally stopped, the exercise highlighted the value of monitoring critical application availability.

Recommended improvements included:

- Automated HTTP availability checks.
- Alerts when a monitored endpoint becomes unavailable.
- Timestamped monitoring records.
- Documented service recovery procedures.
- Appropriate change controls for planned service interruptions.

These recommendations informed the later monitoring change request.

## 10. Service Recovery

### 10.1 Recovery Action

The Infrastructure Support Engineer restored the affected service using:

`sudo systemctl start nginx`

The service was then checked again.

### 10.2 Recovery Verification

An HTTP request was performed using:

`curl -I http://127.0.0.1`

The server returned:

`HTTP/1.1 200 OK`

This confirmed that Nginx was responding again after the restart.

### 10.3 Verification Results

| Check | Result |
|---|---|
| Before outage | Nginx responding — HTTP 200 |
| During outage | Nginx inactive |
| Journal investigation | Successful service shutdown recorded |
| Recovery action | Nginx restarted |
| After recovery | HTTP 200 |
| Final service state | Operational |

The successful HTTP response demonstrated application-level availability, rather than relying only on the service's process status.

### 10.4 Recovery Limitations

The exercise did not establish a formal service-level agreement or measure complete end-user recovery.

It also did not calculate a reliable production downtime metric.

The available evidence confirmed that the local HTTP endpoint responded successfully after service restoration.

## 11. Recording the L2 Investigation

After completing the technical exercise, the findings were documented in GLPI Ticket #1.

The technical follow-up included:

- The environment being investigated.
- The affected Nginx service.
- Commands used during the investigation.
- The observed inactive service state.
- The journal evidence.
- The identified root cause.
- The recovery command.
- The successful HTTP response.
- Recommendations for preventive monitoring.

This documentation connected the technical activity performed on Ubuntu with the incident record maintained in GLPI.

## 12. Incident Resolution

### 12.1 Solution Recorded

The solution entered into GLPI explained that Nginx had been restored using:

`sudo systemctl start nginx`

The successful HTTP 200 response was recorded as recovery evidence.

The resolution also included the identified root cause and recommended follow-up actions.

### 12.2 Ticket Status

After the solution was submitted, the incident moved to:

**Solved**

This indicated that a technical resolution had been provided.

### 12.3 Solution Approval

The solution was subsequently reviewed and approved by the First-line IT Operations Engineer.

The GLPI timeline recorded the solution approval.

### 12.4 Incident Closure

Following approval, the ticket moved to:

**Closed**

The completed incident record contained the original description, investigation notes, escalation, technical resolution and solution approval.

This completed the simulated incident-management lifecycle.

## 13. Incident Workflow Summary

The completed workflow can be summarised as:

**1. Incident reported**

A simulated internal web service outage was recorded in GLPI.

**2. Incident prioritised**

Urgency and impact were assigned, producing a High-priority incident.

**3. L1 investigation**

First-line support documented the initial investigation checklist and assessed the need for escalation.

**4. Incident escalated**

Responsibility was transferred to the Infrastructure Support Engineer.

**5. Technical investigation**

Nginx service status and Linux journal entries were inspected during a controlled outage.

**6. Root cause identified**

The service was confirmed to have been deliberately stopped.

**7. Service restored**

Nginx was restarted successfully.

**8. Recovery verified**

The HTTP endpoint returned status code 200.

**9. Resolution documented**

The technical findings and recovery actions were recorded in GLPI.

**10. Incident closed**

The solution was approved and the ticket reached Closed status.

## 14. Verification and Testing

The completed exercise was evaluated against the following criteria.

| Test | Expected result | Outcome |
|---|---|---|
| Incident creation | GLPI generates ticket | Passed |
| Ticket prioritisation | High priority recorded | Passed |
| L1 assignment | First-line account assigned | Passed |
| Investigation documentation | Follow-up recorded | Passed |
| L2 escalation | Infrastructure engineer assigned | Passed |
| Service availability test | HTTP 200 before shutdown | Passed |
| Controlled service shutdown | Nginx becomes inactive | Passed |
| Service log investigation | Shutdown events available | Passed |
| Recovery action | Nginx restarts | Passed |
| Recovery verification | HTTP 200 after restart | Passed |
| L2 resolution | Technical solution recorded | Passed |
| Solution approval | L1 approves resolution | Passed |
| Ticket closure | Status becomes Closed | Passed |

The results confirmed that the incident workflow was functional and supported the required support, investigation and resolution activities.

## 15. Relationship to Change Management

The incident provided the basis for a subsequent change request.

**Change #1 — Implement Automated Nginx Health Monitoring**

The purpose of this change was to introduce automated availability monitoring so that future interruptions would be identified through monitoring checks rather than relying exclusively on manual investigation.

The proposed monitoring solution included:

- HTTP endpoint checks.
- System resource monitoring.
- Automated scheduled execution.
- Timestamped results.
- Failure and recovery detection.
- Local alert records.

The change was reviewed and approved through GLPI before implementation.

The implementation was subsequently developed and tested as part of the monitoring component of the project.

This demonstrated how an incident can lead to a planned improvement in operational processes.

## 16. Troubleshooting Lessons

### Service State Is Not the Same as Application Availability

Checking whether a service is active is useful, but an active process does not guarantee that an application is functioning correctly.

For this reason, the investigation included HTTP response verification.

### Logs Support Root Cause Analysis

System logs helped establish that Nginx had been intentionally stopped.

They supported a conclusion based on recorded events rather than assumptions.

### Escalation Requires Documentation

Recording why an incident was transferred helps the receiving support engineer understand the issue and continue the investigation.

### Resolution Requires Verification

Restarting a service is an action, not proof of successful recovery.

The HTTP 200 response was used to verify that the service was responding after the restart.

### Closing a Ticket Is a Separate Step

The incident moved through Solved and Closed states.

This demonstrated the distinction between providing a technical fix and completing the incident-management process.

## 17. Limitations

The incident simulation had several limitations:

- The affected application was a local test web service.
- The outage was deliberately introduced.
- No real business users were affected.
- The investigation involved a single Ubuntu VM.
- No complex distributed infrastructure was involved.
- The exercise did not include real production service-level targets.
- Downtime was not independently measured using a continuous availability-monitoring system during the original incident.
- The root cause was known from the controlled test procedure.

These limitations are consistent with the project's educational scope.

Further scenarios could be explored in a larger lab, but they are not required for the completed incident exercise.

## 18. Skills Demonstrated

The incident-management exercise provided practical experience in:

- Incident logging and categorisation.
- Urgency, impact and priority assessment.
- First-line technical triage.
- Ticket assignment and escalation.
- Linux service administration.
- System log investigation.
- HTTP availability testing.
- Root cause analysis.
- Service recovery and validation.
- Technical documentation.
- Solution approval and incident closure.
- Identifying opportunities for operational improvement.

## 19. Conclusion

The incident-management phase successfully demonstrated a complete service desk workflow, from initial incident creation through to technical recovery and closure.

GLPI was used to record and coordinate the incident, while Ubuntu and Nginx provided the environment for practical service troubleshooting.

The exercise demonstrated how first-line and second-line support responsibilities can be separated, how technical evidence can support root cause analysis, and why recovery should be verified before an incident is closed.

The completed incident also provided the justification for implementing automated health monitoring, linking the incident-management and change-management components of the lab.

---

**Related documentation**

- `01-infrastructure.md` — Ubuntu, VMware and Docker environment
- `02-user-management.md` — GLPI users, groups and permissions
- `04-change-management.md` — Monitoring change request and approval
- `05-monitoring-and-alerting.md` — Automated health monitoring
- `06-testing-and-results.md` — Overall testing and evaluation
