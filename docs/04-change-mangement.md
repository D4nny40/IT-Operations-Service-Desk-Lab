# 04 — Change Management and Implementation

## 1. Overview

This document covers the change management process implemented within the IT Operations Service Desk Lab.

The objective was to demonstrate how an infrastructure improvement could be proposed, assessed, approved, implemented and tested using an IT service management platform.

GLPI was used to manage the change request, record supporting documentation and demonstrate the approval process.

The change was created following a simulated Nginx service outage investigated during the incident management phase.

The proposed improvement was to introduce automated Nginx health monitoring using Python.

## 2. Objectives

The main objectives were to:

- Create a formal change request in GLPI.
- Provide a business justification for the change.
- Assess the expected impact and implementation risks.
- Document an implementation and testing plan.
- Prepare a rollback procedure.
- Configure and test a change approval workflow.
- Separate the change requester from the approving function.
- Implement the approved monitoring solution.
- Record test results and supporting technical evidence.

## 3. Change Request Summary

| Field | Details |
|---|---|
| Change ID | GLPI Change #1 |
| Title | Implement Automated Nginx Health Monitoring |
| Related incident | GLPI Ticket #1 — Internal Web Service Unavailable |
| Environment | Ubuntu Server 26.04.1 LTS |
| Change type | Planned infrastructure improvement |
| Urgency | Medium |
| Impact | Low |
| Calculated priority | Low |
| Implementation responsibility | IT Operations Administrator |
| Approval function | Change Management |
| Approval result | Granted |
| Implementation | Monitoring solution developed and tested |

**Change classification:** Controlled educational lab exercise.

The change was performed on a local virtual machine rather than a production system.

## 4. Background and Business Justification

During Incident #1, an Nginx web service was deliberately stopped to simulate an application outage.

The incident was investigated through GLPI, escalated from first-line to second-line support, and resolved by restarting Nginx.

Service recovery was verified using a successful HTTP 200 response.

Although the incident was resolved, the exercise highlighted a limitation in the environment: Nginx availability was initially checked manually.

A service could therefore become unavailable without being detected until an administrator performed a check.

### Proposed Improvement

Develop and deploy an automated monitoring solution capable of regularly checking Nginx availability.

The monitoring system should record successful checks, detect failures and produce evidence that can support incident investigations.

### Expected Benefits

- Earlier identification of service outages.
- Reduced reliance on manual availability checks.
- Timestamped evidence of service health.
- Improved support for troubleshooting and incident response.
- Consistent identification of service recovery.
- Better visibility of server resource utilisation.

The aim was not to build a full enterprise monitoring platform, but to introduce a lightweight monitoring capability appropriate for the lab.

## 5. Creating the Change Request

The change was created within GLPI using:

`Assistance → Changes → Add`

The request was titled:

`Implement Automated Nginx Health Monitoring`

The description included the business justification, proposed monitoring functionality, implementation scope and expected outcome.

### Initial Classification

The change was assigned Medium urgency and Low impact.

GLPI calculated the overall priority as Low.

This was considered appropriate because the proposed monitoring system was a planned improvement in a controlled environment rather than an emergency production repair.

## 6. Scope of the Change

The approved implementation focused on monitoring the local Nginx service and selected Ubuntu system resources.

### Included

- HTTP availability monitoring.
- HTTP response-time measurements.
- CPU utilisation monitoring.
- Memory utilisation monitoring.
- Disk utilisation monitoring.
- Structured JSONL logging.
- Detection of service failures and recoveries.
- Suppression of repeated alerts for unchanged problems.
- Automated monitoring through systemd.
- Local notification records through the Linux journal.

### Excluded

- Modification of the GLPI application code.
- Modification of the MySQL database.
- Enterprise monitoring dashboards.
- Monitoring across multiple physical servers.
- External messaging integrations.
- Automated remediation of failed services.

These exclusions helped keep the implementation focused and reduced the likelihood of disrupting existing components.

## 7. Impact Assessment

The monitoring solution was designed to run locally on the existing Ubuntu Server virtual machine.

It makes periodic HTTP requests to the Nginx test endpoint and collects operating system resource statistics.

### Expected Impact

| Component | Expected impact |
|---|---|
| Nginx | Small periodic HTTP requests |
| Ubuntu Server | Minor CPU, memory and disk activity |
| GLPI | No planned configuration changes |
| MySQL | No planned database changes |
| Network | Minimal loopback HTTP traffic |
| Storage | Gradually increasing monitoring logs |

The overall impact was assessed as Low for the lab.

