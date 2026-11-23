# 05 — Automated Monitoring and Alerting

## 1. Overview

This document covers the design, development and testing of the Python-based IT Operations Health Monitor.

The monitoring solution was developed following a simulated Nginx service outage and the approval of GLPI Change #1.

Its purpose was to automate service availability checks, collect system resource information and record failures and recoveries.

The system runs on Ubuntu Server and uses Python, systemd and structured JSONL logging.

The project demonstrates fundamental IT Operations monitoring concepts without relying on a full enterprise monitoring platform.

## 2. Objectives

The objectives were to:

- Monitor the availability of the Nginx HTTP service.
- Measure HTTP response times.
- Collect CPU, memory and disk utilisation.
- Identify resource usage exceeding warning thresholds.
- Record monitoring results with timestamps.
- Detect service outages and recoveries.
- Prevent repeated alerts for an unchanged condition.
- Automate monitoring using systemd.
- Record significant alerts in the Linux system journal.
- Test the system under healthy and failed service conditions.

## 3. Development Environment

| Component | Configuration |
|---|---|
| Operating system | Ubuntu Server 26.04.1 LTS |
| Virtualisation | VMware Workstation |
| Language | Python 3 |
| Monitored web service | Nginx |
| HTTP endpoint | `http://127.0.0.1/` |
| Monitoring interval | Approximately one minute |
| Scheduler | systemd timer |
| Monitoring log format | JSON Lines |
| Local notification mechanism | Linux system journal |
| Project directory | `/home/itops/it-operations/health-checker/` |

The monitoring system was developed separately from GLPI.

The GLPI platform was used to manage the related incident and change records, while the Python application performed the technical monitoring.

## 4. Application Structure

The project consists of five Python modules and two systemd configuration files.

```text
health-checker/
├── health_checker.py
├── system_metrics.py
├── monitor.py
├── alerting.py
├── notifications.py
└── logs/
    ├── checks.jsonl
    ├── system_metrics.jsonl
    ├── monitor.jsonl
    ├── alerts.jsonl
    └── alert_state.json

systemd/
├── itops-monitor.service
└── itops-monitor.timer
```

The systemd files are stored in `/etc/systemd/system/` on Ubuntu. The structure above represents how they can be organised for GitHub.

The log files contain runtime-generated data and should not be treated as application source code.

## 5. HTTP Availability Monitoring

**Module:** `health_checker.py`

This module checks whether the local Nginx web service responds successfully.

### 5.1 How It Works

The script:

1. Sends an HTTP request to the configured endpoint.
2. Measures the time taken to receive the response.
3. Reads the HTTP status code.
4. Determines whether the service is healthy.
5. Handles network and connection errors.
6. Produces a structured result.
7. Writes the result to a JSONL log when executed independently.

The initial implementation treats HTTP status code 200 as a successful check.

### 5.2 Service Configuration

The monitored service was configured as:

`SERVICE_NAME = "nginx"`

`SERVICE_URL = "http://127.0.0.1/"`

`TIMEOUT = 5`

The timeout prevents a failed connection from causing the script to wait indefinitely.

### 5.3 Healthy Response

When Nginx is operational, the script returns information similar to:

```text
[UP] nginx | HTTP: 200 | Response: 14.89 ms
```

The actual response time depends on the server's current activity.

### 5.4 Failed Response

When Nginx is unavailable, the script handles the connection error and reports:

```text
[DOWN] nginx | HTTP: None | Response: 12.31 ms
Error: Connection refused
```

The measured response time in this case represents how quickly the request failed, not successful application latency.

### 5.5 Exit Codes

The standalone health checker uses:

| Exit code | Meaning |
|---|---|
| 0 | Healthy response |
| 1 | Failed health check |

These exit codes allow other processes to identify failed checks.

## 6. System Resource Monitoring

**Module:** `system_metrics.py`

The system resource module collects CPU, memory and disk usage information directly from Linux.

### 6.1 CPU Monitoring

CPU utilisation is calculated using the cumulative counters available in:

`/proc/stat`

The script samples the counters twice, approximately one second apart.

It calculates the proportion of time spent performing non-idle activity during that interval.

This provides a short measurement of CPU utilisation.

### 6.2 Memory Monitoring

Memory information is collected from:

`/proc/meminfo`

The script uses the `MemTotal` and `MemAvailable` values to estimate memory utilisation.

The calculation is:

`Memory usage % = (MemTotal - MemAvailable) / MemTotal × 100`

