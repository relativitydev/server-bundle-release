# InfraWatch monitoring agent is offline for at least one host

## Description: 
Alert will be true when InfraWatch monitoring agent is offline for at least one host.

## Alert Details:
**Alert ID:** a92e11a2-ebd3-4316-9235-0cef8ba7ca1b

**Tags:**
Each tag should follow "key:value" format.

- Type:Infrastructure
- Group:Server Health
- PageType:Dashboard
- PageID:
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00013-InfraWatch-monitoring-agent-is-offline-for-at-least-one-host-alert-resolution-sop.md

## Metric Details:
**Metric Name:** numeric_labels.heartbeat_count

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.application_name|Name of the application||
|host.name|emttest||
|labels.relsvr_system|Agents||
|labels.relsvr_subsystem|Custom Agent||
|labels.server_address|emttest||

## Rule details
**Alert Condition Description:** Alert triggers on when InfraWatch monitoring agent is offline for at least one host for at least 2 minutes.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| [Relativity] Hosts Heartbeat||
|Filter Query|numeric_labels.heartbeat_count> 0||
|Group| Count|number of host names|
|Threshold| < 0| Count lesser than 0, alert triggers|
|Time Window| 2 min| Verified data for last 2 minutes|
|Frequency| 1 min|Checks for every 1 min|

## Requires User Intervention
- Yes: alert immediately
  - Min time before the alert is active/inactive: 90 seconds

## Visualization link
Kibana Dashboard link

## Related Alerts
Windows Service stopped alert

