# Data Grid Audit Troubleshooting

This document provides troubleshooting guidance for common issues that cause the Data Grid Audit tab in Relativity to fail or display errors. These issues can occur after initial setup, following an API key rotation, or when an existing API key has expired.

## Audit Tab Shows Authentication Error or No Results

### Symptoms

The Audit tab may display one of the following errors:

- **"Authentication failed for user to access Elasticsearch. Please check the permissions for the user."** — Relativity cannot authenticate against Elasticsearch with the current API key.

  ![Relativity Audit tab showing authentication failed error](../../resources/troubleshooting-images/audit-auth-failed.png)

- **"This chart did not return any results."** — Relativity can reach Elasticsearch but data has not loaded, typically seen after services have been restarted but Elasticsearch is still initialising.

  ![Relativity Audit tab showing no results](../../resources/troubleshooting-images/audit-no-results.png)

These errors can be caused by:
- An Elasticsearch API key that has **expired** or been **invalidated**
- Relativity services not yet restarted after an API key rotation
- The Elasticsearch service requiring a restart to accept the new key

---

## Step 1: Check the Elasticsearch Logs

On the Elasticsearch server, open the log file at:

```
C:\elastic\elasticsearch\logs\elasticsearch.log
```

Look for repeated `WARN` entries matching this pattern:

```
[WARN ][o.e.x.s.a.ApiKeyAuthenticator] [<node>] Authentication using apikey failed - api key [<key_id>] has been invalidated
```

![Elasticsearch log showing repeated invalidated API key warnings](../../resources/troubleshooting-images/elasticsearch-apikey-invalidated-log.png)

- **If this warning is present** — the API key Relativity uses to connect to the DataGrid Elasticsearch cluster has expired or been invalidated. Proceed to [Step 2: Rotate the Expired API Key](#step-2-rotate-the-expired-api-key).
- **If this warning is not present** — the key is valid but services may need a restart. Proceed to [Step 3: Restart Services](#step-3-restart-services).

---

## Step 2: Rotate the Expired API Key

When Elasticsearch logs confirm the API key has been invalidated, issue a new key using the Relativity Server CLI.

Follow the [Rotate an Elasticsearch API Key using the Relativity Server CLI](../elastic-stack-setup-02-environment-watch/elastic-stack-rotate-api-key-environment-watch.md) guide, targeting `rel-cluster-datagrid`.

After the rotation completes, continue with [Step 3: Restart Services](#step-3-restart-services) to restore the Audit tab.

---

## Step 3: Restart Services

Restarting Relativity services forces them to re-read the API key from the Secret Store. In some cases, the Elasticsearch service also requires a restart.

1. Open **Services** (`services.msc`) and restart the following Relativity services on **all servers** in the Relativity instance:
   - `kCura Edds Agent Manager`
   - `kCura Edds Web Processing Manager`
   - `kCura Service Host Manager`

2. Navigate to **Audit** > **Audit** in Relativity and refresh the page.

   The error may change to *"This chart did not return any results."* — this indicates Relativity can now reach Elasticsearch but data has not yet loaded.

3. On the Elasticsearch server, open **Services** and restart the **Elasticsearch** service.

4. Wait **1–5 minutes** for Elasticsearch to fully restart and for Relativity to re-establish the connection.

5. Reload the Relativity UI and navigate back to the **Audit** tab. Audit data should now display correctly.

> [!NOTE]
> If the Audit tab still does not load after following these steps, verify that the new API key was successfully persisted to the Secret Store. See [Rotate an Elasticsearch API Key using the Relativity Server CLI](../elastic-stack-setup-02-environment-watch/elastic-stack-rotate-api-key-environment-watch.md#verify-the-rotation) for Secret Store and Kibana verification steps.

