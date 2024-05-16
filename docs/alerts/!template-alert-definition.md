# Alert Title/Name

## Description: 
Brief description of alert.

## Alert Details:
**Alert ID:** GUID

**Tags:**
Each tag should follow "key:value" format.

|Tag Name|Tag Key|Tag Value|Description|
|--|--|--|--|
|Alert Type|Type|Platform/Infrastructure/Application|
|Alert Group|Group||
|Alert Navigation Type|PageType|Dashboard/SavedSearch/RelativityTab|
|Alert Navigation param|PageID|GUID of Kibana object(Dashboard/SavedSearch)|
|Created By|CreatedBy|Relativity| 
|...|...|...|

## Metric/Log/Trace Details:
**Metric Name:**

**Metric Attributes:**

|Attribute Name| Description|
|-------|---
|...||

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

## Kibana Visualization (If applicable)
Breif description on visualization to verify

