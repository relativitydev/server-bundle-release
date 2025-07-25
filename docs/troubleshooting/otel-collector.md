# Environment Watch Monitoring Agent and Open Telemetry Collector Troubleshooting

This document provides basic troubleshooting guidance for common Open Telemetry Collector issues encountered during installation, configuration, and operation in Relativity environments.

> [!NOTE]
The Relativity Environment Watch Windows service is responsible for auto-configuring and managing the Open Telemetry Collector on each server. There are no expectations for the user to configure the collector directly.

## Table of Contents

- [Windows Service Troubleshooting](#windows-service-troubleshooting)
- [Port Configuration Issues](#port-configuration-issues)
- [Error Handling and Diagnostics](#error-handling-and-diagnostics)
- [Pre-requisite Access Checks](#pre-requisite-access-checks)
  - [Secret Server Access Verification](#secret-server-access-verification)
  - [BCP Share Access Verification](#bcp-share-access-verification)
  - [Kepler (SSL Certificate) Verification](#kepler-ssl-certificate-verification)
  - [Elasticsearch Access Verification](#elasticsearch-access-verification)
  - [APM Server Access Verification](#apm-server-access-verification)
  - [Relativity Service Account Verification](#relativity-service-account-verification)
- [OTEL YAML File Auto-Generation](#otel-yaml-file-auto-generation)
- [Metric Scraper Validation](#metric-scraper-validation)

---

## Windows Service Troubleshooting

- Start service:
  ```powershell
  Start-Service -Name "Relativity Environment Watch"
  ```
- Stop service:
  ```powershell
  Stop-Service -Name "Relativity Environment Watch"
  ```
- Check status:
  ```powershell
  Get-Service 'Relativity Environment Watch'
  ```
- **Expected:**
  - Windows service starts/stops successfully.
  - When running, both `rel-envwatch-service` and `otelcol-relativity` processes are present.
  - When stopped, neither process is present.
  - Port 4318 is listening only when service is running.

- Fatal error handling:
  - Simulate fatal error (e.g., rename SecretStore registry key, break SQL/IdentityProviderURL/OAuth2Client settings).
  - Start service and verify:
    - Windows service fails to start or stops immediately.
    - Windows event log includes a clear error message.

---

## Port Configuration Issues

- Only one Open Telemetry Collector can run per environment.
- Default port: OTLP HTTP 4318.
- Check port:
  ```powershell
  netstat -an | findstr ":4318"
  Get-NetTCPConnection -LocalPort 4318 -State Listen
  ```
- Update firewall:
  ```powershell
  New-NetFirewallRule -DisplayName "Open Telemetry OTLP HTTP" -Direction Inbound -Protocol TCP -LocalPort 4318 -Action Allow
  ```

---

## Error Handling and Diagnostics

- All logs are written to:
  - `C:\ProgramData\Relativity\EnvironmentWatch\Services\InfraWatchAgent\Logs`
- All information and error log entries are also written to the Windows event log:
  - `Relativity.EnvironmentWatch`
- Use PowerShell to check logs:
  ```powershell
  Get-EventLog Relativity.EnvironmentWatch | Where { $_.EntryType -eq "Error" }
  Get-EventLog Relativity.EnvironmentWatch | Where { $_.EntryType -eq "Information" }
  Get-EventLog Relativity.EnvironmentWatch | Where { $_.Message -like "*Everything is ready*" }
  ```

---

## Pre-requisite Access Checks

### Secret Server Access Verification

- Test connectivity:
  ```powershell
  Test-NetConnection -ComputerName <hostname_or_ip> -Port 443
  ```
- Test API access:
  ```bash
  curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>/api/v1/secrets"
  ```

### BCP Share Access Verification

- Test as the Relativity Service Account:
  ```powershell
  $cred = Get-Credential # use the Relativity Service Account
  Start-Process "powershell.exe" -Credential $cred -ArgumentList '-NoExit'
  Test-Path "\\bcp-share\relativity-data"
  ```
- Or:
  ```powershell
  runas /user:domain\relativity-service-account "powershell -command Test-Path \\bcp-share\relativity-data"
  ```

### Kepler (SSL Certificate) Verification

- The required web certificate must be installed on the server.
  (Check with your certificate management process or MMC snap-in for Certificates.)

### Elasticsearch Access Verification

- Test connectivity:
  ```bash
  curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cluster/health"
  ```

### APM Server Access Verification

- Test connectivity:
  ```bash
  curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:8200/"
  ```

### Relativity Service Account Verification

- The Relativity Service Account must have full read/write permissions to:
  - `C:\ProgramData\Relativity\EnvironmentWatch` and all subfolders.
- Ensure "Log on as a service" right is assigned.
- If the service account password is changed, update the Windows service with the new password.

---

## OTEL YAML File Auto-Generation

- The OTEL YAML file is auto-generated at:
  - `C:\ProgramData\Relativity\EnvironmentWatch\Services\InfraWatchAgent\otelcol-config-auto-generated.yaml`
- When no Relativity/Invariant product is installed, the file contains only the config from `otelcol-config-all-hosts.yaml`.
- When products are installed, the file merges configs from the appropriate template files in `Templates` based on installed products.

---

## Metric Scraper Validation

- **Heartbeat Metric:**  
  - Query: `numeric_labels.heartbeat_count: *`  
  - Metrics published every 30 seconds.

- **Relativity Fileshare(s) Status:**  
  - Query: `relsvr.resourceserver_networkshare.accessible: *`  
  - Metrics published every 120 seconds.

- **Relativity BCP File Share:**  
  - Query: `relsvr.sqlbcp_networkshare.accessible: *`  
  - Metrics published every 120 seconds.

- **Primary SQL Metric:**  
  - Query: `relsvr.sqlserver_primary.running: *`  
  - Metrics published every 60 seconds.

- **Windows Service Metric:**  
  - Query: `relsvr.windows_service.running: *`  
  - Metrics published every 300 seconds.

- **X509 Certificate Metric:**  
  - Query: `relsvr.x509_certificate: *`  
  - Metrics published every 3600 seconds.

- **OAuth2 Client Metric:**  
  - Query: `numeric_labels.status_period_ms :* and labels.message : *OAuth2*`

- **Kibana Alert Metric:**  
  - Query: `labels.relsvr_alert_name :*`

> For each metric, verify documents are created for each monitored server and contain the expected fields as described in the test cases.

---

