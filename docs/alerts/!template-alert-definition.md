# Alert Title/Name

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

