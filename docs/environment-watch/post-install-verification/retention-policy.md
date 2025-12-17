# Post-Install Verification for Retention Policy
![Post-Install Verification Banner](../../../resources/post-install-verification-images/Post-installation-verification.svg)

## Verify Retention Policy Configuration

This verification step confirms that the retention period (data lifecycle) is properly configured for your Application Performance Monitoring(APM) data streams.

## Verification Steps

1. Navigate to Kibana Dev Tools Console:
   - Open Kibana in your web browser
   - Click on **Dev Tools** in the left navigation menu

2. Run the following queries to verify retention policies for each data stream type:

### Verify Logs Retention Policy

```
GET /_data_stream/logs-apm.app*?filter_path=data_streams.name,data_streams.lifecycle
```

### Verify Metrics Retention Policy

```
GET /_data_stream/metrics-apm.app*?filter_path=data_streams.name,data_streams.lifecycle
```

### Verify Traces Retention Policy

```
GET /_data_stream/traces-apm*?filter_path=data_streams.name,data_streams.lifecycle
```

## Expected Results

Each query should return the data stream names along with their configured lifecycle settings.

**Sample Output:**

```json
{
  "data_streams": [
    {
      "name": "logs-apm.app.relsvr_logging-default",
      "lifecycle": {
        "enabled": true,
        "data_retention": "90d",
        "effective_retention": "90d",
        "retention_determined_by": "data_stream_configuration"
      }
    }
  ]
}
```

## What to Check

- **enabled**: Should be `true`
- **data_retention**: Indicates the configured retention period (e.g., "30d" for 30 days, "90d" for 90 days)

If the lifecycle settings don't match your expected configuration, you may need to update your retention period according to [elasticsearch_retention_policy_guidelines.md](../../elasticsearch_retention_policy_guidelines.md).
