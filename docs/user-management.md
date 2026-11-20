# 02 — User Management and Access Control

## 1. Overview

This document covers the user management, support group configuration and role-based access control (RBAC) implemented within the IT Operations Service Desk Lab.

The objective was to configure a realistic internal IT support structure using GLPI, with separate responsibilities for first-line support, infrastructure support, administration and change management.

Rather than using a single administrator account for every activity, dedicated accounts were created and assigned permissions based on their intended responsibilities.

This configuration supported the incident escalation and change approval exercises conducted later in the project.

## 2. Objectives

The main objectives of this stage were to:

- Secure the default GLPI user accounts.
- Create dedicated accounts for different IT Operations responsibilities.
- Configure first-line and second-line support groups.
- Implement role-based access control through GLPI profiles.
- Separate general administration from technician activities.
- Configure a dedicated Change Manager profile.
- Test permissions using actual incident and change-management workflows.
- Demonstrate incident escalation and change approval between separate accounts.

## 3. User Account Configuration

Following the initial GLPI deployment, four dedicated user accounts were created.

### 3.1 IT Operations Accounts

| Username | Role | Responsibility |
|---|---|---|
| `ops.admin` | IT Operations Administrator | Administration, configuration and operational oversight |
| `ops.l1` | First-line IT Operations Engineer | Incident logging, initial investigation and escalation |
| `infra.l2` | Infrastructure Support Engineer | Advanced troubleshooting, service recovery and technical resolution |
| `change.manager` | Change Manager | Change assessment, approval and review |

Each account was created within GLPI's user management interface.

**Navigation:**

`Administration → Users`

The accounts were configured with separate authorisations to support different activities within the lab.

### 3.2 Why Separate Accounts?

In an IT Operations environment, different responsibilities require different levels of access.

A first-line engineer may need to create incidents and assign tickets, but does not necessarily require access to platform-wide configuration.

An infrastructure engineer may require technical investigation permissions, while a Change Manager needs to assess and approve changes.

Separating these responsibilities supports accountability and reduces unnecessary access.

It also makes incident histories easier to understand because actions can be associated with the accounts that performed them.

## 4. Securing Default GLPI Accounts

After GLPI was installed, several accounts were available with publicly documented default credentials.

Leaving these accounts enabled with unchanged credentials would introduce an unnecessary security risk.

### 4.1 Default Accounts

| Account | Action |
|---|---|
| `glpi` | Retained for system administration; password changed |
| `tech` | Disabled |
| `normal` | Disabled |
| `post-only` | Disabled |
| `glpi-system` | Retained as a system account |

The unused default accounts were deactivated rather than deleted.

### 4.2 Deactivation Procedure

The accounts were managed through:

`Administration → Users`

For each unused account:

1. Open the user record.
2. Locate the **Active** setting.
3. Change the value from Yes to No.
4. Save the configuration.
5. Return to the user list to verify the result.

Disabling these accounts prevents their normal use while retaining their records.

### 4.3 Security Rationale

Deactivating unused accounts follows two important security principles.

**Attack surface reduction**

Unused accounts represent additional possible entry points into an application. Disabling unnecessary accounts reduces that exposure.

**Account lifecycle management**

Accounts should be created, modified and deactivated according to operational requirements.

The exercise demonstrated that user provisioning includes both granting access and removing access when it is no longer required.

## 5. Support Group Configuration

Three support groups were created to represent different responsibilities within the IT Operations environment.

### 5.1 IT Operations — L1

**Group name:** `IT Operations - L1`

**Purpose:** First-line incident response and initial troubleshooting.

**Members:**
- `ops.l1`
- `ops.admin`

This group was responsible for receiving incidents, assessing their impact, recording initial investigations and escalating issues that required further technical support.

### 5.2 Infrastructure — L2

**Group name:** `Infrastructure - L2`

**Purpose:** Advanced technical investigation and infrastructure support.

**Members:**
- `infra.l2`
- `ops.admin`

The group was responsible for investigating escalated incidents, examining server and application behaviour, implementing recovery procedures and documenting technical resolutions.

### 5.3 Change Management

**Group name:** `Change Management`

**Purpose:** Review and approval of proposed infrastructure changes.

**Member:**
- `change.manager`

This group was responsible for reviewing documented changes and recording approval decisions before implementation.

### 5.4 Group Structure

The support model followed this general structure:

**IT Operations Administrator**

Operational oversight and administration across support functions.

**IT Operations — L1**

Receives incidents, records initial investigations and performs basic troubleshooting.

**Infrastructure — L2**

Receives escalated incidents and performs more detailed technical investigations.

**Change Management**

Assesses proposed changes and authorises implementation when the required conditions are satisfied.

The Change Management group is not a mandatory escalation destination for every incident. It is used when proposed modifications require formal assessment or approval.

## 6. Role-Based Access Control

GLPI uses profiles to determine the actions users are authorised to perform.

