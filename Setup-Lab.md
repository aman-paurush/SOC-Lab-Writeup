# Lab Setup

## Objective

The objective of this lab is to build a basic Security Operations Center (SOC) environment using Splunk Enterprise for centralized log collection, monitoring, and threat detection. The lab simulates how security analysts collect logs from endpoints and investigate security events.

---

## Lab Architecture

The lab consists of one Windows endpoint and one Splunk server running on the host machine.

```
+-------------------+        TCP 9997        +----------------------+
|  Windows 10 VM    | ---------------------> |  Splunk Enterprise   |
|                    |                        |      (Host PC)       |
| Event Viewer       |                        | Search & Reporting   |
| Universal Forwarder|                        | Dashboards & Alerts  |
+--------------------+                        +----------------------+
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

## Step 1: Create a Splunk Account

1. Go to [splunk.com](https://www.splunk.com/) and click **Free Trial** (or **Sign Up**).
2. Choose **Splunk Enterprise Free Trial** — this gives full-featured access for 60 days, no credit card required.
3. Fill in your email and create an account. Verify your email if prompted.
4. Once logged in, you'll land on the **Splunk Downloads** page — keep this open for Step 2.

![Splunk Signup](signup.png)

---

## Step 2: Install Splunk Enterprise (on Host Machine)

1. From the Downloads page, select your OS (Windows/Linux/macOS) and download the installer.
![Splunk Download](download.png)

2. Run the installer:
   - Accept the license agreement.
   - Create an **admin username and password** — you'll use this to log into the Splunk web interface. Save it somewhere safe.
   - Keep the default installation path and management port (**8089**) unless you have a specific reason to change them.
3. Once installation completes, Splunk starts automatically. Open a browser and go to:
   ```
   http://localhost:8000
   ```
4. Log in with the admin credentials created above.

**✅ Checkpoint:** You should now see the Splunk Enterprise home screen (**Figure 2**).

---

## Step 3: Set Up the Windows 10 Endpoint (VM)

1. Install **VirtualBox** (or your preferred hypervisor) on the host machine.
2. Create a new VM and install **Windows 10** using an ISO image.
3. Allocate at least 2 vCPUs and 4GB RAM for smooth performance.
4. Set the VM's network adapter to **Bridged Adapter** (or **Host-Only + NAT**) so it can reach the host machine's IP over the network — this is required for the forwarder to send logs later.
5. Once Windows 10 boots, note down its IP address (`ipconfig` in Command Prompt) — you'll need this to confirm connectivity.

**✅ Checkpoint:** Windows 10 VM is running and reachable from the host (**Figure 1**).

![Window 10 VM](win10.png)
---

## Step 4: Enable a Receiving Port on Splunk (Host Machine)

Before installing the forwarder, tell Splunk Enterprise to listen for incoming forwarded data.

1. In the Splunk web interface, go to **Settings → Forwarding and receiving**.
2. Under **Receive data**, click **Add new**.
3. Set the listening port to **9997** (default) and click **Save**.
4. Note the **host machine's IP address** (`ipconfig`/`ifconfig`) — the forwarder on the Windows VM will need this to know where to send logs.

---

## Step 5: Install Splunk Universal Forwarder (on Windows 10 VM)

1. On the **host machine's** Splunk Downloads page, select **Universal Forwarder**, choose Windows, and download the installer.
2. Transfer the installer to the Windows 10 VM (via shared folder, USB, or direct download inside the VM).
3. Run the installer on the VM:
   - Accept the license agreement.
   - When prompted for **deployment server**, you can skip this for a simple lab setup.
   - When prompted for the **receiving indexer**, enter the host machine's IP address and port `9997`.
   - Create a forwarder admin username/password (can differ from the Splunk Enterprise credentials).
4. Complete the installation. The Universal Forwarder runs as a Windows service in the background.

**✅ Checkpoint:** Universal Forwarder is installed and running on the Windows endpoint (**Figure 3**).
![Splunk Forwarder](image.png)

---

## Step 6: Configure Log Forwarding (Windows Event Logs)

1. On the Windows VM, navigate to the Universal Forwarder install directory, typically:
   ```
   C:\Program Files\SplunkUniversalForwarder\etc\system\local\
   ```
2. Create or edit `inputs.conf` to forward Windows Event Logs:
   ```ini
   [WinEventLog://Security]
   disabled = false
   index = main

   [WinEventLog://System]
   disabled = false
   index = main

   [WinEventLog://Application]
   disabled = false
   index = main
   ```
3. Restart the Splunk Forwarder service for changes to take effect:
   ```
   net stop SplunkForwarder
   net start SplunkForwarder
   ```

---

## Step 7: Verify Data Is Flowing into Splunk

1. On the host machine, open the Splunk web interface and go to **Search & Reporting**.
2. Run a basic search to confirm logs are arriving:
   ```spl
   index=main | stats count by sourcetype
   ```
3. You should see event counts for `WinEventLog:Security`, `WinEventLog:System`, and `WinEventLog:Application`.
4. If no data appears, check:
   - Windows Firewall isn't blocking port 9997 on either machine.
   - The forwarder service is running (`services.msc` on the VM).
   - The receiving port is correctly configured on the host (Step 4).

**✅ Checkpoint:** Search results return live events from the Windows endpoint — the pipeline is working end-to-end.

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

Starting from a fresh Splunk account, this setup walks through installing Splunk Enterprise on the host machine, provisioning a Windows 10 endpoint in VirtualBox, and deploying the Splunk Universal Forwarder to collect Windows Event Logs. By the end of this stage, the Windows endpoint is generating security events, the Universal Forwarder is collecting them, and Splunk Enterprise is indexing them — laying the foundation for the detection, dashboarding, and alerting work covered in the next sections of this lab.