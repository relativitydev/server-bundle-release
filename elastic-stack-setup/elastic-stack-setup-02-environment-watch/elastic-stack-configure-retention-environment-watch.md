# Configure Elasticsearch ILM Retention using the Relativity Server CLI

The `configure-retention` command sets Elasticsearch Index Lifecycle Management (ILM) retention policies for logs, metrics, and traces data streams. Use this command to control how long monitoring data is retained in Elasticsearch for the Environment Watch InfraWatch cluster.

> [!NOTE]
> It is recommended to run the CLI from the Primary SQL Server.

> This guide assumes the Relativity Server bundle was extracted to `C:\Server.Bundle.x.y.z` or a similar directory chosen by the user.

## Prerequisites

- The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z`
- Access to the Relativity Secret Store (Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2025/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting.)
- Elasticsearch is running and accessible
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

The command fetches and displays the current ILM retention values for all three signals, then prompts for each one individually. Press **Enter** at any signal prompt to skip that signal — the policy for that signal is left unchanged. If you press **Enter** at all prompts with no values entered, the command exits immediately with no confirmation prompt and no ILM changes made.

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Fetching current ILM retention policies...

  Logs    (infrawatch-logs-policy):    30d
  Metrics (infrawatch-metrics-policy): 30d
  Traces  (infrawatch-traces-policy):  7d

Configure logs retention in days (current: 30d, press Enter to skip): 60
Configure metrics retention in days (current: 30d, press Enter to skip):
Configure traces retention in days (current: 7d, press Enter to skip):

Changes to apply:
  Logs:    30d -> 60d
  Metrics: (no change)
  Traces:  (no change)

Apply changes? [yes/N]: yes

Updating ILM policies ------------------------------------------------- 100%

Successfully updated 1 ILM retention policy.
```

Entering anything other than `yes` at the confirmation prompt aborts cleanly with no changes made:

```
Operation cancelled.
```

### Interactive with a pre-filled default

Passing a `--*-days` flag in interactive mode pre-fills that signal's prompt with the flag value. The current value is still shown as context and confirmation is still required.

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention --logs-days 60

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Fetching current ILM retention policies...

  Logs    (infrawatch-logs-policy):    30d
  Metrics (infrawatch-metrics-policy): 30d
  Traces  (infrawatch-traces-policy):  7d

Configure logs retention in days (current: 30d, default: 60, press Enter to accept): 60
Configure metrics retention in days (current: 30d, press Enter to skip):
Configure traces retention in days (current: 7d, press Enter to skip):

Changes to apply:
  Logs:    30d -> 60d
  Metrics: (no change)
  Traces:  (no change)

Apply changes? [yes/N]: yes

Updating ILM policies ------------------------------------------------- 100%

Successfully updated 1 ILM retention policy.
```

### Quiet mode (automated / scripted)

Combining `--quiet` with one or more `--*-days` flags suppresses all prompts and the confirmation gate. Credentials come exclusively from the Secret Store — `relsvr setup` must have been run first. This is suitable for scheduled tasks or unattended automation scripts.

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention --quiet --logs-days 30 --metrics-days 90

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Updating ILM policies ------------------------------------------------- 100%

Successfully updated 2 ILM retention policies.
```

### Dry run

Use `--dryrun` to preview the ILM policy JSON that would be submitted without writing any changes to Elasticsearch. Dry run works in both interactive and quiet modes.

**Quiet dry run — no prompts:**

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention --quiet --logs-days 30 --dryrun

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Dry run mode — no ILM policies will be modified.
Dry run — ILM policy 'infrawatch-logs-policy' would be submitted with: {"policy":{"phases":{"delete":{"min_age":"30d","actions":{"delete":{}}}}}}
```

**Interactive dry run — prompts and confirmation appear, no changes applied after `yes`:**

```
C:\Server.Bundle.x.y.z\relsvr.exe configure-retention --dryrun

Relativity Server CLI - 102.1.26
Copyright (c) 2026, Relativity ODA LLC

Fetching current ILM retention policies...

  Logs    (infrawatch-logs-policy):    30d
  Metrics (infrawatch-metrics-policy): 30d
  Traces  (infrawatch-traces-policy):  7d

Configure logs retention in days (current: 30d, press Enter to skip): 30
Configure metrics retention in days (current: 30d, press Enter to skip):
Configure traces retention in days (current: 7d, press Enter to skip):

Changes to apply:
  Logs:    30d -> 30d
  Metrics: (no change)
  Traces:  (no change)

Apply changes? [yes/N]: yes

Dry run mode — no ILM policies will be modified.
Dry run — ILM policy 'infrawatch-logs-policy' would be submitted with: {"policy":{"phases":{"delete":{"min_age":"30d","actions":{"delete":{}}}}}}
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
