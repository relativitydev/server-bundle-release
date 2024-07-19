# Memory is exceeding 96% on at least one host

## Description: 
Alert will be true when RAM is exceeding 96% on at least one host for at least 5 consecutive minutes.

## Alert Details:
**Alert ID:** 234989dd-7c31-4980-a9ec-c11fea54c2d1

**Tags:**
Each tag should follow "key:value" format.

- Type:Infrastructure
- Group:System Health
- PageType:Dashboard
- PageID:ee8a7dab-7422-499e-94e0-7b5449468a94
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00014-Memory-is-exceeding-96%-on-at-least-one-host-alert-resolution-sop.md

## Metric Details:
**Metric Name:** system.cpu.utilization

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.state|Breakdown of memory usage by type.||

## Rule details
**Alert Condition Description:** Alert triggers when RAM is exceeding 96% on at least one host for at least 5 consecutive minutes.

|Name|Value|Description|
|-|-|-|
|Rule Type| Metric threshold||
|Group| Average| Average of memory utilization per host|
|Filter Query|labels.state : "used"|Memory used|
|Threshold| > 0.96| Average > 0.96, alert triggers|
|Time Window| 5 min| Verified data for last 5 minutes|
|Frequency| 1 min|Checks for each 1 minute|
|Group alerts by| host.name| Group by host|

## Requires User Intervention
- No
  - Min time before the alert is active: 5 minutes
  - Min time before the alert is inactive: 1 - 2 minutes

## Visualization link
Kibana dashboard link

## Related Alerts
Host Heartbeat alert should not be in active state.