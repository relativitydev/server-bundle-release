# Elasticsearch Retention Policy - Guidelines

## Introduction

### Purpose

These guidelines define retention policies for logs, metrics, and traces collected in Elasticsearch and viewed through Kibana. Proper retention management is critical for:

- **Storage Optimization** – Prevents excessive disk usage by automatically removing outdated data
- **Performance Maintenance** – Keeps query response times fast by limiting the volume of searchable data
- **Compliance Adherence** – Ensures data is retained long enough to meet regulatory and audit requirements
- **Cost Control** – Reduces infrastructure costs associated with storage expansion

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

- **Regulatory Compliance** – Consult with legal and compliance teams to ensure retention settings meet your organization's regulatory obligations. Some industries and frameworks (HIPAA, SOX, PCI DSS) mandate specific retention periods for audit and logging data.

---

## Configuration Steps

### Step 1: Create Component Template with Required Retention Policy

Elastic APM provides the `apm-90d@lifecycle` component template by default for 90-day retention. For 30-day retention (recommended for traces), create a custom component template using the Dev Tools Console in Kibana:

**Sample Request:**

```json
# Here apm-30d@lifecycle is the name of the component template 
PUT _component_template/apm-30d@lifecycle 
{
  "template": {
    "lifecycle": {
      "enabled": true,
      "data_retention": "30d"
    }
  },
  "_meta": {
    "managed": true,
    "description": "Data stream lifecycle for 30 days of retention"
  }
}
```

**Sample Output:**

```json
{
  "acknowledged": true
}
```

### Step 2: Update Index Templates

Update the following index templates to use the appropriate component template based on your retention requirements:

| Index Template | Data Type | Default Component | Recommended Component |
|---------------|-----------|-------------------|----------------------|
| `logs-apm.app@template` | Logs | `apm-10d@lifecycle` | `apm-90d@lifecycle` |
| `metrics-apm.app@template` | Metrics | `apm-90d@lifecycle` | `apm-90d@lifecycle` |
| `traces-apm@template` | Traces | `apm-10d@lifecycle` | `apm-30d@lifecycle` |

#### a. Get Current Index Template Configuration

Use the Dev Tools Console in Kibana to retrieve the existing index template settings:

```json
# Here logs-apm.app@template is the name of the index template 
GET _index_template/logs-apm.app@template 
```

**Sample Output:**

```json
{
  "index_templates": [
    {
      "name": "logs-apm.app@template",
      "index_template": {
        "index_patterns": [
          "logs-apm.app.*-*"
        ],
        "template": {
          "settings": {
            "index": {
              "mode": "standard",
              "default_pipeline": "logs-apm.app@default-pipeline",
              "final_pipeline": "logs-apm@pipeline"
            }
          }
        },
        "composed_of": [
          "logs@mappings",
          "apm@mappings",
          "apm@settings",
          "logs-apm@settings",
          "logs-apm.app-fallback@ilm",
          "ecs@mappings",
          "logs@custom",
          "logs-apm.app@custom",
          "apm-10d@lifecycle"
        ],
        "priority": 210,
        "version": 101,
        "_meta": {
          "managed": true,
          "description": "Index template for logs-apm.app.*-*"
        },
        "data_stream": {
          "hidden": false,
          "allow_custom_routing": false
        },
        "allow_auto_create": true,
        "ignore_missing_component_templates": [
          "logs@custom",
          "logs-apm.app@custom",
          "logs-apm.app-fallback@ilm"
        ]
      }
    }
  ]
}
```

#### b. Update the Index Template

From the output above, copy the entire `index_template` section and modify the `composed_of` array to replace the existing lifecycle component template with the desired retention policy. In this example, we replace `apm-10d@lifecycle` with `apm-90d@lifecycle` for 90-day retention:

```json
# Here logs-apm.app@template is the name of the index template
PUT _index_template/logs-apm.app@template 
{
  "index_patterns": [
    "logs-apm.app.*-*"
  ],
  "template": {
    "settings": {
      "index": {
        "mode": "standard",
        "default_pipeline": "logs-apm.app@default-pipeline",
        "final_pipeline": "logs-apm@pipeline"
      }
    }
  },
  "composed_of": [
    "logs@mappings",
    "apm@mappings",
    "apm@settings",
    "logs-apm@settings",
    "logs-apm.app-fallback@ilm",
    "ecs@mappings",
    "logs@custom",
    "logs-apm.app@custom",
    "apm-90d@lifecycle"
  ],
  "priority": 210,
  "version": 101,
  "_meta": {
    "managed": true,
    "description": "Index template for logs-apm.app.*-*"
  },
  "data_stream": {
    "hidden": false,
    "allow_custom_routing": false
  },
  "allow_auto_create": true,
  "ignore_missing_component_templates": [
    "logs@custom",
    "logs-apm.app@custom",
    "logs-apm.app-fallback@ilm"
  ]
}
```

**Sample Output:**

```json
{
  "acknowledged": true
}
```

#### c. Repeat for Other Templates

Repeat the above steps for `metrics-apm.app@template` and `traces-apm@template`, updating each with the appropriate lifecycle component template based on your retention requirements.

> [!IMPORTANT]
> Changes to index templates only affect **new data streams** created after the update. Existing data streams will continue using their original retention policies until they are manually updated or recreated.

### Step 3: Delete Existing Data Streams (Setup Time Only)

> [!WARNING]
> This step should only be performed once during initial setup. Deleting data streams will permanently remove all data and indices under those data streams.

After updating the index templates with new retention policies, you need to delete the existing data streams so they can be recreated with the updated retention settings. Use the Dev Tools Console in Kibana to run the following commands:

**Delete Logs Data Stream:**

```json
DELETE _data_stream/logs-apm.app* 
```

**Delete Metrics Data Stream:**

```json
DELETE _data_stream/metrics-apm.app* 
```

**Delete Traces Data Stream:**

```json
DELETE _data_stream/traces-apm* 
```

**Sample Output for each command:**

```json
{
  "acknowledged": true
}
```

> [!NOTE]
> After deleting the data streams, new data streams will be automatically created with the updated retention policies when APM agents begin sending new telemetry data.

---

## Advanced Configuration

For more advanced retention management using Index Lifecycle Management (ILM) policies with customizable phases (hot, warm, cold, delete), refer to the official Elasticsearch documentation:

- [Index Lifecycle Management (ILM) Overview](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)
- [Configure ILM Policies](https://www.elastic.co/guide/en/elasticsearch/reference/current/set-up-lifecycle-policy.html)
- [Data Stream Lifecycle vs ILM](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-stream-lifecycle.html)

ILM provides more granular control over data lifecycle phases and allows for tiered storage architectures in large-scale environments.