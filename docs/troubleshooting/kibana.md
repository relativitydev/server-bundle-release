# Kibana Troubleshooting

This document provides troubleshooting guidance for common Kibana issues encountered during installation, configuration, and operation in Relativity environments.

> [!NOTE]
> This guide assumes a default Kibana installation path of `C:\elastic\kibana`. Adjust paths according to your actual installation directory.

## Table of Contents

- [1. Windows Service Issues](#1-windows-service-issues)
  - [1.1 Kibana Service Not Starting](#11-kibana-service-not-starting)
  - [1.2 Service Crashes or Stops Unexpectedly](#12-service-crashes-or-stops-unexpectedly)
- [2. Authentication Issues](#2-authentication-issues)
  - [2.1 Username/Password Authentication Issues](#21-usernamepassword-authentication-issues)
- [3. Port Configuration Issues](#3-port-configuration-issues)
  - [3.1 Port Conflicts](#31-port-conflicts)
  - [3.2 Network Binding Problems](#32-network-binding-problems)
- [4. Memory Issues](#4-memory-issues)
  - [4.1 Insufficient Memory Allocation](#41-insufficient-memory-allocation)
- [5. Kibana Encryption Keys Configuration](#5-kibana-encryption-keys-configuration)
  - [5.1 Missing or Invalid Encryption Keys](#51-missing-or-invalid-encryption-keys)
- [6. Service Verification](#6-service-verification)
  - [6.1 Verifying Kibana Health and Status](#61-verifying-kibana-health-and-status)
- [7. Additional Diagnostic Commands](#7-additional-diagnostic-commands)

---


## 1. Windows Service Issues

> [!IMPORTANT]
> The following troubleshooting steps apply **only if Kibana was set up as a Windows service** (e.g., using NSSM or a similar tool). If you did not install Kibana as a Windows service, you must ensure that `kibana.bat` is running in a command prompt or as a scheduled task.

### 1.1 Kibana Service Not Starting

**Symptoms:**
- Kibana service fails to start
- Service stops immediately after starting

**Troubleshooting Steps:**

* **Check Service Status and Configuration:**
   ```powershell
   Get-Service -Name kibana
   ```
   <details>
   <summary>Expected output</summary>

   ```
   Status   Name   DisplayName
   ------   ----   -----------
   Running  kibana Kibana
   ```
   </details>

   ```powershell
   Get-CimInstance -ClassName Win32_Service -Filter "Name='kibana'" | Select-Object Name, State, StartMode, StartName
   ```
   - Verify the service is running under Local System account (default configuration).

* **Check Kibana Logs:**
  - Navigate to `C:\elastic\kibana\logs\`
  - Review the latest log files (`kibana.log`) for error messages.
  - Look for configuration errors or Elasticsearch connection issues (for example: `Unable to connect to Elasticsearch at https://<host or ip address>:9200`).

* **Start Service Manually:**
   ```powershell
   Start-Service kibana
   ```
   <details>
   <summary>Expected output</summary>

   ```
   (No output if successful. Service status will be "Running" after execution.)
   ```
   </details>

### 1.2 Service Crashes or Stops Unexpectedly

**Symptoms:**
- Kibana service starts but stops after a short period
- Service status shows "Stopped" unexpectedly
- Users lose access to Kibana interface

**Troubleshooting Steps:**

* **Check Kibana Logs:**
  - Navigate to `C:\elastic\kibana\logs\`
  - Review the latest log files (`kibana.log`) for crash details
  - Look for memory issues or connection failures

* **Review Kibana Configuration:**
  - Check `C:\elastic\kibana\config\kibana.yml` file
  - Verify Elasticsearch connection settings
  - Ensure all required configuration parameters are present

* **Verify Elasticsearch Connectivity:**
   ```powershell
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```
   <details>
   <summary>Expected output</summary>

   ```json
   {
      "name" : "EMTTEST",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "PwBZoINKQjGZ53WH4gFfBg",
      "version" : {
         "number" : "8.17.3",
         "build_flavor" : "default",
         "build_type" : "zip",
         "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
         "build_date" : "2025-02-28T10:07:26.089129809Z",
         "build_snapshot" : false,
         "lucene_version" : "9.12.0",
         "minimum_wire_compatibility_version" : "7.17.0",
         "minimum_index_compatibility_version" : "7.0.0"
      },
      "tagline" : "You Know, for Search"
   }
   ```
   </details>

* **Check Memory Usage:**
  - Monitor Kibana process memory consumption using Task Manager or Resource Monitor.
  - Verify sufficient system memory is available.

---

## 2. Authentication Issues

### 2.1 Username/Password Authentication Issues

**Symptoms:**
- Login failures at Kibana interface
- "Invalid username or password" errors
- Users cannot access Kibana dashboards

**Troubleshooting Steps:**

* **Verify User Configuration:**
  - Ask the user to log in using the `elastic` username credentials.
  - Check `C:\elastic\kibana\config\kibana.yml`:
    ```yaml
    elasticsearch.username: "<username>"
    elasticsearch.password: "<password>"
    ```

* **Test Elasticsearch Credentials Independently:**
  - Use the `elastic` username and password from `kibana.yml` to verify connectivity to Elasticsearch:
    ```powershell
    curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
    ```
    <details>
    <summary>Expected output</summary>

    ```json
    {
       "name" : "EMTTEST",
       "cluster_name" : "elasticsearch",
       ...
    }
    ```
    </details>



---



---

## 3. Port Configuration Issues

### 3.1 Port Conflicts

**Symptoms:**
- Kibana fails to bind to default port
- "EADDRINUSE" errors in logs
- Cannot access Kibana web interface

**Troubleshooting Steps:**

* **Check Default Port:**
  - Default Kibana port: 5601
  - Verify port availability:
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
   <details>
   <summary>Expected output</summary>

   ```json
   {
     "level": "available",
     "summary": "All services and plugins are available"
   }
   ```
   </details>



* **Update Firewall Rules:**
   ```powershell
   New-NetFirewallRule -DisplayName "Kibana Web Interface" -Direction Inbound -Protocol TCP -LocalPort 5601 -Action Allow
   ```
   <details>
   <summary>Expected output</summary>

   ```
   (No output if successful. Rule will appear in Windows Firewall.)
   ```
   </details>

### 3.2 Network Binding Problems

**Symptoms:**
- Cannot access Kibana from remote hosts
- Connection refused errors
- Kibana only accessible from localhost

**Troubleshooting Steps:**

* **Verify Network Configuration:**
  - Check `C:\elastic\kibana\config\kibana.yml` configuration:
    ```yaml
    server.host: "0.0.0.0"  # For all interfaces
    # or
    server.host: "<hostname_or_ip>"
    ```



---

## 4. Memory Issues

### 4.1 Insufficient Memory Allocation

**Symptoms:**
- Kibana becomes unresponsive
- Slow loading of dashboards and visualizations
- Out of memory errors in logs

**Troubleshooting Steps:**

* **Check Current Memory Usage:**
   ```powershell
   Get-Process -Name node | Where-Object {$_.ProcessName -eq "node"} | Select-Object WorkingSet, VirtualMemorySize
   ```
   <details>
   <summary>Expected output</summary>

   ```
   WorkingSet VirtualMemorySize
   ---------- ----------------
   12345678   23456789
   ```
   </details>

* **Review Out of Memory Errors in Logs:**
  - Check for "out of memory" or "heap" errors in `C:\elastic\kibana\logs\kibana.log`.
> [!NOTE]
> Out of memory errors typically indicate insufficient system memory or improper Node.js heap settings. Review the error details in the log for specific causes and recommended actions.

* **Verify Disk Space:**
  - Insufficient disk space can also cause memory-related failures.
  - For disk space troubleshooting steps, see [Verify Disk Space](elasticsearch.md#verify-disk-space).

---

## 5. Kibana Encryption Keys Configuration

### 5.1 Missing or Invalid Encryption Keys

**Symptoms:**
- Kibana fails to start with encryption-related errors
- "Saved objects encryption key is missing" warnings
- Unable to save or retrieve saved objects

**Troubleshooting Steps:**

* **Generate Encryption Keys:**
   ```powershell
   cd C:\elastic\kibana\bin
   .\kibana-encryption-keys.bat generate
   ```
   <details>
   <summary>Expected output</summary>

   ```
   xpack.encryptedSavedObjects.encryptionKey: "<randomly-generated-key-1>"
   xpack.reporting.encryptionKey: "<randomly-generated-key-2>"
   xpack.security.encryptionKey: "<randomly-generated-key-3>"
   ```
   </details>

* **Configure Required Encryption Keys in kibana.yml:**
  - Copy the generated lines above and paste them into your `C:\elastic\kibana\config\kibana.yml` file.

---

## 6. Service Verification

### 6.1 Verifying Kibana Health and Status

**Symptoms:**
- Need to confirm Kibana is operating correctly
- Performance monitoring requirements
- Health check automation

**Troubleshooting Steps:**

* **Check Kibana Status:**
   ```powershell
   curl.exe -k -u <username>:<password> -X GET "http://<hostname_or_ip>:5601/api/status"
   ```
   <details>
   <summary>Expected output</summary>

   ```json
   {
     "name": "kibana",
     "version": {
       "number": "8.x.x"
     },
     "status": {
       "overall": {
         "level": "available"
       }
     }
   }
   ```
   </details>

---

## 7. Additional Diagnostic Commands

### 7.1 Configuration Validation

* **Validate YAML syntax**
  ```powershell
  C:\elastic\kibana\bin\kibana.bat --validate-config
  ```
  <details>
  <summary>Expected output</summary>

  ```
  Configuration is valid
  ```
  </details>

* **Check current configuration**
  ```powershell
  C:\elastic\kibana\bin\kibana.bat --config-path="C:\elastic\kibana\config\kibana.yml" --dry-run
  ```
  <details>
  <summary>Expected output</summary>

  ```
  Configuration loaded successfully
  ```
  </details>

* **Check current configuration**
  ```powershell
  C:\elastic\kibana\bin\kibana.bat --config-path="C:\elastic\kibana\config\kibana.yml" --dry-run
  ```
  <details>
  <summary>Expected output</summary>

  ```
  Configuration loaded successfully
  ```
  </details>

