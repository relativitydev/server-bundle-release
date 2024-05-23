# Resource Server Inactive

## Description: 
Resource Server Inactive<br/>


## Alert Details:
**Alert ID:** 1b3b3a99-5c2e-4b5d-8ee6-a6d8b77de3d9

**Tags:**
Each tag should follow "key:value" format.

|Tag Name|Tag Key|Tag Value|Description|
|--|--|--|--|
|Alert Type|Type: |Platform|
|Alert Group|Group: |Server Health|Agents|
|Alert Navigation Type|PageType|RelativityTab|
|Alert Navigation param|PageID:||
|Created By|CreatedBy|Relativity|
|Resolution Text|||Future sprint|
|Resolution URL|ResolutionURL||[Link](00002-Resource-server-inactive-alert-resolution-sop.md)|

## Metric/Log/Trace Details:
**Metric Name:** relsvr.server.inactive

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.application_name||Environment Administration & Operations|
|labels.message|Message describes the issue|Server (ArtifactID:[Artifact ID]) [emttest] is inactive.|
|labels.name|Name of metric|Server Inactive|
|labels.relsvr_artifact_id|Relativity server artifact Id||
|labels.relsvr_subsystem|Resource server||
|labels.relsvr_system|System name||

## Rule details
**Alert Condition Description:** Alert Triggers when resource server is inactive <br/><br/>

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query|relsvr.server.inactive : 1|
|Data View| metrics-*||
|Filter Query|relsvr.server.inactive : 1|
Group| Count|greater than 0|
|Threshold| > 0| Count greater than 0, alert triggers|
|Time Window| 5 min| Verified data for last 5 minutes|
|Frequency| 1 min|Checks for each 1 min|

## Visualization link
Relativity link to Server Tab

