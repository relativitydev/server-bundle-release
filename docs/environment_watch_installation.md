# Environment Watch and Data Grid Audit Installation


## Installation Overview

Environment Watch and Data Grid Audit require installation and configuration of third-party and Relativity software. This installation guide covers the full set up for Environment Watch but only the Elasticsearch and Kibana set up for Data Grid Audit. Additional steps for configuring the Audit application within Relativity are covered [here](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit).

The Relativity applications and components that are referenced in this installation guide are packaged together in the Server bundle release. You can find the latest bundle on GitHub [here](https://github.com/relativitydev/server-bundle-release/releases). Environment Watch and Data Grid Audit also require Relativity applications that are available in the Relativity Application Library and not packaged in the bundle or covered in this installation guide (e.g. Pagebase and Telemetry for Environment Watch, Audit for Data Grid Audit, and InfraWatch Services for both). These applications are identified as pre-requisites in relevant sections of this installation guide.

For information about Environment Watch's performance impact on Relativity workloads, see [Environment Watch Performance Impact](./environment_watch_performance_impact.md).

The Server bundle is generally released quarterly, with hotfixes provided for critical issues as needed.

Environment Watch installation is comprised of the following seven steps. **Steps 1 and 2 are also used to set up Elasticsearch and Kibana for Data Grid Audit. Steps 3-7 are only relevant for Environment Watch.**

![alt text](../resources/stage_environmentwatch01.png)

## Next Step
[Click here for the next step](elasticsearch_pre_installation_overview.md)