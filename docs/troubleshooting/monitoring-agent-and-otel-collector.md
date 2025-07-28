# Environment Watch Monitoring Agent and Open Telemetry Collector Troubleshooting

This document provides troubleshooting guidance for the Relativity Environment Watch Windows service and the Open Telemetry Collector in Relativity environments.

> [!NOTE]
> The Relativity Environment Watch Windows service is responsible for auto-configuring and managing the Open Telemetry Collector on each server. There are no expectations for the user to configure the collector directly.

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
- [Open Telemetry YAML File Auto-Generation](#open-telemetry-yaml-file-auto-generation)
- [Monitoring Agent Validation](#monitoring-agent-validation)

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
  - To check port status:
    ```powershell
    netstat -an | findstr ":4318"
    Get-NetTCPConnection -LocalPort 4318 -State Listen
    ```
    **Expected result:** Output shows port 4318 is listening only when the service is running.

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
  - `C:\ProgramData\Relativity\EnvironmentWatch\Services\InfraWatchAgent\Logs\otelcol-relativity-stderr.log` (errors)
  - `C:\ProgramData\Relativity\EnvironmentWatch\Services\InfraWatchAgent\Logs\otelcol-relativity-stdout.log` (general logs)
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
  **Expected response:** A valid JSON response with secret data.

### BCP Share Access Verification

> If you are not logged in as the Relativity Service Account, use the commands below.

- Test as the Relativity Service Account:
  ```powershell
  $cred = Get-Credential # use the Relativity Service Account
  Start-Process "powershell.exe" -Credential $cred -ArgumentList '-NoExit'
  # In the new PowerShell session window, run:
  Test-Path "\\bcp-share\relativity-data"
  ```

### Kepler (SSL Certificate) Verification

- The required web certificate must be installed on the server.
  (Check with your certificate management process or MMC snap-in for Certificates.)
- To verify Kepler API status:
  ```powershell
  curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>/relativity.rest/api/relativity-infrawatch-services/v1/infrawatch-manager/getkeplerstatusasync"
  ```
  **Expected response:** A JSON status indicating Kepler is healthy.

### Elasticsearch Access Verification

- For connectivity and troubleshooting, see [Elasticsearch Troubleshooting](../elasticsearch.md).

### APM Server Access Verification

- For connectivity and troubleshooting, see [APM Server Troubleshooting](../apm-server.md).

### Relativity Service Account Verification

- For service account requirements and troubleshooting, see [Environment Watch Installer](../environment_watch_installer.md).

---

## Open Telemetry YAML File Auto-Generation

- The Open Telemetry YAML file is automatically created by the Environment Watch Windows service at:
  - `C:\ProgramData\Relativity\EnvironmentWatch\Services\InfraWatchAgent\otelcol-config-auto-generated.yaml`

---

## Monitoring Agent Validation

To verify that metrics, logs, and traces are flowing from the Open Telemetry Collector:

1. **Monitoring Agents Dashboard**
   - Open Kibana and navigate to the **Monitoring Agents** dashboard.
   - Confirm the "Last Check-In" field is updating for each monitored server.
   - This proves the heartbeat metric is working.

2. **Host Infrastructure Overview**
   - Open Kibana and navigate to the **Host Infrastructure Overview** dashboard.
   - Confirm metrics are present for each host.

3. **Discover Query**
   - Open Kibana, go to **Discover**.
   - Select the APM data view.
   - Run the following query:
     ```
     service.name: "relsvr_infrawatch_agent" and host.hostname: "<hostname_or_ip>"
     ```
   - **Expected result:** You should see logs, metrics, and traces from the Open Telemetry Collector for the specified host.

   ![Sample Discover Query Result](../troubleshooting-images/monitoring-agent-discover-result.png)

---

For additional troubleshooting, refer to the main [Environment Watch documentation](../environment_watch_installer.md).

