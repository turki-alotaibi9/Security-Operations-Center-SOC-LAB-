# Enterprise SIEM Deployment & Log Aggregation Lab

![SOC Architecture](Project%20Architecture.jpg)

## Executive Summary
This project demonstrates the end-to-end design, implementation, and operational validation of a centralized **Security Information and Event Management (SIEM)** pipeline using **Splunk Enterprise** on Ubuntu Server[cite: 5]. The setup establishes an enterprise hub-and-spoke telemetry architecture that aggregates real-time event logs from heterogeneous endpoints (Windows 11 workstation and an Ubuntu Server) via lightweight **Splunk Universal Forwarders**[cite: 4, 5, 6].

To maximize search performance and enforce logical data isolation, incoming telemetry streams are strictly partitioned into dedicated index pools, providing a foundational detection framework for SOC operations and security monitoring[cite: 5, 7, 9].

---

## Infrastructure Architecture & Node Mapping

The lab utilizes a hub-and-spoke model to decouple endpoint log generation from central indexing and analytical processing[cite: 6].

| Node Name | Operating System | Role in Architecture | IP Address | Telemetry Source / Details |
| :--- | :--- | :--- | :--- | :--- |
| **`Splunk-Indexer-SRV`** | Ubuntu Server LTS | **Central SIEM / Indexer Receiver** | `192.168.77.140` | Receiver bound to TCP port `9997`[cite: 4, 6]. Web UI accessible on port `8000`[cite: 4]. |
| **`Win11-Endpoint`** | Windows 11 Pro | **Data Producer (Endpoint)** | `192.168.77.143` | Security, Application, and System Event Logs[cite: 4, 6]. |
| **`Linux-Endpoint-SRV`** | Ubuntu Server | **Data Producer (Endpoint)** | `192.168.77.141` | Syslog, `auth.log`, and Audit logs[cite: 4, 6]. |

---

## Technical Configuration Highlights

### 1. Windows Endpoint Configurations (`Win11-Endpoint`)
* **`inputs.conf`** — Captures Windows Event Log channels and routes them to the designated index[cite: 8, 11]:
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