Groups and profiles serve different purposes:

| Component | Purpose |
|---|---|
| Users | Represent individual accounts |
| Groups | Organise users into support teams |
| Profiles | Define available permissions |
| Authorisations | Associate profiles and entities with users |
| Entities | Define administrative and organisational scope |

A user can belong to multiple groups and may hold multiple profiles.

This distinction was important during configuration because group membership alone did not guarantee that a user could perform a specific ticket operation.

### 6.1 Administrator Profile

The `ops.admin` account was configured with the Admin profile.

Its purpose was to support operational configuration, user administration and supervision of the support environment.

The original `glpi` account retained higher-level administration access.

### 6.2 Technician Profile

The `ops.l1` account was assigned the Technician profile.

This provided access to GLPI's standard technician interface, including incident-management functions.

The Technician profile was also used for infrastructure-support activities.

Relevant ticket permissions included the ability to work with assigned incidents, record follow-ups and perform assignment operations where authorised.

### 6.3 Self-Service Profile

Several accounts initially had Self-Service authorisations alongside their operational profiles.

This created an issue during testing.

When `ops.l1` logged into GLPI, the application initially displayed the Self-Service interface instead of the Technician interface.

The Self-Service view provided a simplified request interface and did not expose the expected technician navigation.

**Resolution:**

The active profile was changed from Self-Service to Technician through the user account menu.

After switching profiles, the standard GLPI interface became available.

This demonstrated that a user's active profile affects the features and menus presented during a session.

The distinction between an available profile, an active profile and a user's default profile is important when troubleshooting access issues.

## 7. Custom Change Manager Profile

A custom profile named `Change Manager` was configured.

The purpose was to separate change-management responsibilities from general IT administration.

The custom profile was used by the `change.manager` account during the change approval exercise.

### 7.1 Required Capabilities

The intended Change Manager responsibilities included:

- Viewing change requests.
- Reviewing change descriptions.
- Assessing business impact and operational risk.
- Reviewing implementation and rollback plans.
- Responding to change approval requests.
- Recording approval decisions.

### 7.2 Change Approval Permissions

During configuration, the **Validate** permission for change approvals was confirmed as enabled.

This was an important requirement for responding to change approval requests.

The profile configuration was accessed through:

`Administration → Profiles → Change Manager → Assistance`

The change approval exercise subsequently confirmed that an approval decision could be recorded using the Change Manager account.

### 7.3 Least-Privilege Considerations

During profile review, destructive permissions such as Purge were also observed as enabled.

These permissions are not necessary for ordinary change approval duties.

A recommended security refinement is to review the custom profile and disable unnecessary destructive actions.

This was identified as a least-privilege improvement; the available evidence does not establish that every unnecessary permission was subsequently removed.

## 8. Ticket Assignment and Escalation

The configured accounts and groups were tested during the simulated Nginx service outage.

### 8.1 Initial Incident Assignment

An incident titled:

`Internal Web Service Unavailable`

was created in GLPI as Ticket #1.

The incident was initially handled by the First-line IT Operations Engineer.

The ticket was configured with High urgency and High impact, resulting in a High calculated priority.

### 8.2 L1 Investigation

The first-line account documented the initial incident assessment.

This included identifying the reported service outage, outlining troubleshooting checks and recording the need for additional technical investigation.

### 8.3 Escalation to L2

The incident was subsequently escalated to Infrastructure Support.

The Infrastructure Support Engineer account was assigned to continue the technical investigation.

The escalation reason was recorded in GLPI, providing context for the handover.

### 8.4 L2 Resolution

The infrastructure engineer investigated the simulated Nginx outage using Linux service-management commands and system logs.

The Nginx service was restored, and its availability was verified through a successful HTTP response.

The investigation and resolution were documented in the ticket.

### 8.5 Closure

The solution was reviewed and approved by the First-line IT Operations Engineer.

Ticket #1 was subsequently marked Closed.

This demonstrated a complete workflow involving multiple accounts with different operational responsibilities.

## 9. Change Approval Workflow

The configured Change Management group was tested using:

`Change #1 — Implement Automated Nginx Health Monitoring`

The change request was created following the Nginx incident simulation.

It proposed introducing automated monitoring to identify future service outages.

The request contained:

- Business justification.
- Proposed implementation scope.
- Impact assessment.
- Risk assessment.
- Deployment plan.
- Rollback procedure.

### 9.1 Approval Request

An approval request was submitted by the IT Operations Administrator.

The requested approver was the Change Management group.

This established a separation between the operational requester and the approving function.

### 9.2 Approval Decision

The Change Manager account reviewed the request and recorded an approval decision.

The final approval status displayed by GLPI was:

**Granted**

The approval history recorded the requester, approver and decision.

This demonstrated that the configured group and profile permissions supported the intended workflow.

### 9.3 Why Approval Separation Matters

