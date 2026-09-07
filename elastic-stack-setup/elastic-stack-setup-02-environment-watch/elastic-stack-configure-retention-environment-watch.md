# Configure Elasticsearch ILM Retention using the Relativity Server CLI

The `configure-retention` command sets Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long monitoring data is retained in Elasticsearch for the Environment Watch InfraWatch cluster.

> [!NOTE]
> It is recommended to run the CLI from the Primary SQL Server.

> This guide assumes the Relativity Server bundle was extracted to `C:\Server.Bundle.x.y.z` or a similar directory chosen by the user.

## Prerequisites

- The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z`
- Access to the Relativity Secret Store (Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2025/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting.)
- Elasticsearch is running and reachable. Confirm by browsing to the cluster endpoint, `https://<hostname>:9200` (for example `https://emttest:9200`) — a running cluster returns its version and cluster details.
- The initial Environment Watch setup has been completed. See [Set up Environment Watch using the Relativity Server CLI](./elastic-stack-setup-02-environment-watch.md)

## Options

| Flag | Description | Default |
|------|-------------|---------|
| `--logs-days <value>` | Retention period in days for the logs ILM policy (`infrawatch-logs-policy`). Must be greater than 0. | Prompted interactively |
| `--metrics-days <value>` | Retention period in days for the metrics ILM policy (`infrawatch-metrics-policy`). Must be greater than 0. | Prompted interactively |
| `--traces-days <value>` | Retention period in days for the traces ILM policy (`infrawatch-traces-policy`). Must be greater than 0. | Prompted interactively |
| `--quiet` | Suppress all prompts and the confirmation gate. Credentials are read exclusively from the Secret Store. At least one `--*-days` flag must be supplied. Use for automated or scripted execution. | `false` |
| `--dryrun` | Preview the ILM policy JSON that would be submitted without making any changes to Elasticsearch. Compatible with both interactive and quiet modes. | `false` |

## Usage

### Interactive

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

### Interactive with a pre-filled default

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

### Quiet mode (automated / scripted)

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

### Dry run

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

### Secret Store has no Elasticsearch credentials

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

### Current retention state can't be fetched (interactive mode)

If the CLI can't reach Elasticsearch to read the existing ILM policies before showing prompts — for example the Elasticsearch service is stopped or the cluster is briefly unreachable — it warns and asks whether to continue anyway. Policies that simply don't exist yet do **not** trigger this warning; they are reported as `not set`.

```
Could not retrieve current ILM retention state. Proceed with caution.
Continue? [y/n] (n):
```

Declining prints `Operation cancelled.` and exits with no changes made, the same message used when all prompts are skipped or the final confirmation is declined. Before re-running, confirm the cluster is up by browsing to `https://<hostname>:9200`.

### Invalid retention value

Entering a non-numeric or non-positive value at a signal prompt re-prompts inline in place, rather than aborting:

```
Retention must be a positive number
```

## Verify the changes

### Kibana Dev Tools

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
