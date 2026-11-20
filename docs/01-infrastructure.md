# 01 — Infrastructure Setup and Configuration

## 1. Overview

This document covers the infrastructure setup and configuration for the IT Operations Service Desk Lab.

The aim was to build a virtualised environment capable of supporting an IT service management (ITSM) platform, allowing incident management, change management and infrastructure troubleshooting to be practised in a controlled environment.

The lab was deployed using VMware Workstation with Ubuntu Server as the main operating system. Docker was used to host GLPI and its MySQL database, while SSH provided remote administration.

A separate Nginx web service was also installed on Ubuntu for service availability and incident response testing.

## 2. Technologies Used

| Technology | Purpose |
|---|---|
| VMware Workstation | Virtual machine management |
| Ubuntu Server 26.04.1 LTS | Main server operating system |
| Docker Engine | Container runtime |
| Docker Compose | Multi-container application deployment |
| GLPI | IT service management platform |
| MySQL | Database for GLPI |
| OpenSSH | Secure remote administration |
| UFW | Host-based firewall |
| Nginx | Web service used for incident simulations |
| Python 3 | Supporting monitoring applications |
| systemd | Service management and scheduling |

### Why these technologies?

**VMware Workstation** was selected because it allowed the infrastructure to be created locally without requiring additional physical hardware. It also provides control over resource allocation, networking and virtual machine configuration.

**Ubuntu Server** was selected because Linux is commonly used in server environments. Working through the command line provided practical experience with package management, networking, permissions, service management and troubleshooting.

**Docker** was used to separate the GLPI application and its database into containers, making deployment and ongoing service management more straightforward.

**GLPI** was chosen because it provides an existing ITSM environment with ticket management, support groups, user profiles, incident tracking and change approval functionality.

## 3. Virtual Machine Configuration

The environment was created using VMware Workstation on a Windows host.

### Server details

| Setting | Configuration |
|---|---|
| Virtual machine name | IT Operations Lab |
| Hostname | `itops1` |
| Operating system | Ubuntu Server 26.04.1 LTS |
| Administration account | `itops` |
| Network adapter | NAT |
| Observed IPv4 address | `192.168.212.130` |
| Remote management | SSH, port 22 |
| GLPI web access | SSH port forwarding |

The initial VM resource plan was 2 virtual CPU cores, 4 GB RAM and 40 GB storage. Actual resource allocation should be verified against VMware settings before recording the final specifications, as the running VM may differ from the original plan.

### Networking

NAT networking was selected to provide internet access through the Windows host without directly exposing the VM as an independently addressed device on the physical network.

The VM received an IPv4 address from VMware's virtual network.

The following commands were used to inspect networking:

`hostname -I`

`ip -4 addr show`

Network connectivity was verified using:

`ping -c 4 1.1.1.1`

`ping -c 4 ubuntu.com`

The first test checks IP connectivity, while the second also checks DNS resolution.

## 4. Ubuntu Server Installation

Ubuntu Server was installed using an ISO image attached to the VMware virtual machine.

The installation included:

- Standard Ubuntu Server installation.
- Automatic network configuration using DHCP.
- Creation of a dedicated administration account.
- Installation of OpenSSH Server.
- Installation of operating system updates.
- Installation of common diagnostic utilities.

Following installation, the package manager was updated:

`sudo apt update`

`sudo apt upgrade -y`

Additional utilities were installed for troubleshooting and administration, including `curl`, `wget`, `git` and `htop`.

### Verification

The operating system version was checked using:

`cat /etc/os-release`

The output confirmed Ubuntu Server 26.04.1 LTS.

## 5. Remote Administration Using SSH

OpenSSH was configured during the Ubuntu installation.

SSH allows the server to be administered remotely from Windows without requiring direct access to the VMware console.

### Checking the SSH service

`sudo systemctl status ssh`

The service was confirmed to be active.

### Connecting from Windows

The Windows workstation connected using:

`ssh itops@192.168.212.130`

Successful authentication confirmed that remote administration was working.

### Why SSH?

SSH provides an encrypted remote connection and allows routine administrative operations to be performed through the command line.

It was also used to securely forward access to GLPI's web interface.

Password-based authentication was used during the initial lab setup. SSH key-based authentication is a potential future security improvement.

## 6. Firewall Configuration

Ubuntu's Uncomplicated Firewall (UFW) was configured to restrict incoming connections.

