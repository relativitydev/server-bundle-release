
# Server Bundle Release

This release bundle contains packages that are required to set up or upgrade two optional Relativity Server features:

- **Environment Watch**: [Click here to learn more about Environment Watch](./elastic-stack-setup/environment_watch_product_overview.md)
  
- **Data Grid Audit**: [Click here to learn more about Data Grid Audit and setup instructions](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit)



## System Requirements

| **Requirement**   | **Relativity Server 2024 Patch 2**                                  | **Relativity Server 2025**                                  |
| :---------------- | :------------------------------------------------------------------ | :---------------------------------------------------------- |
| Elastic Stack     |v7.17 and above (within the v7 series) - Data Grid Audit only<br/> Server 2024 is compatible with Elasticsearch 8.x and 9.x. The following versions are officially certified for both Data Grid Audit and Environment Watch: <br/> v8.17.3, v9.1.3 <br/> <br/> <i>Note: Elasticsearch v7 reached end of vendor support on January 15, 2026. When planning an upgrade, Relativity recommends using the latest supported Elasticsearch version</i>| Server 2025 is compatible with Elasticsearch 8.x and 9.x. The following versions are officially certified for both Data Grid Audit and Environment Watch: <br/> v8.17.3, v9.1.3|

**Network Ports**: Ensure required ports are open for Elastic Stack communication. [Refer to the port diagram](./elastic-stack-setup/elastic-stack-port-diagram.md) for full network configuration details.


## Getting Started

- **First-Time Installation**: [Click here for installation instructions](./elastic-stack-setup/elastic-stack-setup-01-installation.md).

- **Upgrade**: [Click here for upgrade instructions](./elastic-stack-setup/elastic-stack-setup-02-environment-watch/environment_watch_upgrade.md).

- **Troubleshooting**: [Click here for the troubleshooting guide](./elastic-stack-setup/troubleshooting/environment_watch_troubleshooting.md).
