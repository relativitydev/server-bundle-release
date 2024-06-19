# Disk IO has been exceeding 90% on at least one host

## Description: 
Alert will be true if Disk IO has exceeded 90% for at least 15 minutes on at least one host.

## Alert Details:
**Alert ID:** 50ccd14c-8ec9-43d5-971d-14fc8b5dd700

**Tags:**
Each tag should follow "key:value" format.

- Type:Infrastructure
- Group:System Health
- PageType:Dashboard
- PageID:ee8a7dab-7422-499e-94e0-7b5449468a94
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00004-Disk-io-has-been-exceeding-90%-on-atleast-one-host-alert-resolution-sop.md

## Metric Details:
**Metric Name:** system.filesystem.utilization

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.device|Identifier of the filesystem.||
|labels.mode|Mountpoint mode such "ro", "rw", etc.||
|labels.mountpoint|Mountpoint path.||
|labels.type|Filesystem type, such as, "ext4", "tmpfs", etc.||

## Rule details
**Alert Condition Description:** Alert triggers on Disk IO has exceeded 90% for at least 15 minutes on at least one host.

|Name|Value|Description|
|-|-|-|
|Rule Type| Metric threshold||
|Group| Average| Average of Disk utilization per host|
|Threshold| > 0.9| Average > 0.9, alert triggers|
|Time Window| 15 min| Verified data for last 15 minutes|
|Frequency| 1 min|Checks for each 1 minute|
|Group alerts by| host.name| Group by host |

## Requires User Intervention
- No
  - Min time before the alert is active: 15 minutes
  - Min time before the alert is inactive: 2 - 5 minutes

## Visualization link
Kibana dashboard link

## Related Alerts
Host Heartbeat alert should not be in active state.

