# IT Operations Service Desk Lab

A virtualised IT Operations environment designed to demonstrate practical skills in incident management, change control, infrastructure administration, and IT service management (ITSM).

## Project Overview

This project focuses on building and managing an internal IT service desk environment using VMware Workstation, Ubuntu Server, and GLPI.

The aim is to simulate common responsibilities of an IT Operations Engineer, including responding to incidents, investigating technical problems, managing service requests, and documenting infrastructure changes.

Rather than developing a ticketing application from scratch, the project uses an existing open-source ITSM platform to focus on real-world operational processes.

**Project Status:** In Development

## Objectives

- Deploy and configure a virtualised IT service desk.
- Implement incident logging, categorisation, and prioritisation.
- Configure support groups, ticket assignment, and escalation workflows.
- Manage change requests and planned infrastructure maintenance.
- Simulate IT incidents and document troubleshooting procedures.
- Create technical documentation and incident reports.
- Gain practical experience administering Linux-based infrastructure.

## Technology Stack

| Technology              | Purpose                             |
| ----------------------- | ----------------------------------- |
| VMware Workstation      | Virtualisation platform             |
| Ubuntu Server 24.04 LTS | Server operating system             |
| GLPI                    | IT service management and ticketing |
| Docker & Docker Compose | Application deployment              |
| MySQL                   | GLPI database                       |
| SSH                     | Remote server administration        |
| Git & GitHub            | Version control and documentation   |

## Planned Architecture

The initial environment will consist of a single Ubuntu Server virtual machine running GLPI and its database through Docker containers.

**Host Machine:** Windows PC running VMware Workstation

**Virtual Machine:** ITOPS-01

**Guest OS:** Ubuntu Server 24.04 LTS

**VM Configuration:**

- 2 virtual CPU cores
- 4 GB RAM
- 40 GB virtual storage
- NAT networking

The architecture may expand to include additional Windows and Linux machines for incident simulations.

## Planned Features

### Incident Management

- Ticket creation and categorisation
- Incident severity and priority assignment
- Ticket ownership and escalation
- Troubleshooting notes and resolution tracking

### Change Management

- Planned maintenance requests
- Risk and impact assessments
- Change approval workflows
- Rollback and recovery documentation

### IT Operations

- Server administration
- Service availability checks
- Incident investigation
- Root cause analysis
- Operational documentation

## Project Roadmap

- [x] Phase 1: Install and configure Ubuntu Server in VMware
- [X] Phase 2: Install Docker and deploy GLPI
- [X] Phase 3: Configure users, roles, and support groups
- [X] Phase 4: Implement incident management workflows
- [X] Phase 5: Configure change management procedures
- [ ] Phase 6: Simulate incidents and document resolutions
- [ ] Phase 7: Complete technical documentation


## Disclaimer

This is an independent educational and portfolio project. It is not affiliated with or endorsed by any employer or software vendor
