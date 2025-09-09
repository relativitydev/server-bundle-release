# APM Server Troubleshooting

This document provides troubleshooting guidance for common APM Server issues encountered during installation, configuration, and operation in Relativity Server environments.

> [!NOTE]
> This guide assumes a default APM Server installation path of `C:\elastic\apm-server`. Adjust paths according to your actual installation directory.

## Table of Contents

  - [1. Windows Service Issues](#1-windows-service-issues)
    - [1.1 APM Server Service Not Starting](#11-apm-server-service-not-starting)
    - [1.2 Service Crashes or Stops Unexpectedly](#12-service-crashes-or-stops-unexpectedly)
    - [1.3 Permission and Access Issues](#13-permission-and-access-issues)
  - [2. Memory Issues](#2-memory-issues)
  - [3. Self-Instrumentation](#3-self-instrumentation)
  - [4. Service Verification](#4-service-verification)

---

## 1. Windows Service Issues

### 1.1 APM Server Service Not Starting

**Symptoms:**
- APM Server service fails to start
- Service stops immediately after starting
- Error messages in APM Server logs

**Troubleshooting Steps:**

**Check APM Server Status:**
   ```powershell
   Get-Service -Name apm-server
   ```
   <details>
   <summary>Expected response</summary>

   ```
   Status   Name        DisplayName
   ------   ----        -----------
   Running  apm-server  Elastic APM Server
   ```
   </details>

**Verify Service Configuration:**
   ```powershell
   (Get-CimInstance Win32_Service -Filter "Name = 'apm-server'").StartName
   ```
   <details>
   <summary>Expected response</summary>

   ```
   LocalSystem
   ```
   </details>

**Check APM Server Logs:**
   1. Navigate to `C:\Program Files\apm-server\logs\`
   2. Review the latest log files (`apm-server.log`) for error messages
   3. Look for configuration errors or connection issues with Elasticsearch

> [!NOTE]
> For Elasticsearch connection issues, see [Elasticsearch Troubleshooting](elasticsearch.md)

**Verify Configuration File:**
   ```powershell
   C:\elastic\apm-server\apm-server.exe test config -c "C:\elastic\apm-server\apm-server.yml"
   ```
   <details>
   <summary>Expected response</summary>

   ```
   Config OK
   ```
   </details>

---

### 1.2 Service Crashes or Stops Unexpectedly

**Symptoms:**
- APM Server service starts but stops after a short period
- Service status shows "Stopped" unexpectedly
- APM data collection stops working

**Troubleshooting Steps:**

* **Check APM Server Logs:**  
  See above.

* **Review APM Server Configuration:**
  - Check `apm-server.yml` file in `C:\elastic\apm-server\`
  - Verify Elasticsearch connection settings (see [Elasticsearch Troubleshooting](elasticsearch.md) for detailed troubleshooting)
  - Common configuration issues:
    - **TLS**: Ensure correct protocol (`http` vs `https`)
    - **Hostname**: Verify correct Elasticsearch server hostname
    - **Port**: Confirm correct Elasticsearch port (usually 9200)

> [!NOTE]
> API keys are the preferred authentication method and expire by default in 6 months. Consider switching from username/password to API key authentication. For API key creation, see [Kibana Troubleshooting](kibana.md).

   ```yaml
   output.elasticsearch:
     hosts: ["https://<hostname_or_ip>:9200"]
     api_key: "your-api-key-here"
     # OR (not recommended)
     # username: "<username>"
     # password: "<password>"
   ```
> [!NOTE]
> This section in `apm-server.yml` configures how APM Server connects to your Elasticsearch cluster.  
> - `hosts`: The URL(s) of your Elasticsearch node(s).
> - `api_key`: The recommended authentication method.  
> - `username`/`password`: Legacy authentication (not recommended; use API keys instead).
> For instructions on creating an API key, see [Kibana Troubleshooting](kibana.md).

* **To verify the connection, run:**
   ```powershell
   C:\elastic\apm-server\apm-server.exe test output -c "C:\elastic\apm-server\apm-server.yml"
   ```
   <details>
   <summary>Expected output for successful connection</summary>

   ```
   elasticsearch: https://<hostname_or_ip>:9200...
     parse url... OK
     connection...
       parse host... OK
       dns lookup... OK
       addresses: fe80::61a7:3f3f:210:8d65%Ethernet 2, 10.0.2.2
       dial up... OK
     TLS...
       security... WARN server's certificate chain verification is disabled
       handshake... OK
       TLS version: TLSv1.3
       dial up... OK
     talk to server... OK
     version: 8.17.3
   ```
   </details>

> [!NOTE]
> To verify Elasticsearch connectivity, see [Elasticsearch Troubleshooting](elasticsearch.md).

---

### 1.3 Permission and Access Issues

**Symptoms:**
- Access denied errors when starting service
- Unable to write to log directories
- Configuration file access errors

**Troubleshooting Steps:**

* The APM Server Windows service runs under Local System account by default.
* Verify access to `C:\elastic\apm-server\` directory.
* Check write permissions to `C:\Program Files\apm-server\logs\` directory.

---

## 2. Memory Issues

### 2.1 High Memory Usage

**Symptoms:**
- APM Server consuming excessive memory
- System performance degradation
- Out of memory errors in logs

**Troubleshooting Steps:**

* **Check APM Server Logs:**
  - Look for memory-related error messages or stack traces.
  - Example error message:
    ```
    fatal error: out of memory
    ```

* **Review APM Server Configuration:**
  - Check `apm-server.yml` for memory-related settings.
  - Common settings to review:
    - `apm-server.memory.limit`: Maximum memory APM Server can use.
    - `apm-server.memory.queue`: Size of the memory queue for incoming events.

  - Example configuration:
    ```yaml
    apm-server:
      memory:
        limit: 512mb
        queue: 1000
    ```

* **Monitor System Resources:**
  - Use Task Manager or Resource Monitor to check APM Server memory usage.
  - Compare with configured limits in `apm-server.yml`.

* **Adjust Memory Settings:**
  - If APM Server is using too much memory, consider adjusting the following settings in `apm-server.yml`:
    - `apm-server.memory.limit`: Decrease the maximum memory limit.
    - `apm-server.memory.queue`: Decrease the memory queue size.

  - Example:
    ```yaml
    apm-server:
      memory:
        limit: 256mb
        queue: 500
    ```

* **Restart APM Server:**
  - After making changes, restart the APM Server service:
    ```powershell
    Restart-Service apm-server
    ```

    <details>
   <summary>Expected response</summary>

   ```
   WARNING: Waiting for service 'apm-server
   (apm-server)' to stop...
   ```
   </details>

---

## 3. Self-Instrumentation

**Symptoms:**
- Need to monitor APM Server itself for performance and health metrics
- Want to enable self-monitoring and observability for the APM Server

**Troubleshooting Steps:**

* **Enable Self-Instrumentation:**
  - Self-instrumentation allows APM Server to monitor its own performance.
  - This feature is configured in the `apm-server.yml` file.
  - Refer to Step 3 in the [Install Elasticsearch, Kibana and APM Server - Development Environment](../elasticsearch_setup_development.md) guide for complete configuration details.

* **Verify Self-Instrumentation:**
  - After configuration, restart the APM Server service:
    ```powershell
    Restart-Service apm-server
    ```

    <details>
   <summary>Expected response</summary>

   ```
   WARNING: Waiting for service 'apm-server
   (apm-server)' to stop...
   ```
   </details>

  - Check Kibana to verify that APM Server metrics are being collected

---

## 4. Service Verification

### 4.1 Verifying APM Server Health

**Symptoms:**
- Need to confirm APM Server is operating correctly
- Health check automation

**Troubleshooting Steps:**

* **Check APM Server Status:**
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

