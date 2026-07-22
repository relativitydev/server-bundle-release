# Data Grid Audit Troubleshooting

This document provides troubleshooting guidance for common issues encountered with the Data Grid Audit tab in Relativity after initial setup or API key rotation.

## Audit Tab Does Not Load After API Key Rotation

### Symptoms

After rotating the DataGrid cluster API key using the `rotate-api-key` command, the Audit tab may display one of the following errors:

- **"Authentication failed for user to access Elasticsearch. Please check the permissions for the user."** — appears immediately after key rotation, before Relativity services have been restarted.
- **"This chart did not return any results."** — appears after restarting Relativity services, while Elasticsearch is still applying the new key.

### Cause

When the API key is rotated, the new key is persisted to the Relativity Secret Store. Relativity services must be restarted to read the updated key from the Secret Store. In some cases, the Elasticsearch service also requires a restart to fully accept connections using the new key.

### Resolution

1. After rotating the DataGrid API key, navigate to **Audit** > **Audit** in Relativity.

   If the page shows *"Authentication failed for user to access Elasticsearch. Please check the permissions for the user."*, proceed to step 2.

2. Open **Services** (`services.msc`) and restart the following Relativity services on **all servers** in the Relativity instance:
   - `kCura Edds Agent Manager`
   - `kCura Edds Web Processing Manager`
   - `kCura Service Host Manager`

3. Refresh the Audit tab in the browser. The error may change to *"This chart did not return any results."* — this indicates Relativity can now reach Elasticsearch but data has not yet loaded.

4. On the Elasticsearch server, open **Services** and restart the **Elasticsearch** service.

5. Wait **1–5 minutes** for Elasticsearch to fully restart and for Relativity to re-establish the connection.

6. Reload the Relativity UI and navigate back to the **Audit** tab. Audit data should now display correctly.

> [!NOTE]
> If the Audit tab still does not load after following these steps, verify that the new API key was successfully persisted to the Secret Store. See [Rotate an Elasticsearch API Key using the Relativity Server CLI](../elastic-stack-setup-02-environment-watch/elastic-stack-rotate-api-key-environment-watch.md#verify-the-rotation) for Secret Store and Kibana verification steps.

