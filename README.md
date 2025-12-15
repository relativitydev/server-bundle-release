
# Server Bundle Release

This release bundle contains packages that are required to set up or upgrade two optional Relativity Server features:

- **Environment Watch**: [Click here to learn more about Environment Watch](docs/environment_watch_product_overview.md)
  
- **Data Grid Audit**: [Click here to learn more about Data Grid Audit and setup instructions](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit)



## System Requirements

| **Requirement**   | **Relativity Server 2024 Patch 2**                                  | **Relativity Server 2025**                                  |
| :---------------- | :------------------------------------------------------------------ | :---------------------------------------------------------- |
| Elastic Stack     | 8.17.3 is required for Environment Watch.<br/>7.17.x can be used but for Data Grid Audit only. | 8.17.3 and 9.1.3 versions are officially certified for both Data Grid Audit and Environment Watch. |

**Network Ports**: Ensure required ports are open for Elastic Stack communication. [Refer to the port diagram](docs/environment-watch/port-diagram.md) for full network configuration details.

> **Note**: Relativity Server includes visualization of the Environment Watch Catalog for easier monitoring and management. For more details, refer to the Environment Watch Catalog documentation for your Server version:
> - [Environment Watch Catalog - Relativity Server 2024 Patch 2](https://github.com/relativityone/server-relativity-docs/blob/release-24.0-2024/environment-watch/environment-watch-catalog/readme.md)
> - [Environment Watch Catalog - Relativity Server 2025](https://github.com/relativityone/server-relativity-docs/blob/release-25.0-2025/environment-watch/environment-watch-catalog/readme.md)


## Getting Started

- **First-Time Installation**: [Click here for installation instructions](docs/environment_watch_installation.md).

- **Upgrade**: [Click here for upgrade instructions](docs/environment_watch_upgrade.md).

- **Troubleshooting**: [Click here for the troubleshooting guide](docs/environment_watch_troubleshooting.md).
