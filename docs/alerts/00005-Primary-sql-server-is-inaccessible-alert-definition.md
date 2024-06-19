# Primary SQL Server is inaccessible

## Description: 
Alert is true if primary SQL Server is inaccessible.

## Alert Details:
**Alert ID:** 27280455-f5fa-4d8a-b051-77e11621fe1e

**Tags:**
Each tag should follow "key:value" format.

- Type:Infrastructure
- Group:Access
- PageType:
- PageID:
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00005-Primary-sql-server-is-inaccessible-alert-resolution-sop.md

## Metric/Log/Trace Details:
**Metric Name:** relsvr.sqlserver.running

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.name|Name of Database|EDDS|
|labels.state||accessible/inaccessible|

## Rule details
**Alert Condition Description:** Alert triggers on primary SQL Server is inaccessible for last 90 seconds.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.sqlserver.running : 0|Primary SQL Server is inaccessible|
|Group| Count|number of Primary SQL Server is inaccessible documents|
|Threshold| > 0| Count greter than 0, alert triggers|
|Time Window| 90 sec| Verified data for last 90 sec|
|Frequency| 30 sec|Checks for each 30 seconds|

## Requires User Intervention
- Yes: alert immediately
  - Min time before the alert is active/inactive: 90 seconds

## Visualization link
No link

## Related Alerts
Host Heartbeat alert should not be in active state.
