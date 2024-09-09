# Alert Title/Name
Brief alert title.
- Do **NOT** use special characters (e.g. %,$)
- Do **NOT** use title casing but DO capitilize official Relativity Object/Object Types
- Do **NOT** include threshold values in the markdown filename or title but **DO** include the threshold values within appropriate sections below   
  - Alert Title: Disk IO is exceeding 90% on at least one host --> Disk IO is exceeding threshold on at least one host
  - Filename: Disk-io-is-exceeding-90%-on-atleast-one-host-alert-definition.md --> Disk-io-is-exceeding-threshold-on-atleast-one-host-alert-definition.md

## Description: 
Brief description of alert.

## Alert Details:
**Alert ID:** GUID

**Tags:**
Each tag should follow "key:value" format.

- Type:Platform/Infrastructure/Application
- Group:[Group Name]
- PageType:Dashboard/SavedSearch/RelativityTab
- PageID:[GUID of Kibana object(Dashboard/SavedSearch)]
- CreatedBy:Relativity
- ResolutionURL:[relative path to alert-resolution-sop.md]

## Metric/Log/Trace Details:
**Metric Name:**

**Metric Attributes:**

|Attribute Name| Description| Value|
|-------|---|--|
|...|||

## Rule details
**Alert Condition Description:** <br/><br/>

|Name|Value|Description|
|-|-|-|
|Rule Type| Metric/Log/Latency Threshold/  Elastic Query/...|
|Data View|- logs-* for Relativity Logging or the new health check API<br/>- apm-* for Traces<br/>- metrics-* for Meters
|Filter Query||
Group| Count/sum/average|
|Threshold| Threshold value to trigger alert|
|Time Window| Last n sec/min/hour/day data to be considered|
|Frequency| How frequent it should check for alert condition|

## Requires User Intervention
- Yes: alert immediately
  - Is the metric based on expiration?
    - Min time before the alert is active/inactive: 30 minutes
  - Otherwise:
    - Min time before the alert is active/inactive: 90 seconds
- No: define time window before the alert fires
  - Min time before the alert is active/inactive: 3-10 minutes

## Visualization link
(e.g. a Kibana link to a dashboard, a Relativity link to a tab, etc.)

## Related Alerts
List down any dependent alerts to be fired before this alert
List down any related alerts

