# Port Configuration Troubleshooting

This document provides a centralized guide for troubleshooting port-related issues for all components of the Elastic Stack and Environment Watch in a Relativity environment.

> [!NOTE]
> Click here to view the latest port diagram that includes the Elastic Stack [Port Diagram](../environment-watch/port-diagram.md)

## Default Port Reference

The following table summarizes the default ports used by the Elastic Stack and Environment Watch components.

| Component | Port | Protocol | Purpose |
| :--- | :--- | :--- | :--- |
| Elasticsearch | 9200 | HTTP | Client communication, REST API |
| Elasticsearch | 9300 | TCP | Inter-node communication (Transport) |
| Kibana | 5601 | HTTP | Kibana web interface |
| APM Server | 8200 | HTTP | APM agent data ingestion |
| OpenTelemetry Collector | 4318 | HTTP | OTLP data reception |

## Table of Contents

  [1. Elasticsearch Port Issues](#1-elasticsearch-port-issues) <br>
  [2. Kibana Port Issues](#2-kibana-port-issues) <br>
  [3. APM Server Port Issues](#3-apm-server-port-issues)<br>
  [4. OpenTelemetry Collector Port Issues](#4-opentelemetry-collector-port-issues) <br>
  [5. General Port Troubleshooting](#5-general-port-troubleshooting)<br>

---

## 1. Elasticsearch Port Issues

**Symptoms:**
- Elasticsearch fails to bind to default ports.
- "Address already in use" errors in logs.
- Cannot access Elasticsearch via HTTP/HTTPS.

**Troubleshooting Steps:**

* **Check if Ports are in Use:**
  Verify that ports 9200 and 9300 are listening.
  ```powershell
  netstat -an | findstr ":9200"
  netstat -an | findstr ":9300"
  ```
  <details>
  <summary>Expected output</summary>

  ```
  TCP    0.0.0.0:9200           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:9300           0.0.0.0:0              LISTENING
  ```
  </details>

* **Identify Conflicting Processes:**
  If a port is in use by another application, identify the process.
  ```powershell
  Get-NetTCPConnection -LocalPort 9200 -State Listen
  Get-NetTCPConnection -LocalPort 9300 -State Listen
  ```

* **Test Elasticsearch Connectivity:**
  ```powershell
  curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
  ```

* **Verify Network Binding:**
  Check `C:\elastic\elasticsearch\config\elasticsearch.yml` configuration:
  ```yaml
  network.host: 0.0.0.0  # For all interfaces
  ```

---

## 2. Kibana Port Issues

**Symptoms:**
- Kibana fails to bind to the default port.
- "EADDRINUSE" errors in logs.
- Cannot access Kibana web interface.

**Troubleshooting Steps:**

* **Check if Port is in Use:**
  ```powershell
  netstat -an | findstr ":5601"
  ```
  <details>
  <summary>Expected output</summary>

  ```
  TCP    0.0.0.0:5601           0.0.0.0:0              LISTENING
  ```
  </details>

* **Test Kibana Connectivity:**
  ```powershell
  (curl.exe -s -k -u <username>:<password> -X GET "http://<hostname_or_ip>:5601/api/status" | ConvertFrom-Json).status.overall | ConvertTo-Json -Depth 10
  ```

* **Verify Network Binding:**
  Check `C:\elastic\kibana\config\kibana.yml` configuration:
  ```yaml
  server.host: "0.0.0.0"  # For all interfaces
  ```

---

## 3. APM Server Port Issues

**Symptoms:**
- APM Server fails to bind to the default port.
- "Address already in use" errors in logs.
- APM agents cannot connect to the server.

**Troubleshooting Steps:**

* **Check if Port is in Use:**
  ```powershell
  netstat -an | findstr ":8200"
  ```
  <details>
  <summary>Expected output</summary>

  ```
  TCP    0.0.0.0:8200           0.0.0.0:0              LISTENING
  ```
  </details>

* **Test APM Server Connectivity:**
  ```powershell
  curl.exe -k "http://<hostname_or_ip>:8200/"
  ```
  <details>
  <summary>Expected output</summary>

  ```json
  {
    "build_date": "...",
    "build_sha": "...",
    "publish_ready": true,
    "version": "8.17.3"
  }
  ```
  </details>

* **Verify Network Binding:**
  Check `C:\elastic\apm-server\apm-server.yml` configuration:
  ```yaml
  host: "0.0.0.0:8200"
  ```

---

## 4. OpenTelemetry Collector Port Issues

**Symptoms:**
- The `otelcol-relativity.exe` process is running, but no data is being sent.
- Port 4318 is not in a listening state.

**Troubleshooting Steps:**

* **Check if Port is in Use:**
  This port is used by the OpenTelemetry Collector to receive data. The `Relativity Environment Watch` service must be running.
  ```powershell
  netstat -an | findstr ":4318"
  ```
  <details>
  <summary>Expected output</summary>

  ```
  TCP    0.0.0.0:4318           0.0.0.0:0              LISTENING
  ```
  </details>

  You can also use `Get-NetTCPConnection`:
  ```powershell
  Get-NetTCPConnection -LocalPort 4318 -State Listen
  ```

---

## 5. General Port Troubleshooting

* **Firewall Rules:**
  Ensure that Windows Firewall or any other network security software is not blocking the required ports. You may need to create inbound rules to allow traffic on these ports.

  **Example for Kibana (port 5601):**
  ```powershell
  New-NetFirewallRule -DisplayName "Kibana Web Interface" -Direction Inbound -Protocol TCP -LocalPort 5601 -Action Allow
  ```

* **Network Connectivity:**
  Use `Test-NetConnection` to verify that a remote server can reach the port.
  ```powershell
  Test-NetConnection -ComputerName <hostname_or_ip> -Port <port_number>
  ```
  <details>
  <summary>Expected output</summary>

  ```
  ComputerName     : <hostname_or_ip>
  RemoteAddress    : <ip>
  RemotePort       : <port_number>
  TcpTestSucceeded : True
  ```
  </details>