Using available memory is preferable to treating all cache usage as permanently consumed RAM.

### 6.3 Disk Monitoring

Disk usage is collected using Python's `shutil.disk_usage()` function.

The initial implementation monitors the root filesystem:

`/`

The calculation is:

`Disk usage % = Used space / Total space × 100`

This provides visibility into available storage on the Ubuntu server.

## 7. Warning Thresholds

The initial monitoring thresholds were:

| Resource | Warning threshold |
|---|---|
| CPU | 90% |
| Memory | 85% |
| Disk | 85% |

These thresholds were selected for the lab and are not intended as universal production recommendations.

When a measurement reaches or exceeds its threshold, the script records a warning.

Examples include:

- High CPU usage.
- High memory usage.
- Low disk space.

The warning functionality was implemented, although the screenshots primarily demonstrate healthy resource readings rather than a verified threshold-crossing exercise.

## 8. Combined Monitoring Application

**Module:** `monitor.py`

The main monitoring program combines the HTTP and system resource checks.

Rather than requiring the two monitoring modules to be run separately, the combined program executes the checks and produces a single monitoring result.

### 8.1 Processing Sequence

1. Check Nginx HTTP availability.
2. Measure CPU utilisation.
3. Measure memory utilisation.
4. Measure disk utilisation.
5. Compare readings with warning thresholds.
6. Determine the overall monitoring state.
7. Display the results.
8. Record the combined result.
9. Process new failure and recovery alerts.

### 8.2 Overall Status

The application uses three overall states:

| Status | Meaning |
|---|---|
| OK | HTTP service healthy and resources below thresholds |
| WARNING | One or more resource thresholds exceeded |
| CRITICAL | HTTP service unavailable |

A failed HTTP check takes priority over resource warnings.

This means the application can identify a service outage even when CPU, memory and disk usage remain normal.

### 8.3 Example Output

```text
=== IT Operations Health Monitor ===
Overall status: OK

--- Service Health ---
Service: nginx
Status: UP
HTTP: 200
Response: 13.53 ms

--- System Resources ---
CPU: 1.51%
Memory: 41.5%
Disk: 54.25%
```

This output was recorded during testing of the combined monitoring application.

## 9. Structured Monitoring Logs

The application records monitoring results using JSON Lines, commonly abbreviated to JSONL.

Each line contains a separate JSON object.

### 9.1 Why JSONL?

JSONL was selected because it:

- Supports structured monitoring records.
- Allows new records to be appended.
- Is readable by Python and other tools.
- Can be processed line by line.
- Supports future reporting or analysis.

### 9.2 Combined Log Structure

The combined monitor records:

- UTC timestamp.
- Overall monitoring status.
- Service availability result.
- HTTP response code.
- HTTP response time.
- CPU percentage.
- Memory percentage.
- Disk percentage.
- Active resource warnings.

The combined results are written to:

`logs/monitor.jsonl`

### 9.3 Viewing Logs

The most recent result can be displayed using:

`tail -n 1 logs/monitor.jsonl | python3 -m json.tool`

Multiple records can be viewed using:

`tail -n 3 logs/monitor.jsonl`

### 9.4 JSONL Troubleshooting

During testing, an error occurred when multiple JSONL records were passed directly to Python's JSON formatter.

The command attempted to parse several standalone objects as one JSON document.

This produced an `Extra data` error.

The issue was resolved by formatting one line at a time.

The underlying monitoring data was not corrupted.

## 10. Automated Monitoring with systemd

Manual execution was useful for early testing, but it did not provide continuous scheduled monitoring.

To address this, a systemd service and timer were configured.

### 10.1 Monitoring Service

**File:** `itops-monitor.service`

```ini
[Unit]
Description=IT Operations Health Monitor
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
User=itops
WorkingDirectory=/home/itops/it-operations/health-checker
ExecStart=/usr/bin/python3 /home/itops/it-operations/health-checker/monitor.py
```

The service uses `Type=oneshot` because each monitoring execution performs its checks and then exits.

It does not need to remain running continuously.

The script runs under the dedicated Ubuntu user rather than requiring root privileges for routine measurements.

### 10.2 Monitoring Timer

**File:** `itops-monitor.timer`

```ini
[Unit]
Description=Run IT Operations Health Monitor every minute

[Timer]
OnCalendar=*-*-* *:*:00
AccuracySec=5s
Persistent=true
Unit=itops-monitor.service

[Install]
WantedBy=timers.target
```