Separating change submission from approval helps ensure that proposed changes receive an independent review.

The approver can examine risks, potential impact, implementation steps and rollback arrangements before authorising work.

In the lab, this process was simulated using dedicated GLPI accounts.

## 10. Troubleshooting and Issues Encountered

Several access-control and user-management issues were encountered during the configuration process.

### Issue 1 — Technician Interface Not Displayed

**Problem:** The first-line engineer account opened GLPI using the Self-Service interface.

**Cause:** The account had multiple authorised profiles and Self-Service was the active profile.

**Resolution:** The active profile was changed to Technician.

**Lesson learned:** Profile selection affects which interface and actions are available to a user.

### Issue 2 — Unable to Assign Tickets

**Problem:** Ticket assignment options initially appeared limited or unavailable.

**Investigation:** Technician permissions, group assignment settings and the ticket's assignment controls were reviewed.

**Outcome:** The technician assignment interface was subsequently used successfully, including assignment to the Infrastructure Support Engineer.

**Lesson learned:** Ticket creation, ticket ownership and reassignment are separate operational capabilities that depend on the relevant authorisations and assignment settings.

### Issue 3 — Change Approval Controls Not Immediately Available

**Problem:** A change approval request remained in Waiting for approval, and editing the approval record did not expose a decision control.

**Investigation:** The Change Manager profile was reviewed, including its Validate permission and approval group configuration.

**Outcome:** The approval was eventually granted and confirmed in GLPI's global approval status.

**Lesson learned:** Editing an approval request and responding to an approval request are distinct actions. Troubleshooting must account for the active user, profile, group and workflow.

## 11. Testing and Verification

The following access-control and workflow tests were performed.

| Test | Expected result | Outcome |
|---|---|---|
| Administrator authentication | Successful login | Passed |
| Disable unused default accounts | Accounts marked inactive | Passed |
| Create dedicated operational accounts | Accounts available in GLPI | Passed |
| Create L1 support group | Group available | Passed |
| Create L2 support group | Group available | Passed |
| Create Change Management group | Group available | Passed |
| Assign users to groups | Memberships recorded | Passed |
| Technician profile selection | Standard interface accessible | Passed |
| Incident creation | Ticket generated | Passed |
| L1-to-L2 incident escalation | Support responsibility transferred | Passed |
| L2 resolution documentation | Investigation and solution recorded | Passed |
| Solution approval | Incident approved and closed | Passed |
| Change approval | Global approval status Granted | Passed |

These tests confirmed that the configured accounts and support structure were capable of supporting the primary incident and change-management exercises.

## 12. Security Evaluation

The configuration demonstrated several useful access-control principles.

### Account Separation

Dedicated user accounts were created for administration, first-line support, infrastructure support and change management.

This allowed operational actions to be associated with different accounts.

### Least Privilege

Different profiles were used to separate technician and change-management responsibilities.

Further permission refinement was identified as an improvement, particularly for removing unnecessary destructive permissions.

### Default Account Security

Unused default accounts were disabled and the main GLPI administrator credentials were secured.

### Accountability

Incident and change histories recorded actions associated with the accounts participating in each workflow.

### Separation of Duties

The change approval exercise used separate requester and approver roles to demonstrate the review process.

## 13. Limitations

Although the access-control configuration supported the lab objectives, several limitations remain.

- The environment uses a small number of simulated users.
- The accounts represent roles rather than separate real employees.
- The lab does not use an external identity provider.
- Multi-factor authentication was not demonstrated.
- Fine-grained permissions require further security review.
- The environment does not include a full employee onboarding or offboarding process.
- Group and role configurations were tested primarily through a small number of incident and change scenarios.

These limitations are consistent with the project's scope as an educational IT Operations environment.

## 14. Skills Demonstrated

This stage provided practical experience in:

- User account administration.
- Account deactivation and security.
- Role-based access control.
- Permission troubleshooting.
- Support group configuration.
- Incident assignment and escalation.
- Technician profile management.
- Change approval configuration.
- Separation of duties.
- IT service management workflows.

## 15. Conclusion

The user management and access-control stage established a structured IT Operations support environment within GLPI.

Dedicated accounts, support groups and profiles were configured to represent different operational responsibilities.

The configuration was tested through incident creation, L1-to-L2 escalation, resolution approval and formal change authorisation.

The troubleshooting performed during this stage also demonstrated the importance of understanding the relationship between account permissions, group memberships and active profiles.

Overall, the completed configuration provided a functional foundation for the incident-management and change-management activities performed throughout the lab.

---

**Related documentation**

- `01-infrastructure.md` — Infrastructure deployment and configuration
- `03-incident-management.md` — Incident #1 and service recovery
- `04-change-management.md` — Change request and approval workflow
- `05-monitoring-and-alerting.md` — Python monitoring implementation
- `06-testing-and-results.md` — Project testing and evaluation
