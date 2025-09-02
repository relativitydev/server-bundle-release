# Secret Store Troubleshooting

This document provides a centralized guide for troubleshooting issues related to the Relativity Secret Store.

## Table of Contents

[1. Secret Store Access Verification](#1-secret-store-access-verification)

---

## 1. Secret Store Access Verification

### 1.1. Network Connectivity Test

Verify that the Secret Store host is reachable on port 443.

```powershell
Test-NetConnection -ComputerName <hostname_or_ip> -Port 443
```

<details>
<summary>Expected output</summary>

```
ComputerName     : <hostname_or_ip>
RemoteAddress    : <ip>
RemotePort       : 443
TcpTestSucceeded : True
```
</details>

### 1.2. API Access Test

1.  Open an elevated PowerShell and run the following command to list secrets and retrieve connection details:

    ```powershell
    C:\Program Files\Relativity Secret Store\Client\secretstore.exe secret list /
    ```

    The output will look similar to:
    ```
    Secret Store URL: https://<hostname_or_ip>:9090/
    Client Certificate Thumbprint: 20F8F2516EC86EBF993075F64B0C6EA6777A4F83
    ```

2.  Copy the **Client Certificate Thumbprint** and **Secret Store URL** from the output.

### 1.3. Seal Status Check

To check the seal status of the Secret Store, run the following script in an elevated PowerShell ISE.

1.  Replace `<insert-secret-store-client-certificate-thumbprint-here>` with the thumbprint you copied.
2.  Replace `<insert-secret-store-url-here>` with the URL you copied.

```powershell
$thumbprint = "<insert-secret-store-client-certificate-thumbprint-here>"
$url = "<insert-secret-store-url-here>"

# Find the client certificate
$store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My", "LocalMachine")
$store.Open("ReadOnly")
$cert = $store.Certificates | Where-Object { $_.Thumbprint -eq $thumbprint }

if (-not $cert) {
    Write-Error "Certificate with thumbprint $thumbprint not found."
    return
}

# Check the seal status
$response = Invoke-RestMethod -Uri "$url/v1/sys/seal-status" -Certificate $cert
$response | ConvertTo-Json
```

<details>
<summary>Expected output (for a healthy, unsealed store)</summary>

```json
{
    "type": "shamir",
    "initialized": true,
    "sealed": false,
    "t": 3,
    "n": 5,
    "progress": 0,
    "nonce": "",
    "version": "1.6.2",
    "migration": false,
    "cluster_name": "secret-store",
    "cluster_id": "...",
    "recovery_seal": false,
    "storage_type": "raft"
}
```
</details>
