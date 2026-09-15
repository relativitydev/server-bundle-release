# Elasticsearch Retention Policy - Guidelines

## Introduction

Environment Watch works out of the box using default Elasticsearch retention settings. Configuring custom retention policies is optional and typically unnecessary for dev environments. This guidance applies only when customers need to adjust retention to meet storage, performance, or compliance requirements.

### Purpose

These guidelines define retention policies for logs, metrics, and traces collected in Elasticsearch and viewed through Kibana. Proper retention management is critical for:

- **Storage Optimization & Cost Control** – Prevents excessive disk usage and reduces infrastructure costs by automatically removing outdated data
- **Performance Maintenance** – Keeps query response times fast by limiting the volume of searchable data
- **Compliance Adherence** – Ensures data is retained long enough to meet regulatory and audit requirements

### Impact of Improper Retention

Failing to configure appropriate retention policies can lead to:

- **Excessive Storage Usage** – Uncontrolled data growth consuming available disk space
- **Degraded Query Performance** – Large data volumes slow down search and aggregation operations
- **Risk of Data Loss** – Critical audit data may be prematurely deleted if retention is too short
- **Compliance Violations** – Insufficient retention periods may fail to meet legal or regulatory requirements
- **System Instability** – Disk space exhaustion can cause Elasticsearch cluster failures

---

## Retention Strategy

### Recommended Retention Periods

The following table provides baseline retention recommendations for different data types:

| Data Type | Default Retention | Recommended Retention |
|-----------|-------------------|----------------------|
| **Logs** | 10 days | 90 days |
| **Metrics** | 90 days | 90 days |
| **Traces** | 10 days | 30 days | 

> [!NOTE]
> The default retention values are configured out-of-the-box to minimize storage usage in new installations. The recommended retention periods represent industry best practices for Relativity environments, providing sufficient historical data for troubleshooting and trend analysis. Consider upgrading from default to recommended retention based on your organization's specific requirements, compliance obligations, and available storage capacity.

### Calculating Storage Requirements

Use the following formula to estimate storage requirements based on your Relativity environment size and desired retention period:

**Formula:**

```
Docs/Day (Daily Documents) = 6M + (Web_Server_Count × 2M) + (Agent_Server_Count × 2M) + (Worker_Server_Count × 400k) + (SQL_Distributed_Server_Count × 500k)

GiB/Day (Daily Storage) = Docs/Day × 380 / 1024³

Total Storage with Retention = GiB/Day × R (where R is retention in days)
```

**Understanding Daily Storage Calculation:**

The formula multiplies your daily document count by 380 (average bytes per document) and divides by 1024³ to convert bytes to Gibibytes. This gives you the storage space consumed per day. For example, 16.4 million documents × 380 bytes ÷ 1,073,741,824 bytes/GiB ≈ 5.8 GiB/day.

**Example Calculation:**

For an environment with 1 Web Server, 4 Agent Servers, 1 Worker, and 0 SQL Distributed Servers:

```
Docs/Day = 6M + (1 × 2M) + (4 × 2M) + (1 × 400k) + (0 × 500k)
         = 16.4M documents/day

GiB/Day = 16,400,000 × 380 / 1,073,741,824
        ≈ 5.8 GiB/day

Total Storage (90-day retention) = 5.8 × 90 ≈ 522 GiB (~0.5 TB)
Total Storage (10-day retention) = 5.8 × 10 ≈ 58 GiB
```

This calculation helps you understand the storage impact of different retention periods and plan your infrastructure accordingly.

### Factors Influencing Retention

When determining the appropriate retention period for your environment, consider:

- **Environment Size** – Development environments typically use default retention to minimize storage, while Small through X-Large environments benefit from recommended retention (90 days for logs/metrics, 30 days for traces) for better operational visibility and troubleshooting capabilities.

- **Storage Capacity and Cost** – Evaluate available disk space using the storage calculation formula above. Longer retention requires more storage investment, so balance retention needs against available capacity and infrastructure costs.

