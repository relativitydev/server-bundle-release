# Enable Data Grid Audit

> [!NOTE]
> This section applies to Datagrid Only.

After installing the required Elastic components for Data Grid Audit, the integration between Elastic and Relativity is configured by running the Relativity Server CLI on the Primary SQL Server.

> Please review the following important information before proceeding:
> * **For Existing Data Grid Audit Customers:** You must be on Elasticsearch 7.17 or later the first time you run the Relativity Server CLI for Data Grid Audit against this cluster — this applies regardless of how long Data Grid Audit has been in use.
> * Before upgrading to Elasticsearch 8.x or 9.x, the `ESIndexCreationSetting` may need to be updated. For details, refer to the [Instance setting Details](https://help.relativity.com/Server2024/Content/System_Guides/Instance_Setting_Guide/Instance_setting_descriptions.htm#ESIndexCreationSettings).
> * Always verify the minimum required Elasticsearch version in your specific release bundle, as it may differ from the versions mentioned here.

### Prerequisites

1. Install the mapper-size plugin on all nodes in the Elasticsearch cluster (instructions available [here](https://www.elastic.co/guide/en/elasticsearch/plugins/current/mapper-size.html)). The Elasticsearch service must be restarted on each node after installing the plugin.

2. The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z`

3. Verify that the InfraWatch Services application is installed in the Relativity instance (this RAP is delivered as part of the base Relativity Server 2024 installation package). To confirm, open the Relativity **Application Library** and verify that **InfraWatch Services** appears in the installed application list.

4. **Network accessibility from the SQL Primary server** — The CLI runs on the SQL Primary server and makes outbound HTTPS connections to both Relativity and Elasticsearch. Before running the CLI, verify that both endpoints are reachable from the SQL Primary server:

   ```powershell
   Test-NetConnection -ComputerName <relativity-hostname> -Port 443
   Test-NetConnection -ComputerName <elasticsearch-masternode-hostname> -Port 9200
   ```

   Example output:

   ```
   PS C:\Users\relserviceaccount> Test-NetConnection -ComputerName <relativity-hostname> -Port 443

   ComputerName     : <relativity-hostname>
   RemoteAddress    : <relativity-ip-address>
   RemotePort       : 443
   InterfaceAlias   : Ethernet 3
   SourceAddress    : <sql-primary-ip-address>
   TcpTestSucceeded : True


   PS C:\Users\relserviceaccount> Test-NetConnection -ComputerName <elasticsearch-masternode-hostname> -Port 9200

   ComputerName     : <elasticsearch-masternode-hostname>
   RemoteAddress    : <elasticsearch-masternode-ip-address>
   RemotePort       : 9200
   InterfaceAlias   : Ethernet 3
   SourceAddress    : <sql-primary-ip-address>
   TcpTestSucceeded : True
   ```

   Both must return `TcpTestSucceeded : True` before proceeding. If either fails, resolve the network or firewall issue before continuing.

5. **SSL/TLS certificate trust — SQL Primary server** — The SSL certificates for both Relativity and Elasticsearch must be trusted on the SQL Primary server before running the CLI. Certificate trust on all other servers (Web Servers, Agent Servers) is covered in prerequisite 6.

   **Relativity SSL certificate:**
   - The SSL certificate must be trusted on the SQL Primary server, and must be issued to the same hostname you provide as the Relativity instance URL. A mismatch between the certificate hostname and the URL will cause SSL validation to fail even if the certificate itself is valid.
   - To verify from the SQL Primary server (curl will prompt for the password):
     ```powershell
     curl.exe -v -u <relativity-admin-username> "https://<relativity-hostname>/Relativity/"
     ```
   - **Success** — any HTTP response, including `401 Unauthorized`, confirms the TLS handshake completed and the certificate is trusted:
     ```
     * SSL connection using TLSv1.2 / ...
     < HTTP/1.1 401 Unauthorized
     ```
   - **Failure** — a `curl: (60)` error means the certificate is not trusted on this server:
     ```
     curl: (60) SSL certificate problem: unable to get local issuer certificate
     ```

   **Elasticsearch SSL certificate:**
   - The Elasticsearch cluster certificate must be trusted on the SQL Primary server.
   - To verify from the SQL Primary server (`--ssl-no-revoke` bypasses CRL/OCSP revocation checks, which often fail in environments where revocation endpoints are unreachable):
     ```powershell
     curl.exe --ssl-no-revoke -u <elasticsearch-admin-username> "https://<elasticsearch-masternode-hostname>:9200/"
     ```
   - **Success** — any HTTP response body confirms the certificate is trusted.
   - **Failure** — a `curl: (60)` error means the certificate is not trusted:
     ```
     curl: (60) SSL certificate problem: self-signed certificate in certificate chain
     ```

   If either command returns a `curl: (60)` error, import the relevant CA certificate into the Windows **Trusted Root Certification Authorities** store on the SQL Primary before proceeding. See [SSL/TLS Certificate Issues](./troubleshooting/pre-requisite-troubleshooting.md#ssltls-certificate-issues) for import instructions.

   **If using a private CA certificate:** After importing the certificate, if the CLI still fails with an SSL or connection error, .NET Framework 4.x on the server may be defaulting to TLS 1.0/1.1, which is rejected by corporate HTTPS endpoints. Set the following registry keys on the SQL Primary server and reboot before retrying the CLI. See [TLS Version Mismatch](./troubleshooting/pre-requisite-troubleshooting.md#tls-version-mismatch) for full steps.

   | Registry Key | Value Name | Value |
   | :--- | :--- | :--- |
   | `HKLM\SOFTWARE\Microsoft\.NETFramework\v4.0.30319` | `SchUseStrongCrypto` | `1` (DWORD) |
   | `HKLM\SOFTWARE\Wow6432Node\Microsoft\.NETFramework\v4.0.30319` | `SchUseStrongCrypto` | `1` (DWORD) |

6. **SSL/TLS certificate trust — Web Servers and Agent Servers** — The Elasticsearch SSL certificate must also be trusted on every Web Server and Agent Server in the Relativity environment. ARM jobs that include Data Grid content communicate directly with Elasticsearch from Agent Servers and will fail with 401/SSL errors if the certificate is not trusted there. **SQL Secondary servers do not require the Elasticsearch CA certificate** — they do not make direct connections to Elasticsearch for Data Grid Audit setup or operation.

   For each Web Server and Agent Server:
   1. Import the Elasticsearch CA certificate into the Windows **Trusted Root Certification Authorities** store. See [SSL/TLS Certificate Issues](./troubleshooting/pre-requisite-troubleshooting.md#ssltls-certificate-issues) for import steps.
   2. Restart all Relativity services on the host (`kCura Edds Agent Manager`, `kCura Edds Web Processing Manager`, `kCura Service Host Manager`).

   > **Note:** A working Audit tab does **not** confirm that Agent Servers have the certificate — the Audit tab communicates through the web tier, while ARM agents connect to Elasticsearch directly.

7. **Relativity admin account permissions** — The Relativity admin account used with the CLI must:
   - Be a member of the **System Administrators** group in Relativity.
   - Have read/write access to the **Secret Store**.
   - **Not have two-factor authentication (2FA) enabled.** The CLI cannot complete an interactive 2FA challenge. Using an account with 2FA enforced will result in authentication failures during setup.
   - The CLI must be run from an **elevated (Run as Administrator) command prompt or PowerShell** on the SQL Primary server — see Step 1 of the setup instructions.

8. **Elasticsearch admin account** — The Elasticsearch credential provided to the CLI must have **superuser** privileges (or equivalent cluster-level read/write permissions). Using a limited-privilege account will result in `Unauthorized` errors during API key creation and index operations.

9. **Instance URL requirements** — Providing the wrong Relativity or Elasticsearch URL is one of the most common causes of setup failure. Review these requirements before running the CLI.

   **Relativity instance URL:**
   - Use the **load balancer or primary web server hostname** that is reachable from the SQL Primary server, in the format `https://<domainurl>/Relativity`.
   - Do **not** use a private or internal hostname that resolves differently from the SQL server than from workstations. The CLI makes REST API calls from the SQL Primary server, so the URL must resolve and be routable from that machine.
   - The URL must use HTTPS and the certificate at that hostname must be trusted on the SQL Primary server (see prerequisite 5).
   - Example: `https://<domainurl>/Relativity`

   **Elasticsearch cluster endpoint URL:**
   - Use the **master node hostname** — do not use a data node URL. The CLI communicates with the cluster through the master/coordinating node, which handles cluster-level operations such as API key creation and index management.
   - The default port is `9200`. The URL must use HTTPS if TLS is enabled on the cluster.
   - Use a hostname that matches the **Subject Alternative Name (SAN) or Common Name (CN)** on the Elasticsearch TLS certificate. Using an IP address or alternate hostname not covered by the certificate will cause a certificate mismatch (SSL error) even if the certificate is otherwise trusted.
   - Example: `https://<elasticsearch-masternode-hostname>:9200`

   > **Tip:** If your environment uses a load balancer in front of Elasticsearch, confirm that the load balancer certificate covers the hostname you are providing, and that the backend nodes are also individually accessible for certificate validation. When in doubt, use the individual node hostname that matches the certificate CN/SAN.

### Set up instructions

> **Note:** The CLI setup is safe to re-run. If a run fails partway through, simply re-execute `.\relsvr.exe setup` — no manual cleanup is needed. The CLI will recreate API keys, update secrets, and reapply all configuration from the beginning. There is no partial state that needs to be removed before retrying.

Follow these steps to set up Data Grid Audit using the Relativity Server CLI. All setup will occur on the SQL Primary server.

1. Open an **elevated command prompt or PowerShell** on the SQL Primary server (right-click → **Run as Administrator**). Run the following command and select **DataGrid**:
    ```
    C:\Server.Bundle.x.y.z\relsvr.exe setup
    Relativity Server CLI - 24.0.1196
    Copyright (c) 2025, Relativity ODA LLC

    What would you like to setup?
    > DataGrid
      Environment watch
      Exit
    ```

2. Enter the required Relativity and Elasticsearch parameters.

    ```
    Confirm you would like to perform the 'DataGrid' setup [y/n] (y): y

    Existing settings do not exist
    Enter the Relativity admin username (<relativity-admin-username>): <relativity-admin-username>
    Enter the Relativity admin password: *********
    Enter the Relativity instance url (https://<domainurl>/Relativity): https://<domainurl>/Relativity
    Relativity instance is verified
    Enter the Elasticsearch admin username (elastic): elastic
    Enter the Elasticsearch admin password: *********
    Enter the Elasticsearch cluster endpoint URL (https://<elasticsearch-masternode-hostname>:9200): https://<elasticsearch-masternode-hostname>:9200

    ```

    | Parameter | Description | Example |
    | :--- | :--- | :--- |
    | Relativity admin username | The username of a Relativity System Administrator account. Must use Forms Authentication with two-factor authentication disabled. | `<relativity-admin-username>` |
    | Relativity admin password | The password for the Relativity admin account. | |
    | Relativity instance URL | The HTTPS URL of the Relativity web server or load balancer, reachable from the SQL Primary server. Must end with `/Relativity`. | `https://<domainurl>/Relativity` |
    | Elasticsearch admin username | The username of an Elasticsearch account with superuser privileges. | `elastic` |
    | Elasticsearch admin password | The password for the Elasticsearch admin account. | |
    | Elasticsearch cluster endpoint URL | The HTTPS URL of the Elasticsearch **master**. Do not use a data node URL. The hostname must match the CN/SAN on the Elasticsearch TLS certificate. Default port is `9200`. | `https://<elasticsearch-masternode-hostname>:9200` |

3. Wait for Setup to Complete.

    ```
    Elasticsearch cluster endpoint URL is verified
    Elasticsearch plugin verified

    API Key creation and validation completed ------------------------- 100%
    Relativity instance setting validation completed ------------------ 100%
    Relativity secret store updated ----------------------------------- 100%
    Elastic Stack settings validation completed ----------------------- 100%
    Relativity toggles validation completed --------------------------- 100%

    The Relativity Data Grid setup has been completed. Please restart all Relativity services, including "kCura Edds Agent Manager," "kCura Edds Web Processing Manager," and "kCura Service Host Manager" on each server contained within this Relativity instance to complete the setup.
    ```

    If the setup completes successfully, Datagrid is now configured for the environment.

> **Warning:** Before restarting services, check whether the `NewDataGridMigratorToggleOverwrite` instance setting exists in your Relativity instance. If it is present, it **must** be set to `True` before running any ARM jobs — otherwise ARM jobs will fail during the Audit Migration stage and may time out and discard all migration progress. If the setting is not present, no action is required.

4. Restart the following Relativity services on **all machines** in the Relativity instance: `kCura Edds Agent Manager`, `kCura Edds Web Processing Manager`, and `kCura Service Host Manager`.

5. Verify the Audit setup — navigate to the Audit tab in the Relativity environment and confirm all of the following:
   - Recent audit events are visible and populating (new user actions appear within a few minutes).
   - No error banners or "Elasticsearch connection failed" messages are displayed.
   - Audit search returns results without errors.
