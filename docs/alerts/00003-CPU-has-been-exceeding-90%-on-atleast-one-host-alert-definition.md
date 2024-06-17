# CPU has been exceeding 90% on at least one host

## Description: 
Alert will be true if CPU has exceeded 90% for at least 15 minutes on at least one host.

## Alert Details:
**Alert ID:** bc26a7d4-b4c9-419a-9b39-805231f7621d

**Tags:**
Each tag should follow "key:value" format.

- Type:Infrastructure
- Group:System Health
- PageType:Dashboard
- PageID:ee8a7dab-7422-499e-94e0-7b5449468a94
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00003-CPU-has-been-exceeding-90%-on-atleast-one-host-alert-resolution-sop.md

## Metric Details:
**Metric Name:** system.cpu.utilization

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.cpu|Logical CPU number starting at 0.||
|labels.state|Breakdown of CPU usage by type.||

## Rule details
**Alert Condition Description:** Alert triggers on CPU has exceeded 90% for at least 15 minutes on at least one host.

|Name|Value|Description|
|-|-|-|
|Rule Type| Metric threshold||
|Group| Average| Average of CPU utilization per host|
|Threshold| > 0.9| Average > 0.9, alert triggers|
|Time Window| 15 min| Verified data for last 15 minutes|
|Frequency| 1 min|Checks for each 1 minute|
|Group alerts by| host.name| Group by host|

## Requires User Intervention
- No
  - Min time before the alert is active: 15 minutes
  - Min time before the alert is inactive: 2 - 5 minutes

## Visualization link
Kibana dashboard link

## Related Alerts
Host Heartbeat alert should not be in active state.