- **Regulatory Compliance** – Consult with legal and compliance teams to ensure retention settings meet your organization's regulatory obligations. Some industries and frameworks (HIPAA (Health Insurance Portability and Accountability Act), SOX (Sarbanes-Oxley Act), PCI DSS (Payment Card Industry Data Security Standard)) mandate specific retention periods for audit and logging data.

---

## Configure Elasticsearch ILM Retention using the Relativity Server CLI

The `configure-retention` command sets Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long monitoring data is retained in Elasticsearch for the Environment Watch InfraWatch cluster.

> [!NOTE]
> It is recommended to run the CLI from the Primary SQL Server.

> This guide assumes the Relativity Server bundle was extracted to `C:\Server.Bundle.x.y.z` or a similar directory chosen by the user.

### Prerequisites

- The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z`
- Access to the Relativity Secret Store (Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2025/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting.)
- Elasticsearch is running and reachable. Confirm by browsing to the cluster endpoint, `https://<hostname>:9200` (for example `https://emttest:9200`) — a running cluster returns its version and cluster details.
- The initial Environment Watch setup has been completed. See [Set up Environment Watch using the Relativity Server CLI](../elastic-stack-setup-02-environment-watch.md)

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--logs-days <value>` | Retention period in days for the logs ILM policy (`infrawatch-logs-policy`). Must be greater than 0. | Prompted interactively |
| `--metrics-days <value>` | Retention period in days for the metrics ILM policy (`infrawatch-metrics-policy`). Must be greater than 0. | Prompted interactively |
| `--traces-days <value>` | Retention period in days for the traces ILM policy (`infrawatch-traces-policy`). Must be greater than 0. | Prompted interactively |
| `--quiet` | Suppress all prompts and the confirmation gate. Credentials are read exclusively from the Secret Store. At least one `--*-days` flag must be supplied. Use for automated or scripted execution. | `false` |
| `--dryrun` | Preview the ILM policy JSON that would be submitted without making any changes to Elasticsearch. Compatible with both interactive and quiet modes. | `false` |

### Usage

#### Interactive

Running `configure-retention` without `--quiet` launches an interactive session. If `relsvr setup` has been run, credentials are fetched silently from the Secret Store — no prompt for cluster URL, admin username, or password. If setup has not been run, the CLI prompts for those credentials before continuing.

The command fetches and displays the current ILM retention values for all three signals, then prompts for each one individually. Press **Enter** at any signal prompt to skip that signal — the policy for that signal is left unchanged, and the summary below only lists the signals you actually changed.

> [!NOTE]
> On a first run, before the `infrawatch-*-policy` policies have been created, the fetch still succeeds — each signal is reported as `not set` rather than a number, the prompts show `[current: ?d, press Enter to skip]`, and the summary shows the old value as `not set` (for example `not set -> 21d`). This is expected on a new cluster and is not the same as the fetch failure described in [Current retention state can't be fetched](#current-retention-state-cant-be-fetched-interactive-mode) below.

If you press **Enter** at all three prompts, the command prints `Operation cancelled.` and exits immediately — it never shows the "Summary of changes" block or the "Apply these changes?" confirmation, because there is nothing to confirm:

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Configures Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long telemetry data is retained before Elasticsearch automatically deletes it.

Current ILM retention state:
  Logs: 30d
  Metrics: 30d
  Traces: 7d

Enter new retention for Logs in days [current: 30d, press Enter to skip]:
Enter new retention for Metrics in days [current: 30d, press Enter to skip]:
Enter new retention for Traces in days [current: 7d, press Enter to skip]:

Operation cancelled.
```