The timer schedules the monitoring service approximately once every minute.

### 10.3 Enabling Monitoring

The systemd configuration was loaded using:

`sudo systemctl daemon-reload`

Automatic monitoring was enabled using:

`sudo systemctl enable --now itops-monitor.timer`

### 10.4 Verification

Timer status was inspected using:

`sudo systemctl status itops-monitor.timer`

Scheduled executions were checked using:

`systemctl list-timers --all | grep itops-monitor`

The service journal was inspected using:

`sudo journalctl -u itops-monitor.service -n 40 --no-pager`

Testing confirmed that repeated monitoring executions occurred automatically.

### 10.5 Important Limitation

The VM must remain powered on for monitoring to operate.

The systemd timer does not provide monitoring while Ubuntu itself is unavailable.

The configured persistent timer can trigger a catch-up execution after a period of inactivity, but it does not reconstruct every missed monitoring result.

## 11. Alert Detection

**Module:** `alerting.py`

The alerting module was introduced to identify new problems and service recoveries.

The initial monitoring program reported the current state during every execution.

However, simply recording CRITICAL every minute would not distinguish a newly detected outage from one that had already been acknowledged.

### 11.1 State Tracking

The alerting module stores active problems in:

`logs/alert_state.json`

During each execution, the current problems are compared against the previously recorded problems.

### 11.2 New Problem Detection

If a problem appears that was not present during the previous check, a new alert event is recorded.

For example:

`[TRIGGERED] nginx is unavailable`

### 11.3 Recovery Detection

If a previously recorded problem is no longer present, a recovery event is generated.

For example:

`[RECOVERED] Resolved: nginx is unavailable`

### 11.4 Duplicate Suppression

If Nginx remains unavailable over several consecutive checks, the monitoring system does not create a new TRIGGERED event for every execution.

It retains the active problem state until the condition changes.

### 11.5 Alert Records

Events are written to:

`logs/alerts.jsonl`

Each event includes:

- Timestamp.
- Event type.
- Severity.
- Message.

### 11.6 State Persistence

The alert state is stored between script executions.

This is important because the systemd timer starts a new Python process for each scheduled check.

Without persistent state, each execution would not know whether a failure had already been recorded.

## 12. Local Notifications

**Module:** `notifications.py`

The notification module was introduced to write significant monitoring events to the Linux system journal.

It uses Linux's `logger` command.

### 12.1 Notification Types

The initial implementation handles:

- TRIGGERED events.
- RECOVERED events.

A newly detected service outage is recorded with a higher logging priority than a normal recovery notification.

### 12.2 Viewing Notifications

Local notifications can be inspected using:

`journalctl -t itops-alert -n 10 --no-pager`

During testing, the journal contained:

```text
[TRIGGERED] nginx is unavailable
[RECOVERED] Resolved: nginx is unavailable
```

### 12.3 Notification Limitations

These notifications are local journal entries.

They do not automatically send email, mobile push notifications or messages to an external chat platform.

Discord webhook integration was discussed as a possible extension but was not completed as part of the tested implementation.

## 13. Outage Detection Testing

A controlled service outage was used to verify the monitoring functionality.

### 13.1 Initial Healthy State

The monitor was executed with Nginx running.

It reported:

`Status: UP`

`HTTP: 200`

The overall monitoring state was OK.

### 13.2 Simulated Service Outage

Nginx was stopped using:

`sudo systemctl stop nginx`

The monitor was then executed:

`python3 monitor.py`

It reported:

`Overall status: CRITICAL`

`Status: DOWN`

The connection attempt produced a connection-refused error.

The alerting module recorded a TRIGGERED event.

### 13.3 Recovery Test

Nginx was restarted using:

`sudo systemctl start nginx`

The monitoring program was executed again.

The service returned HTTP 200, and the overall state returned to OK.

A RECOVERED event was recorded.

### 13.4 Result

The exercise confirmed that the monitor could identify both an unavailable service and successful recovery.

## 14. Duplicate Alert Prevention Testing

A further controlled test was performed to check that repeated monitoring executions would not generate duplicate failure alerts.

The test sequence was:

1. Stop Nginx.
2. Run the monitor.
3. Run the monitor again while Nginx remains stopped.
4. Run the monitor a third time.
5. Restart Nginx.
6. Run the monitor again.

### Expected Behaviour

- One TRIGGERED alert for the new outage.
- No additional TRIGGERED alerts while the condition remains unchanged.
- One RECOVERED alert after the service is restored.

### Observed Result
