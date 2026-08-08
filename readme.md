# SOC Lab Writeup

A hands-on Security Operations Center (SOC) lab documenting the process of collecting, monitoring, and analyzing security logs from multiple sources using Splunk Enterprise.

The objective of this project is to understand how logs are generated, collected, and analyzed to detect suspicious activities and security incidents. The lab covers Windows Event Logs, IIS Web Server Logs, Snort IDS Logs, and their integration with Splunk for centralized monitoring and threat detection.

---

## Lab Environment

| Component | Description |
|-----------|-------------|
| Host Machine | Windows 11 |
| Web Server | Windows 10 |
| Attacker Machine | Ubuntu Linux |
| SIEM | Splunk Enterprise |
| Log Collector | Splunk Universal Forwarder |
| IDS | Snort IDS |
| Web Server | IIS |

---

## Architecture

```
                  +-----------------------+
                  |    Ubuntu Attacker    |
                  |  Nmap • Hydra • SQLi  |
                  +----------+------------+
                             |
                             |
                             ▼
               +-----------------------------+
               |     Windows 10 Web Server   |
               |-----------------------------|
               | Windows Event Logs          |
               | IIS Web Server Logs         |
               | Snort IDS Logs              |
               +--------------+--------------+
                              |
                 Splunk Universal Forwarder
                              |
                              ▼
                +---------------------------+
                |     Splunk Enterprise     |
                |---------------------------|
                | Search & Reporting        |
                | Dashboards                |
                | Detection Rules           |
                | Alerts                    |
                +---------------------------+
```

The logging documentation set now includes local logging, IIS logs, Snort IDS logs, and Splunk logging guides for a more complete lab walkthrough.

---

## Project Structure

```
SOC-Lab-Writeup/
│
├── docs/
│   ├── Incident Detection and Triage/
│   └── Logging/
│       ├── IIS-logs.md
│       ├── local-logging.md
│       ├── snort-IDS-logs.md
│       └── Splunk-Logging.md
│
├── images/
│   ├── Incident Detection and Triage/
│   ├── lab-setup/
│   ├── logging/
│   │   ├── IIS-logs/
│   │   ├── local-logging/
│   │   ├── snort/
│   │   └── various-sources/
│   └──
├── readme.md
└── Setup-Lab.md
```

---

## Topics Covered

- Lab Setup
- Windows Event Logging
- Windows Audit Policy
- IIS Logging
- Snort IDS Logging
- Log Generation
- Log Analysis
- Network Scan Detection
- SQL Injection Detection
- Splunk Integration
- Dashboard Creation
- Alert Generation

---

## Tools Used

- Splunk Enterprise
- Splunk Universal Forwarder
- Snort IDS
- IIS Web Server
- Windows Event Viewer
- Nmap
- Hydra
- Notepad++
- Ubuntu Linux
- Windows 10
- VirtualBox

---

## Learning Outcomes

After completing this lab, I was able to:

- Configure Windows Audit Policies.
- Analyze Windows Security Event Logs.
- Monitor IIS Web Server Logs.
- Configure and use Snort IDS.
- Detect network scanning activities using Snort.
- Generate and analyze web attack logs.
- Forward logs to Splunk Enterprise.
- Search and investigate security events.
- Create dashboards and alerts for security monitoring.

---

## Author

**Aman Paurush**
