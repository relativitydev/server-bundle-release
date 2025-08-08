# Environment Watch and Data Grid Audit Installation



## Installation Overview

Environment Watch and Data Grid Audit require installation and configuration of third-party and Relativity software. This installation guide covers the full set up for Environment Watch but only the Elasticsearch and Kibana set up for Data Grid Audit. Additional steps for configuring the Audit application within Relativity are covered [here](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit).

The Relativity applications and components that are referenced in this installation guide are packaged together in the Server bundle release. You can find the latest bundle on GitHub [here](https://github.com/relativitydev/server-bundle-release/releases). Environment Watch and Data Grid Audit also require Relativity applications that are available in the Relativity Application Library and not packaged in the bundle or covered in this installation guide (e.g. Pagebase and Telemetry for Environment Watch, Audit for Data Grid Audit, and InfraWatch Services for both). These applications are identified as pre-requisites in relevant sections of this installation guide.

The Server bundle is generally released quarterly but hot fixes are provided for critical issues on an ad hoc basis.

Environment Watch installation is comprised of the following four stages. **Stages 1 and 2 are also used to set up Elasticsearch and Kibana for using Data Grid Audit. Stages 3 and 4 are only relevant for Environment Watch.**

![alt text](../resources/stage_environmentwatch01.png)

<table><tbody>
<tr>
  <th><p><strong>Stage</strong></p></th>
  <th><p><strong>Name</strong></p></th>
  <th><p><strong>Applicable For</strong></p></th>
  <th><p><strong>Description</strong></p></th>
  <th><p><strong>Environment Watch Bundle Assets</strong></p></th>
</tr>
<tr><td><p>1</p></td><td><p>Install Elasticsearch, Kibana, and APM Server</p></td><td><p>Environment Watch and Data Grid Audit</p></td><td><p>Install the Elastic stack software required for Environment Watch and/or Data Grid Audit (Elasticsearch, Kibana, and APM Server).</p><div class="note">APM Server is only used for Environment Watch.</div></td><td><ul><li>N/A</li></ul></td></tr>
<tr><td><p>2</p></td><td><p>Configure Elastic Stack and Generate API Keys</p></td><td><p>Environment Watch and Data Grid Audit</p></td><td><p>Configure Elasticsearch, Kibana, and APM Server, including security, monitoring, and API key generation for integration.</p></td><td><ul><li>N/A</li></ul></td></tr>
<tr><td><p>3</p></td><td><p>Install and Configure Relativity Server CLI</p></td><td><p>Environment Watch and Data Grid Audit</p></td><td><p>Install and use the Relativity Server CLI to set up integration between Elastic and Relativity, and import Kibana objects for Environment Watch.</p></td><td><ul><li>Relativity.Server.CLI</li></ul></td></tr>
<tr><td><p>4</p></td><td><p>Install Other Integrations</p></td><td><p>Environment Watch only</p></td><td><p>Install and configure other required integrations and supporting services for Environment Watch.</p></td><td><ul><li>Relativity.EnvironmentWatch.Installer</li></ul></td></tr>
<tr><td><p>5</p></td><td><p>Install Environment Watch on All Hosts</p></td><td><p>Environment Watch only</p></td><td><p>Install Environment Watch on all hosts to be monitored in your Relativity Server environment.</p></td><td><ul><li>Rel-infrawatch-agent</li></ul></td></tr>
<tr><td><p>6</p></td><td><p>Install/Upgrade Relativity Alerts Application</p></td><td><p>Environment Watch only</p></td><td><p>Install or upgrade the Relativity Alerts application (RAP) for in-app alert notifications within Relativity.</p></td><td><ul><li>Relativity.Alerts.VERSION.rap</li></ul></td></tr>
<tr><td><p>7</p></td><td><p>Verify and Finalize Setup</p></td><td><p>Environment Watch and Data Grid Audit</p></td><td><p>Verify all components, integrations, and monitoring are working as expected. Finalize configuration and perform any required troubleshooting.</p></td><td><ul><li>N/A</li></ul></td></tr>
</tbody></table>

## Next 
Proceed to stage 1: [Installing Elasticsearch, Kibana, and APM Server](elasticsearch_pre_installation_overview.md)