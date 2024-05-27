# One or more agents are disabled

## Description: 
One or more agents are disabled.<br/>
*Note: Excluding Billing Agents.*

## Alert Details:
**Alert ID:** dae7d8e9-c33d-470c-b36a-5b6e380e0a25

**Tags:**
Each tag should follow "key:value" format.

|Tag Name|Tag Key|Tag Value|Description|
|--|--|--|--|
|Alert Type|Type|Platform|
|Alert Group|Group|Agents|
|Alert Navigation Type|PageType|RelativityTab|
|Alert Navigation param|PageID||
|Created By|CreatedBy|Relativity|
|Resolution Text|ResoltionText||Future sprint|
|Resolution URL|ResolutionURL||[Link](00001-One-or-more-agents-are-disabled-alert-resolution-sop.md)|

## Metric/Log/Trace Details:
**Metric Name:** relsvr.agent.disabled

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.agent_name|Relativity Agent Name||
|labels.agent_type_name|Relativity Agent Type|
|labels.application_name||Environment Administration & Operations|
|labels.exception_message|Any exception message on Agent||
|labels.message|Message describes the issue|Agent [Agent Name placeholder] is disabled.|
|labels.name|Name of metric|Agent Disabled|
|labels.relsvr_artifact_id|Relativity agent artifact Id||
|labels.relsvr_subsystem|Agent Name||
|labels.relsvr_system|System name|Agents|

## Rule details
**Alert Condition Description:** Alert triggers on disabled alerts count greater than 0 for last 5 minutes. Excluding "Billing Agent" type. <br/><br/>

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.agent.disabled : 1 and NOT labels.agent_type_name : "Billing Agent" |Agent should be disabled and should not be "Billing Agent" type|
Group| Count|number of agent disabled|
|Threshold| > 0| Count greter than 0, alert triggers|
|Time Window| 5 min| Verified data for last 5 minutes|
|Frequency| 1 min|Checks for each 1 min|

## Visualization link
Relativity link to Agents Tab

