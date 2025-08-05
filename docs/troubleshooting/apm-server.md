# APM Server Troubleshooting

This document provides troubleshooting guidance for common APM Server issues encountered during installation, configuration, and operation in Relativity Server environments.

> [!NOTE]
> This guide assumes a default APM Server installation path of `C:\elastic\apm-server`. Adjust paths according to your actual installation directory.

## Table of Contents

- [1. Windows Service Issues](#1-windows-service-issues)
  - [1.1 APM Server Service Not Starting](#11-apm-server-service-not-starting)
  - [1.2 Service Crashes or Stops Unexpectedly](#12-service-crashes-or-stops-unexpectedly)
  - [1.3 Permission and Access Issues](#13-permission-and-access-issues)
- [2. Port Configuration Issues](#2-port-configuration-issues)
  - [2.1 Port Conflicts](#21-port-conflicts)
  - [2.2 Network Connectivity Problems](#22-network-connectivity-problems)
- [3. Service Verification](#3-service-verification)
  - [3.1 Verifying APM Server Health and Status](#31-verifying-apm-server-health-and-status)
- [4. Self-Instrumentation](#4-self-instrumentation)

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
   Stopped  apm-server  Elastic APM Server
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
   # Stop Windows service first, then test configuration syntax
   Stop-Service apm-server
   C:\elastic\apm-server\apm-server.exe test config -c "C:\elastic\apm-server\apm-server.yml"
   ```
   <details>
   <summary>Expected response</summary>

   ```
   Config OK
   ```
   </details>

**Start Service Manually:**
   ```powershell
   Start-Service apm-server
   ```

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

## 2. Port Configuration Issues

### 2.1 Port Conflicts

**Symptoms:**
- APM Server fails to bind to default port
- "bind: address already in use" errors in logs
- APM agents cannot connect to server

**Troubleshooting Steps:**

* **Check Default Port:**
  - Default APM Server port: 8200
  - Verify port availability:
    ```powershell
    netstat -an | findstr ":8200"
    ```
    <details>
    <summary>Expected response</summary>

    ```
    (No output if port is available. If you see LISTENING, port is in use.)
    ```
    </details>

* **Identify Port Conflicts:**
  ```powershell
  Get-NetTCPConnection -LocalPort 8200 -State Listen
  ```
  <details>
  <summary>Expected response</summary>

  ```
  (No results if port is available. If results are returned, another process is using port 8200.)
  ```
  </details>

> [!IMPORTANT]
> Do not change the APM Server port. Instead, identify and stop the conflicting service using port 8200, as changing the APM Server port requires extensive configuration changes across Environment Watch, Relativity, and other components.

---

### 2.2 Network Connectivity Problems

**Symptoms:**
- Service Not Running: APM Server or Elasticsearch may not be running or listening on the expected endpoints.
- Incorrect Configuration: The APM Server or Elasticsearch endpoint URLs may be misconfigured (wrong host, port, or protocol).
- Firewall Rules: Firewalls on the VM host or network may be blocking required ports (e.g., 9200 for Elasticsearch, 8200 for APM Server).

**Troubleshooting Steps:**

* **Verify Network Binding:**
  - Check `apm-server.yml` configuration:
    ```yaml
    apm-server:
      host: "0.0.0.0:8200"  # Listen on all interfaces
      # or
      host: "<hostname_or_ip>:8200"
    ```

* **Test Remote Connectivity:**
  ```powershell
  Test-NetConnection -ComputerName <hostname_or_ip> -Port 8200
  ```
  <details>
  <summary>Expected response</summary>

  ```
  TcpTestSucceeded : True
  ```
  </details>

---

## 3. Service Verification

### 3.1 Verifying APM Server Health and Status

**Symptoms:**
- Need to confirm APM Server is operating correctly
- Performance monitoring requirements
- Health check automation

**Troubleshooting Steps:**

* **Verify Server Configuration:**
   ```powershell
   C:\elastic\apm-server\apm-server.exe test config -c "C:\elastic\apm-server\apm-server.yml"
   ```
   <details>
   <summary>Expected response</summary>

   ```
   Config OK
   ```
   </details>

* **Check Elasticsearch Connection:**
   ```powershell
   # Stop Windows service first, then test output connectivity
   Stop-Service apm-server
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
       addresses: 192.168.1.100
       dial up... OK
     TLS... WARN secure connection disabled
     talk to server... OK
   ```
   </details>

---

## 4. Self-Instrumentation

To enable instrumentation of the APM Server itself, add the following section to your `apm-server.yml` file:

```yaml
instrumentation:

  # Set to true to enable instrumentation of the APM Server itself.
  enabled: true

  # Environment in which the APM Server is running (e.g., staging, production, etc.)
  environment: production

  # Hosts to report instrumentation results to.
  # For reporting to itself, leave this field commented or set to the local APM endpoint.
  hosts:
    - http://<hostname_or_ip>:8200
```

After updating the configuration, restart the APM Server service:
```powershell
Restart-Service apm-server
```


