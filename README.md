# Enterprise SIEM Deployment & Log Aggregation Lab

![SOC Architecture](Project%20Architecture.jpg)

## Executive Summary
This project demonstrates the end-to-end design, implementation, and operational validation of a centralized **Security Information and Event Management (SIEM)** pipeline using **Splunk Enterprise** on Ubuntu Server. The setup establishes an enterprise hub-and-spoke telemetry architecture that aggregates real-time event logs from heterogeneous endpoints (a Windows 11 workstation and an Ubuntu Server) via lightweight **Splunk Universal Forwarders**.

To maximize search performance and enforce logical data isolation, incoming telemetry streams are strictly partitioned into dedicated index pools, providing a foundational detection framework for SOC operations and security monitoring.

---

## Infrastructure Architecture & Node Mapping

The lab utilizes a hub-and-spoke model to decouple endpoint log generation from central indexing and analytical processing.

| Node Name | Operating System | Role in Architecture | IP Address | Telemetry Source / Details |
| :--- | :--- | :--- | :--- | :--- |
| **`Splunk-Indexer-SRV`** | Ubuntu Server LTS | **Central SIEM / Indexer Receiver** | `192.168.77.140` | Receiver bound to TCP port `9997`. Web UI accessible on port `8000`. |
| **`Win11-Endpoint`** | Windows 11 Pro | **Data Producer (Endpoint)** | `192.168.77.143` | Security, Application, and System Event Logs. |
| **`Linux-Endpoint-SRV`** | Ubuntu Server | **Data Producer (Endpoint)** | `192.168.77.141` | Syslog, `auth.log`, and Audit logs. |

---

## Technical Configuration Highlights

### 1. Windows Endpoint Configurations (`Win11-Endpoint`)
* **`inputs.conf`** — Located at `C:\Program Files\SplunkUniversalForwarder\etc\apps\Search\local\inputs.conf`:
  ```ini
  [WinEventLog://Application]
  index = windows_logs
  disabled = 0

  [WinEventLog://Security]
  index = windows_logs
  disabled = 0

  [WinEventLog://System]
  index = windows_logs
  disabled = 0
  ```

* **`outputs.conf`** — Located at `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf`:
  ```ini
  [tcpout]
  defaultGroup = default-autolb-group

  [tcpout:default-autolb-group]
  server = 192.168.77.140:9997

  [tcpout-server://192.168.77.140:9997]
  ```

---

### 2. Linux Endpoint Configurations (`Linux-Endpoint-SRV`)
* **`inputs.conf`** — Located at `/opt/splunkforwarder/etc/apps/search/local/inputs.conf`:
  ```ini
  [monitor:///var/log/syslog]
  disabled = false
  index = linux
  sourcetype = syslog

  [monitor:///var/log/auth.log]
  disabled = false
  index = linux
  sourcetype = syslog
  ```

* **`outputs.conf`** — Located at `/opt/splunkforwarder/etc/system/local/outputs.conf`:
  ```ini
  [tcpout]
  defaultGroup = primary_indexers

  [tcpout:primary_indexers]
  server = 192.168.77.140:9997
  ```

---

## Data Segregation & Index Storage Layer

To prevent index pollution and optimize Search Processing Language (SPL) query execution, log data is partitioned by operating system platform:
* **`windows_logs`**: Encapsulates structured Windows Event Log channels (Event IDs 4624, 4625, etc.) originating from `Win11-Endpoint`.
* **`linux`**: Captures unstructured system messages, shell events, and authentication logs (`/var/log/syslog`, `/var/log/auth.log`) from `Linux-Endpoint-SRV`.

---

## Operational Pipeline Validation

To confirm log fidelity and pipeline stability across endpoints, targeted SPL queries were executed in Splunk Web:

### Linux Log Stream Verification
```spl
index="linux" | stats count by host, sourcetype
```
* **Result:** Confirmed active event streams from `Linux-Endpoint-SRV`, tracking local authentication and system events.

### Windows Event Stream Verification
```spl
index="windows_logs" | top limit=10 EventCode
```
* **Result:** Parsed structured security events, registering critical Windows Event Codes such as **4624** *(Successful Account Logon)* and **4625** *(Failed Account Logon)*.

---