### Operational Considerations

The monitoring script should not interrupt or modify the service it is checking.

The implementation should use reasonable execution intervals and avoid requiring elevated privileges for ordinary monitoring operations.

It should also be possible to stop the monitor without disrupting Nginx, GLPI or MySQL.

## 8. Risk Assessment

The main risks were documented before implementation.

| Risk | Potential consequence | Mitigation |
|---|---|---|
| False-positive alerts | Unnecessary investigations | Controlled testing and clear thresholds |
| Excessive monitoring frequency | Additional resource usage | Schedule checks at reasonable intervals |
| Python execution errors | Missed monitoring results | Syntax checks and test executions |
| Scheduler failure | Monitoring stops running | Verify systemd timer and journal |
| Incorrect thresholds | Unhelpful resource warnings | Use documented configurable thresholds |
| Log growth | Increased disk usage | Review logs and consider rotation |
| Configuration changes | Unexpected service disruption | Keep GLPI and MySQL configuration unchanged |

### Risk Classification

**Overall assessment:** Low, within the controlled lab environment.

This assessment is specific to the project. A production deployment would require further consideration of service criticality, security, monitoring coverage, notification reliability and operational dependencies.

## 9. Implementation Plan

The implementation was divided into manageable stages.

### Stage 1 — Prepare the Environment

Create a dedicated directory for the monitoring solution.

`/home/itops/it-operations/health-checker/`

Verify that Python and the Nginx test service are available.

### Stage 2 — Implement HTTP Monitoring

Develop `health_checker.py` to:

- Send HTTP requests to the Nginx endpoint.
- Record response status and response time.
- Handle connection errors.
- Return a health status.
- Save timestamped monitoring results.

### Stage 3 — Implement Resource Monitoring

Develop `system_metrics.py` to collect:

- CPU usage.
- Memory usage.
- Disk usage.

Compare the readings against predefined warning thresholds.

### Stage 4 — Integrate Monitoring

Develop `monitor.py` to combine the service and system checks.

Use the results to determine an overall monitoring status.

### Stage 5 — Automate Execution

Configure a systemd service and timer to execute the monitoring application approximately once per minute.

### Stage 6 — Implement Local Alerting

Develop alerting functionality to identify newly detected problems and recovery events.

Store alert state between executions to reduce duplicate notifications.

Record significant events in the Linux system journal.

### Stage 7 — Validate the Implementation

Perform controlled tests to verify healthy operation, outage detection, recovery detection and automatic execution.

## 10. Rollback Plan

A rollback procedure was documented before implementation.

If the monitoring solution caused unexpected behaviour, the following actions could be taken.

### Stop Scheduled Monitoring

`sudo systemctl stop itops-monitor.timer`

### Disable Automatic Execution

`sudo systemctl disable itops-monitor.timer`

### Verify Services

Check the Nginx service:

`sudo systemctl status nginx`

Check the GLPI containers:

`cd ~/it-operations/glpi`

`sudo docker compose ps`

### Investigate Monitoring Errors

Review the monitoring service logs:

`sudo journalctl -u itops-monitor.service --no-pager -n 40`

The monitoring scripts could be corrected or disabled while preserving the existing application environment.

The rollback plan was documented; a complete rollback exercise was not separately verified.

## 11. Change Approval Workflow

The GLPI approval process was configured to demonstrate separation of duties.

### Approval Structure

| Responsibility | Account or group |
|---|---|
| Operational requester | IT Operations Administrator |
| Requested approver | Change Management group |
| Recorded decision | Change Manager |

### Approval Request

The approval request was submitted for the proposed monitoring implementation.

The supporting change record contained the business justification, risk assessment, impact assessment and implementation plan.

### Approval Permission Configuration

The custom Change Manager profile was reviewed through:

`Administration → Profiles → Change Manager → Assistance`

The **Validate** permission for change approvals was confirmed as enabled.

### Approval Decision

The Change Manager reviewed the proposed change and recorded an approval response.

GLPI displayed:

**Global approval status: Granted**

The approval history also identified the requester and approving function.

This demonstrated a formal authorisation process before implementation.

## 12. Approval Troubleshooting

During the approval exercise, the request initially remained in the Waiting for approval state.

### Issue Encountered

Selecting the pencil icon beside the approval record opened an editor for the approval request rather than the decision interface.

The approval could not be completed through that control.

### Investigation

The following were reviewed:

- Active GLPI user account.
- Change Manager profile.
- Change approval Validate permission.
- Change Management group membership.
- Existing approval request.
- Change approval interface and timeline.

