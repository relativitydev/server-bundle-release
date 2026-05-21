# Enable Data Grid Audit

> [!NOTE]
> This section applies to Datagrid Only.

After installing the required Elastic components for Data Grid Audit, the integration between Elastic and Relativity is configured by running the Relativity Server CLI on the Primary SQL Server.

> Please review the following important information before proceeding:
> * **For Existing Data Grid Audit Customers:** You must be on Elasticsearch 7.17 or later when initially running this setup.
> * Before upgrading to Elasticsearch 8.x or 9.x, the `ESIndexCreationSetting` may need to be updated. For details, refer to the [Instance setting Details](https://help.relativity.com/Server2024/Content/System_Guides/Instance_Setting_Guide/Instance_setting_descriptions.htm#ESIndexCreationSettings).
> * Always verify the minimum required Elasticsearch version in your specific release bundle, as it may differ from the versions mentioned here.
> * **Cutover from Custom Realms authentication (pre-Server 2024 Patch 1):** If your environment used Data Grid Audit before Server 2024 Patch 1, running this CLI setup is a one-time cutover — it replaces Custom Realms-based authentication with API key-based OAuth2 authentication. After a successful setup, you must update your Elasticsearch license from the Relativity-provided Platinum key to a free/Basic license (or your organization's own Platinum/Enterprise license). Open Kibana → **Stack Management → License Management** to update the license. See [Audit — Update to basic license for Elasticsearch](https://help.relativity.com/Server2024/Content/Elasticstack/One-Time_Relativity_Elastic_License_Migration_Platinum_to_Basic.htm) for full instructions.

### Prerequisites

1. Install the mapper-size plugin on all nodes in the Elasticsearch cluster (instructions available [here](https://www.elastic.co/guide/en/elasticsearch/plugins/current/mapper-size.html)). The Elasticsearch service must also be restarted after installing the plugin.

2. The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z`

3. Verify that the InfraWatch Services application is installed in the Relativity instance (this RAP is delivered as part of the base Relativity Server 2024 installation package).

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

5. **SSL/TLS certificate trust — SQL Primary server** — The SSL certificates for both Relativity (Kepler) and Elasticsearch must be trusted on the SQL Primary server before running the CLI. Certificate trust on all other servers (Web Servers, Agent Servers) is covered in prerequisite 6.

   **Relativity SSL certificate:**
   - The SSL certificate must be trusted on the SQL Primary server, and must be issued to the same hostname you provide as the Relativity instance URL. A mismatch between the certificate hostname and the URL will cause SSL validation to fail even if the certificate itself is valid.
   - To verify from the SQL Primary server:
     ```powershell
     curl.exe -u <username>:<password> "https://<relativity-hostname>/Relativity/Identity/Get"
     ```

   **Elasticsearch SSL certificate:**
   - The Elasticsearch cluster certificate must be trusted on the SQL Primary server.
   - To verify from the SQL Primary server:
     ```powershell
     curl.exe -u <username>:<password> -X GET "https://<elasticsearch-masternode-hostname>:9200/"
     ```

   A successful response (without `-k`) for each confirms the certificate is trusted. If a command only succeeds with `-k` (skip verification), import the relevant CA certificate into the Windows **Trusted Root Certification Authorities** store on the SQL Primary before proceeding. See [SSL/TLS Certificate Issues](./troubleshooting/pre-requisite-troubleshooting.md#ssltls-certificate-issues) for import instructions.

   **If using a private CA certificate:** After importing the certificate, if the CLI still fails with an SSL or connection error, .NET Framework 4.x on the server may be defaulting to TLS 1.0/1.1, which is rejected by corporate HTTPS endpoints. Set the following registry keys on the SQL Primary server and reboot before retrying the CLI. See [TLS Version Mismatch](./troubleshooting/pre-requisite-troubleshooting.md#tls-version-mismatch) for full steps.

   | Registry Key | Value Name | Value |
   | :--- | :--- | :--- |
   | `HKLM\SOFTWARE\Microsoft\.NETFramework\v4.0.30319` | `SchUseStrongCrypto` | `1` (DWORD) |
   | `HKLM\SOFTWARE\Wow6432Node\Microsoft\.NETFramework\v4.0.30319` | `SchUseStrongCrypto` | `1` (DWORD) |

6. **SSL/TLS certificate trust — Web Servers and Agent Servers** — The Elasticsearch SSL certificate must also be trusted on every Web Server and Agent Server in the Relativity environment. ARM jobs that include Data Grid content communicate directly with Elasticsearch from Agent Servers and will fail with 401/SSL errors if the certificate is not trusted there.

   For each Web Server and Agent Server:
   1. Import the Elasticsearch CA certificate into the Windows **Trusted Root Certification Authorities** store. See [SSL/TLS Certificate Issues](./troubleshooting/pre-requisite-troubleshooting.md#ssltls-certificate-issues) for import steps.
   2. Restart all Relativity services on the host (`kCura Edds Agent Manager`, `kCura Edds Web Processing Manager`, `kCura Service Host Manager`).

   > [!NOTE]
   > The Audit tab in Relativity working correctly does **not** confirm that Agent Servers have the certificate — the Audit tab communicates through the web tier, while ARM agents connect to Elasticsearch directly.

7. **Relativity admin account permissions** — The Relativity admin account used with the CLI must:
   - Be a member of the **System Administrators** group in Relativity.
   - Have read/write access to the **Secret Store**.
   - **Not have two-factor authentication (2FA) enabled.** The CLI cannot complete an interactive 2FA challenge. Using an account with 2FA enforced will result in authentication failures during setup.

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

   > [!TIP]
   > If your environment uses a load balancer in front of Elasticsearch, confirm that the load balancer certificate covers the hostname you are providing, and that the backend nodes are also individually accessible for certificate validation. When in doubt, use the individual node hostname that matches the certificate CN/SAN.

### Set up instructions

Follow these steps to set up Data Grid Audit using the Relativity Server CLI. All setup will occur on the SQL Primary server.

1. Open elevated command prompt/powershell. Run below command. Select **Datagrid**
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

4. Restart the Relativity services on all machines for the changes to take effect.

> [!WARNING]
> **Required for ARM jobs that include Data Grid content:** Check whether the `NewDataGridMigratorToggleOverwrite` instance setting exists in your Relativity instance. If it is present, it **must** be set to `True` before running any ARM jobs — otherwise ARM jobs will fail during the Audit Migration stage and may time out and discard all migration progress. If the setting is not present, no action is required.

5. Verify Audit Dashboard - navigate to the Audit tab in the Relativity environment and confirm that the dashboard and its data are loading correctly.

> [!NOTE]
> **Large or high-volume environments:** The `AuditRecord_PrimaryPartition` table uses an INT identity column with a maximum value of 2,147,483,647 rows. Environments with high audit activity can approach this limit over time, which causes all users to be locked out of Relativity. Proactively monitor the row count and convert the Audit ID column to BIGINT before the limit is reached. **Do not reset the identity sequence back to 1** — this creates duplicate Audit IDs that cause Data Grid and SQL search results to return different records for the same ID. Contact Relativity Support for the BIGINT conversion procedure.
