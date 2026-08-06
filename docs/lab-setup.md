# Lab Setup

## Objective

The objective of this lab is to build a basic Security Operations Center (SOC) environment using Splunk Enterprise for centralized log collection, monitoring, and threat detection. The lab simulates how security analysts collect logs from endpoints and investigate security events.

---

## Lab Architecture

The lab consists of one Windows endpoint and one Splunk server running on the host machine.

```
+------------------+        TCP 9997        +----------------------+
|  Windows 10 VM   | ---------------------> |  Splunk Enterprise   |
|                  |                         |      (Host PC)       |
| Event Viewer     |                         | Search & Reporting   |
| UniversalForwarder|                        | Dashboards & Alerts  |
+------------------+                         +----------------------+
```

---

## Components Used

| Component | Purpose |
|----------|---------|
| Windows 10 VM | Generates Windows Event Logs |
| Splunk Universal Forwarder | Collects and forwards logs |
| Splunk Enterprise | Receives, indexes, and analyzes logs |
| VirtualBox | Hosts the Windows virtual machine |

---

## Screenshot 1 – Windows Virtual Machine

![Windows VM](../images/lab-setup/vm.png.jpg)

**Figure 1:** Windows 10 virtual machine used as the monitored endpoint.

---

## Screenshot 2 – Splunk Enterprise

![Splunk Enterprise](../images/lab-setup/splunk-home.png)

**Figure 2:** Splunk Enterprise dashboard running on the host machine.

---

## Screenshot 3 – Splunk Universal Forwarder

![Forwarder](../images/lab-setup/forwarder-installed.png)

**Figure 3:** Splunk Universal Forwarder installed on the Windows endpoint.

---

## Summary

After completing the setup, the Windows endpoint is ready to generate security events. The Universal Forwarder collects these events and forwards them to Splunk Enterprise, where they can be searched, visualized, and used to create security alerts.