### Outcome

The approval was eventually granted through the change-management workflow.

The recorded result was confirmed by the global approval status.

### Lesson Learned

Creating or editing an approval request is not necessarily the same as responding to it.

The available actions depend on the user's authorisation, the target approver and the application's workflow.

## 13. Implementation Summary

Following approval, the Python monitoring solution was developed and tested on Ubuntu.

The implementation consisted of five main Python modules:

| Module | Responsibility |
|---|---|
| `health_checker.py` | HTTP availability and response-time checks |
| `system_metrics.py` | CPU, memory and disk monitoring |
| `monitor.py` | Combined execution and status reporting |
| `alerting.py` | New problem and recovery detection |
| `notifications.py` | Local system journal notifications |

The monitor was configured to execute automatically using systemd.

The implementation was kept separate from the GLPI and MySQL containers.

## 14. Validation and Test Results

The following tests were performed.

| Test | Expected result | Outcome |
|---|---|---|
| Healthy Nginx | HTTP 200 and UP status | Passed |
| Nginx stopped | DOWN and CRITICAL status | Passed |
| Nginx restored | UP and OK status | Passed |
| CPU monitoring | CPU percentage recorded | Passed |
| Memory monitoring | Memory percentage recorded | Passed |
| Disk monitoring | Disk percentage recorded | Passed |
| JSONL logging | Timestamped result saved | Passed |
| systemd scheduling | Automatic executions recorded | Passed |
| Failure alert | TRIGGERED event recorded | Passed |
| Recovery alert | RECOVERED event recorded | Passed |
| Repeated outage checks | Duplicate alerts suppressed | Passed |
| Linux journal notification | Trigger and recovery entries logged | Passed |

The completed tests demonstrated that the monitoring solution met the main functional objectives of the change.

## 15. Implementation Outcome

The monitoring solution was successfully implemented and tested within the Ubuntu lab.

It provided automated checks, structured monitoring results, outage detection and local alert records.

The implementation also demonstrated that a planned infrastructure improvement could be connected to an earlier incident-management exercise.

### Change Record Status

Formal approval was verified as Granted.

The implementation and testing were completed, but the final Closed status of the GLPI change record was not independently confirmed in the available screenshots.

The change record should only be documented as Closed once that status has been verified in GLPI.

## 16. Lessons Learned

### Change Approval Should Precede Implementation

The monitoring solution was proposed and assessed before the implementation exercise proceeded.

This demonstrated the purpose of change control in reducing operational risk.

### Risk Assessment Should Be Proportionate

The environment was a local lab, so the change was assessed as low impact.

However, the assessment still considered resource usage, false alerts, execution failures and rollback requirements.

### Testing Is Required After Implementation

Installing a monitoring script does not prove that it will detect failures.

Controlled outage and recovery tests were therefore performed to validate the implementation.

### Changes Need Supporting Evidence

GLPI records explained the proposed change, while terminal results, systemd logs and JSONL files demonstrated the actual implementation.

### Approval and Closure Are Separate

An approved change is not automatically a completed change.

The implementation must be performed and validated before final review and closure.

## 17. Limitations

The change-management exercise had several limitations:

- It was performed within a single virtualised lab.
- The users and approval roles were simulated.
- The monitoring solution initially covered one local HTTP service.
- No enterprise change advisory board was involved.
- No external production users were affected.
- Formal maintenance-window coordination was not tested across multiple services.
- A full rollback exercise was not demonstrated.
- Final GLPI change closure requires separate verification.

These limitations are consistent with the educational purpose of the project.

## 18. Skills Demonstrated

This stage provided practical experience in:

- IT change management.
- Business justification.
- Risk and impact assessment.
- Deployment planning.
- Rollback planning.
- Change approval workflows.
- Separation of duties.
- Linux service administration.
- Python monitoring deployment.
- Technical testing and validation.
- Implementation documentation.

## 19. Conclusion

The change-management phase demonstrated how an incident can result in a planned infrastructure improvement.

Following the Nginx outage exercise, a monitoring solution was proposed, assessed and approved through GLPI.

The approved solution was subsequently developed and tested on Ubuntu.

The completed work demonstrated the relationship between incident management, change control, technical implementation and operational validation.

---

**Related documentation**

- `01-infrastructure.md` — Infrastructure setup
- `02-user-management.md` — Roles and permissions
- `03-incident-management.md` — Nginx incident investigation
- `05-monitoring-and-alerting.md` — Monitoring implementation
- `06-testing-and-results.md` — Overall evaluation