> [!NOTE]
> `Operation cancelled.` is the same message shown when you decline the confirmation prompt below, and when you decline to continue after a failed current-state fetch (see [Current retention state can't be fetched](#current-retention-state-cant-be-fetched-interactive-mode) below). All three cases look identical in the CLI output — none of them make any ILM changes.

Entering at least one value shows an old → new summary before applying anything:

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Configures Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long telemetry data is retained before Elasticsearch automatically deletes it.

Current ILM retention state:
  Logs: 30d
  Metrics: 30d
  Traces: 7d

Enter new retention for Logs in days [current: 30d, press Enter to skip]: 60
Enter new retention for Metrics in days [current: 30d, press Enter to skip]:
Enter new retention for Traces in days [current: 7d, press Enter to skip]:

Summary of changes:
  Logs: 30d -> 60d

Apply these changes? [y/n] (n): y

Configuring ILM retention policies on https://emttest:9200/...
  Applying Logs retention: 60 days (policy: infrawatch-logs-policy)...

Applying ILM retention policies...

  Logs retention set to 60 days.
Retention policies applied successfully.
```

> [!NOTE]
> `Applying ILM retention policies...` is shown as a spinner while the update is in progress, not a percentage progress bar — it disappears once the update finishes and is replaced by the per-signal success lines shown above. Only signals you changed appear in the summary and success output; unchanged signals are omitted entirely, not listed as "(no change)".

Entering anything other than `y` at the confirmation prompt aborts cleanly with no changes made:

```
Operation cancelled.
```

#### Interactive with a pre-filled default

Passing a `--*-days` flag in interactive mode pre-fills that signal's prompt with the flag value. The current value is still shown as context, pressing **Enter** accepts the pre-filled default without retyping it, and confirmation is still required.

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention --logs-days 60

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Configures Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long telemetry data is retained before Elasticsearch automatically deletes it.

Current ILM retention state:
  Logs: 30d
  Metrics: 30d
  Traces: 7d

Enter new retention for Logs in days [current: 30d, default: 60]:
Enter new retention for Metrics in days [current: 30d, press Enter to skip]:
Enter new retention for Traces in days [current: 7d, press Enter to skip]:

Summary of changes:
  Logs: 30d -> 60d

Apply these changes? [y/n] (n): y

Configuring ILM retention policies on https://emttest:9200/...
  Applying Logs retention: 60 days (policy: infrawatch-logs-policy)...

Applying ILM retention policies...

  Logs retention set to 60 days.
Retention policies applied successfully.
```

#### Quiet mode (automated / scripted)

Combining `--quiet` with one or more `--*-days` flags suppresses all prompts and the confirmation gate. Credentials come exclusively from the Secret Store — `relsvr setup` must have been run first. This is suitable for scheduled tasks or unattended automation scripts.

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention --quiet --logs-days 30 --metrics-days 90

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Configures Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long telemetry data is retained before Elasticsearch automatically deletes it.

Configuring ILM retention policies on https://emttest:9200/...
  Applying Logs retention: 30 days (policy: infrawatch-logs-policy)...
  Applying Metrics retention: 90 days (policy: infrawatch-metrics-policy)...

Applying ILM retention policies...

  Logs retention set to 30 days.
  Metrics retention set to 90 days.
Retention policies applied successfully.
```

#### Dry run

Use `--dryrun` to preview the ILM policy JSON that would be submitted without writing any changes to Elasticsearch. Dry run works in both interactive and quiet modes.

**Quiet dry run — no prompts, no update step at all:**

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention --quiet --logs-days 30 --dryrun

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Configures Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long telemetry data is retained before Elasticsearch automatically deletes it.

Dry run mode — no ILM policies will be modified.
Dry run — ILM policy 'infrawatch-logs-policy' would be submitted with: {"policy":{"phases":{"delete":{"min_age":"30d","actions":{"delete":{}}}}}}
```

**Interactive dry run — prompts and confirmation appear as usual, then a preview instead of an update:**

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention --dryrun

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Configures Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long telemetry data is retained before Elasticsearch automatically deletes it.

Current ILM retention state:
  Logs: 30d
  Metrics: 30d
  Traces: 7d

Enter new retention for Logs in days [current: 30d, press Enter to skip]: 30
Enter new retention for Metrics in days [current: 30d, press Enter to skip]:
Enter new retention for Traces in days [current: 7d, press Enter to skip]:

Summary of changes:
  Logs: 30d -> 30d

Apply these changes? [y/n] (n): y

Dry run mode — no ILM policies will be modified.
Dry run — ILM policy 'infrawatch-logs-policy' would be submitted with: {"policy":{"phases":{"delete":{"min_age":"30d","actions":{"delete":{}}}}}}
```

#### Secret Store has no Elasticsearch credentials

If `relsvr setup` has not been run — or the Elasticsearch secret has been removed from the Secret Store — the behavior depends on the mode.

**Interactive mode — the CLI prompts for the credentials:**

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Configures Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long telemetry data is retained before Elasticsearch automatically deletes it.

Enter the Elasticsearch cluster endpoint URL: https://emttest:9200
Enter the Elasticsearch admin username: elastic
Enter the Elasticsearch admin password: ********************

Current ILM retention state:
  Logs: not set
  Metrics: not set
  Traces: not set

Enter new retention for Logs in days [current: ?d, press Enter to skip]: 21
Enter new retention for Metrics in days [current: ?d, press Enter to skip]: 22
Enter new retention for Traces in days [current: ?d, press Enter to skip]: 23

Summary of changes:
  Logs: not set -> 21d
  Metrics: not set -> 22d
  Traces: not set -> 23d

Apply these changes? [y/n] (n): y
```

The command then applies the policies as normal. The credentials entered this way are used for that run only — run `relsvr setup` to store them in the Secret Store.

**Quiet mode — no prompt is shown, the command exits with an error:**

Because `--quiet` suppresses all prompts, the credentials cannot be supplied interactively and the run fails immediately:

```
Elasticsearch credentials are not available in the Secret Store. Run 'relsvr setup' first, or use interactive mode to enter credentials manually.
```

#### Current retention state can't be fetched (interactive mode)

If the CLI can't reach Elasticsearch to read the existing ILM policies before showing prompts — for example the Elasticsearch service is stopped or the cluster is briefly unreachable — it warns and asks whether to continue anyway. Policies that simply don't exist yet do **not** trigger this warning; they are reported as `not set`.

```
Could not retrieve current ILM retention state. Proceed with caution.
Continue? [y/n] (n):
```

Declining prints `Operation cancelled.` and exits with no changes made, the same message used when all prompts are skipped or the final confirmation is declined. Before re-running, confirm the cluster is up by browsing to `https://<hostname>:9200`.

#### Invalid retention value

Entering a non-numeric or non-positive value at a signal prompt re-prompts inline in place, rather than aborting:

```
Retention must be a positive number
```

### Verify the changes

#### Kibana Dev Tools

After running `configure-retention`, confirm the updated retention value in Kibana Dev Tools.

1. In Kibana, navigate to **Dev Tools** > **Console**.
2. Run the following query for each signal you updated, replacing `<signal>` with `logs`, `metrics`, or `traces`:

    ```
    GET /_ilm/policy/infrawatch-<signal>-policy
    ```

3. In the response, locate the `delete` phase and confirm `min_age` matches the value you set:

    ```json
    {
      "infrawatch-logs-policy": {
        "policy": {
          "phases": {
            "delete": {
              "min_age": "30d",
              "actions": {
                "delete": {}
              }
            }
          }
        }
      }
    }
    ```

---

## Advanced Configuration

For more advanced retention management using Index Lifecycle Management (ILM) policies with customizable phases (hot, warm, cold, delete), refer to the official Elasticsearch documentation:

- [Index Lifecycle Management (ILM) Overview](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)
- [Configure ILM Policies](https://www.elastic.co/guide/en/elasticsearch/reference/current/set-up-lifecycle-policy.html)
- [Data Stream Lifecycle vs ILM](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-stream-lifecycle.html)

ILM provides more granular control over data lifecycle phases and allows for tiered storage architectures in large-scale environments.