The firewall configuration was based on denying unsolicited incoming traffic while allowing necessary outbound communication.

### Commands used

Allow SSH:

`sudo ufw allow OpenSSH`

Enable the firewall:

`sudo ufw enable`

Verify the configuration:

`sudo ufw status verbose`

### Configuration summary

| Traffic | Policy |
|---|---|
| Incoming connections | Deny by default |
| Outgoing connections | Allow by default |
| SSH (TCP 22) | Allowed |
| GLPI web interface | Restricted to loopback access through Docker |
| Nginx test service | Used locally for monitoring tests |

The firewall was configured to permit SSH before enabling UFW to avoid unintentionally blocking remote administration.

### Security considerations

Docker-published ports can behave differently from ordinary host services when UFW is enabled.

To avoid exposing the GLPI web application unnecessarily, the Docker deployment was configured to bind its web port to Ubuntu's loopback interface.

This allowed GLPI to remain accessible through SSH forwarding without publishing its web interface directly on the VM's network address.

## 7. Docker Engine and Docker Compose

Docker Engine and Docker Compose were installed from Docker's official Ubuntu package repository.

Docker provides the container runtime used to execute GLPI and MySQL.

Docker Compose was selected to manage the application and database as a single deployment.

### Verification commands

`docker --version`

`docker compose version`

`sudo systemctl status docker`

The Docker service was confirmed to be running.

A basic test container was executed using:

`sudo docker run --rm hello-world`

The successful Hello World message confirmed that Docker could download and run container images.

### Why containerisation?

Using Docker provided several advantages:

- Application and database processes are separated.
- Configuration can be documented using a Compose file.
- Services can be started and stopped together.
- Persistent volumes retain application data.
- Recreating containers does not inherently require reinstalling the operating system.

Containerisation also provided practical experience with technologies commonly used in infrastructure operations.

## 8. GLPI and MySQL Deployment

GLPI was deployed using Docker Compose alongside a MySQL database.

The deployment was organised inside:

`/home/itops/it-operations/glpi/`

The configuration included:

- A GLPI application container.
- A MySQL database container.
- Docker-managed persistent storage.
- A dedicated Docker network.
- Environment variables for database configuration.

### Database configuration

The environment file contained configuration variables for the database host, port, name, username and password.

These were referenced by Docker Compose rather than placing database passwords directly into the Compose configuration.

The environment file contains sensitive credentials and must not be uploaded to a public GitHub repository.

### Persistent storage

Docker volumes were used to retain the GLPI application data and MySQL database.

Persistent volumes are important because removing or recreating a container should not automatically remove its stored data.

Volumes still require appropriate backup procedures. They do not protect against every type of data loss.

### Starting the containers

`cd ~/it-operations/glpi`

`sudo docker compose up -d`

### Checking container status

`sudo docker compose ps`

Both GLPI and MySQL were confirmed to be running.

### Application access

The GLPI container was exposed through the following host-side port mapping:

`127.0.0.1:8080:80`

This maps Ubuntu's loopback port 8080 to the container's HTTP port 80.

The database was accessed internally by GLPI through the Docker network rather than being exposed to Windows.

## 9. Secure Access to GLPI

GLPI was accessed from Windows using SSH local port forwarding.

A working example is:

`ssh -N -L 8081:127.0.0.1:8080 itops@192.168.212.130`

With the SSH tunnel active, GLPI can be opened in the Windows browser at:

`http://127.0.0.1:8081`

### How the connection works

1. Windows establishes an SSH connection to Ubuntu.
2. SSH listens on port 8081 on the Windows computer.
3. Connections to that port are forwarded to Ubuntu's loopback port 8080.
4. Docker forwards those requests to the GLPI container on port 80.
5. GLPI serves the web interface.

This arrangement avoids exposing the GLPI HTTP service directly to the wider network.

The tunnel encrypts communication between Windows and Ubuntu. It does not make the local HTTP service itself an HTTPS application.

The SSH tunnel must be active whenever GLPI is accessed using this method.

## 10. Nginx Test Service

Nginx was installed directly on the Ubuntu host as a separate service.

It was used to simulate service outages and test troubleshooting procedures without deliberately disrupting GLPI.

### Installation

`sudo apt install -y nginx`

### Service management

Check status:

`sudo systemctl status nginx`

