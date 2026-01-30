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

## Configuration Steps

### Step 1: Create Component Template with Required Retention Policy

Elastic APM provides the `apm-90d@lifecycle` component template by default for 90-day retention. For 30-day retention (recommended for traces), create a custom component template using the Dev Tools Console in Kibana:

**Navigate to Dev Tools Console:**

1. Open Kibana in your web browser
2. Click on **Dev Tools** in the left navigation menu (or use the search bar at the top to find "Dev Tools")
3. You'll see the Console interface where you can execute Elasticsearch queries

**Sample Request:**

```
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

> [!IMPORTANT]
> Changes to index templates only affect **new data streams** created after the update. Existing data streams will continue using their original retention policies until they are manually updated or recreated.

#### a. Update Logs Index Template

First, use the Dev Tools Console in Kibana to retrieve the existing index template settings using a GET request:

**Sample Request:**

```
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

Then, copy the `index_template` section from the output above and update it by replacing `apm-10d@lifecycle` with `apm-90d@lifecycle` in the `composed_of` array using a PUT request:

**Sample Request:**

```
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

#### b. Update Metrics Index Template (Optional)

The `metrics-apm.app@template` already uses the `apm-90d@lifecycle` component template by default, so it does not require any updates if you are using the recommended 90-day retention period. If you need a different retention period, retrieve the current template configuration using a GET request:

**Sample Request:**

```
# Get the current template configuration
GET _index_template/metrics-apm.app@template
```

**Sample Output:**

```json
{
  "index_templates": [
    {
      "name": "metrics-apm.app@template",
      "index_template": {
        "index_patterns": [
          "metrics-apm.app.*-*"
        ],
        "template": {
          "settings": {
            "index": {
              "mode": "standard",
              "default_pipeline": "metrics-apm.app@default-pipeline",
              "final_pipeline": "metrics-apm@pipeline"
            }
          }
        },
        "composed_of": [
          "metrics@mappings",
          "apm@mappings",
          "apm@settings",
          "metrics-apm@settings",
          "metrics-apm.app-fallback@ilm",
          "ecs@mappings",
          "metrics@custom",
          "metrics-apm.app@custom",
          "apm-90d@lifecycle"
        ],
        "priority": 210,
        "version": 101,
        "_meta": {
          "managed": true,
          "description": "Index template for metrics-apm.app.*-*"
        },
        "data_stream": {
          "hidden": false,
          "allow_custom_routing": false
        },
        "allow_auto_create": true,
        "ignore_missing_component_templates": [
          "metrics@custom",
          "metrics-apm.app@custom",
          "metrics-apm.app-fallback@ilm"
        ]
      }
    }
  ]
}
```

Then, if you need to change the retention period, copy the `index_template` section from the output above and update it by replacing `apm-90d@lifecycle` with your desired retention component template in the `composed_of` array using a PUT request:

**Sample Request:**

```
PUT _index_template/metrics-apm.app@template
{
  "index_patterns": [
    "metrics-apm.app.*-*"
  ],
  "template": {
    "settings": {
      "index": {
        "mode": "standard",
        "default_pipeline": "metrics-apm.app@default-pipeline",
        "final_pipeline": "metrics-apm@pipeline"
      }
    }
  },
  "composed_of": [
    "metrics@mappings",
    "apm@mappings",
    "apm@settings",
    "metrics-apm@settings",
    "metrics-apm.app-fallback@ilm",
    "ecs@mappings",
    "metrics@custom",
    "metrics-apm.app@custom",
    "apm-90d@lifecycle"
  ],
  "priority": 210,
  "version": 101,
  "_meta": {
    "managed": true,
    "description": "Index template for metrics-apm.app.*-*"
  },
  "data_stream": {
    "hidden": false,
    "allow_custom_routing": false
  },
  "allow_auto_create": true,
  "ignore_missing_component_templates": [
    "metrics@custom",
    "metrics-apm.app@custom",
    "metrics-apm.app-fallback@ilm"
  ]
}
```

**Sample Output:**

```json
{
  "acknowledged": true
}
```

#### c. Update Traces Index Template

For traces, retrieve the current template configuration using a GET request:

**Sample Request:**

```
# Get the current template configuration
GET _index_template/traces-apm@template
```

**Sample Output:**

```json
{
  "index_templates": [
    {
      "name": "traces-apm@template",
      "index_template": {
        "index_patterns": [
          "traces-apm*"
        ],
        "template": {
          "settings": {
            "index": {
              "mode": "standard",
              "default_pipeline": "traces-apm@default-pipeline",
              "final_pipeline": "traces-apm@pipeline"
            }
          }
        },
        "composed_of": [
          "traces@mappings",
          "apm@mappings",
          "apm@settings",
          "traces-apm@settings",
          "traces-apm-fallback@ilm",
          "ecs@mappings",
          "traces@custom",
          "traces-apm@custom",
          "apm-10d@lifecycle"
        ],
        "priority": 210,
        "version": 101,
        "_meta": {
          "managed": true,
          "description": "Index template for traces-apm*"
        },
        "data_stream": {
          "hidden": false,
          "allow_custom_routing": false
        },
        "allow_auto_create": true,
        "ignore_missing_component_templates": [
          "traces@custom",
          "traces-apm@custom",
          "traces-apm-fallback@ilm"
        ]
      }
    }
  ]
}
```

Then, copy the `index_template` section from the output above and update it by replacing `apm-10d@lifecycle` with `apm-30d@lifecycle` (which you created in Step 1) in the `composed_of` array using a PUT request:

**Sample Request:**

```
PUT _index_template/traces-apm@template
{
  "index_patterns": [
    "traces-apm*"
  ],
  "template": {
    "settings": {
      "index": {
        "mode": "standard",
        "default_pipeline": "traces-apm@default-pipeline",
        "final_pipeline": "traces-apm@pipeline"
      }
    }
  },
  "composed_of": [
    "traces@mappings",
    "apm@mappings",
    "apm@settings",
    "traces-apm@settings",
    "traces-apm-fallback@ilm",
    "ecs@mappings",
    "traces@custom",
    "traces-apm@custom",
    "apm-30d@lifecycle"
  ],
  "priority": 210,
  "version": 101,
  "_meta": {
    "managed": true,
    "description": "Index template for traces-apm*"
  },
  "data_stream": {
    "hidden": false,
    "allow_custom_routing": false
  },
  "allow_auto_create": true,
  "ignore_missing_component_templates": [
    "traces@custom",
    "traces-apm@custom",
    "traces-apm-fallback@ilm"
  ]
}
```

**Sample Output:**

```json
{
  "acknowledged": true
}
```

### Step 3: Delete Existing Data Streams (Setup Time Only)

> [!CAUTION]
> **⚠️ DESTRUCTIVE OPERATION – PERMANENT DATA LOSS**
> 
> **This step is optional and is not required for most Environment Watch deployments.** It should only be performed during initial setup or in controlled, non-production scenarios.
> 
> This step will **permanently delete all data and indices** in the specified data streams. There is no recovery. Only proceed if:
> - You are in a **development or non-production environment**, OR
> - You have **backed up all critical data** from these data streams, OR
> - You are performing **initial setup** and no production data exists yet
> 
> **Do NOT run this on production systems with active data.**


After updating the index templates with new retention policies, you need to delete the existing data streams so they can be recreated with the updated retention settings. Use the Dev Tools Console in Kibana to run the following commands:

**Delete Logs Data Stream:**

```
DELETE _data_stream/logs-apm.app* 
```

**Delete Metrics Data Stream:**

```
DELETE _data_stream/metrics-apm.app* 
```

**Delete Traces Data Stream:**

```
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