Stop service:

`sudo systemctl stop nginx`

Start service:

`sudo systemctl start nginx`

Inspect logs:

`sudo journalctl -u nginx --no-pager -n 30`

Check HTTP availability:

`curl -I http://127.0.0.1`

### Verification

A successful HTTP response returned status code 200.

During controlled testing, Nginx was deliberately stopped and then restarted.

The service status and journal entries were inspected to confirm the shutdown and subsequent recovery.

The tests provided practical evidence for the incident management and monitoring components of the lab.

## 11. Infrastructure Troubleshooting

Several issues were encountered during configuration and testing.

### Issue 1 — GLPI unavailable after restarting the environment

**Symptoms:** The GLPI website could not be accessed from the Windows browser.

**Investigation:** The SSH tunnel, Ubuntu IP address and Docker container status were checked.

**Finding:** Some connection problems resulted from the SSH tunnel not being active.

**Resolution:** A new SSH port-forwarding connection was established from Windows.

### Issue 2 — SSH forwarding port already in use

**Symptoms:** SSH reported that the local address or port was already in use.

**Cause:** A previous tunnel or another process was already using the requested local port.

**Resolution:** An alternative local port was used while retaining the GLPI destination port of 8080.

### Issue 3 — GLPI connection reset

**Symptoms:** A local HTTP request to GLPI returned:

`curl: (56) Recv failure: Connection reset by peer`

**Investigation:** Docker container status and GLPI logs were inspected. The containers were running, but the web interface was not responding correctly.

**Resolution:** The GLPI application container was restarted using:

`sudo docker compose restart glpi`

GLPI access was restored without deleting the database or recreating the environment.

The precise underlying cause of the reset was not conclusively established.

### Lessons learned

These troubleshooting exercises demonstrated that a running container does not necessarily mean the application inside it is healthy.

They also reinforced the importance of checking individual components rather than assuming that every connection problem originates from the same service.

## 12. Verification and Testing

The infrastructure was validated using several checks.

| Test | Expected result | Outcome |
|---|---|---|
| Ubuntu startup | System reaches login prompt | Passed |
| SSH connectivity | Remote administration successful | Passed |
| Network connectivity | Outbound connection available | Passed |
| UFW firewall | Active with SSH permitted | Passed |
| Docker Engine | Service running | Passed |
| Docker Hello World | Container executes | Passed |
| Docker Compose | GLPI and MySQL running | Passed |
| GLPI web access | Login interface reachable | Passed |
| GLPI authentication | Administrator login successful | Passed |
| Nginx availability | HTTP 200 response | Passed |
| Nginx recovery | Service restored after controlled shutdown | Passed |

These checks established that the environment was suitable for the subsequent incident and change-management exercises.

## 13. Security Considerations and Limitations

The lab includes several basic security controls:

- UFW firewall enabled.
- Remote administration through SSH.
- GLPI HTTP access restricted to loopback and SSH forwarding.
- Default GLPI accounts secured or disabled where unused.
- Separate GLPI user accounts and role-based permissions.
- Database credentials stored outside published source files.

However, this remains a training environment.

Limitations include:

- Single virtual machine hosting the main services.
- No high-availability or clustered deployment.
- No externally verified backup and recovery strategy.
- Password-based SSH authentication.
- No direct HTTPS deployment for the GLPI application.
- Monitoring limited to selected local resources and services.
- Dependence on the Windows host and VMware VM remaining operational.

These limitations are acceptable for the scope of this project but would require additional consideration in a production environment.

## 14. Conclusion

The infrastructure phase established a functioning virtualised Linux environment for IT service management and service monitoring.

Ubuntu Server, Docker, GLPI and MySQL were deployed and verified successfully. SSH and UFW provided basic remote administration and network access controls, while Nginx supplied a separate service for controlled incident simulations.

The environment subsequently supported user management, incident escalation, change approval and automated monitoring exercises.

The main skills developed during this phase were Linux administration, virtualisation, container deployment, network troubleshooting, service management and technical problem-solving.

---

**Related documentation**

- `02-user-management.md` — GLPI users, groups and permissions
- `03-incident-management.md` — Incident investigation and escalation
- `04-change-management.md` — Change request and approval process
- `05-monitoring-and-alerting.md` — Python monitoring implementation
- `06-testing-and-results.md` — Testing and project evaluation
