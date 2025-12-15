
# Server Bundle Release

This release bundle contains packages that are required to set up or upgrade two optional Relativity Server features:

- **Environment Watch**: [Click here to learn more about Environment Watch](docs/environment_watch_product_overview.md)
  
- **Data Grid Audit**: [Click here to learn more about Data Grid Audit and setup instructions](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit)



## System Requirements

| **Requirement**   | **Version/Details (Relativity Server 2024 Patch 2)**      | **Version/Details (Relativity Server 2025)**   |
| :---------------- | :-------------------------------------------------------- | :--------------------------------------------- |
| Elastic Stack     | 8.17.3 is required for Environment Watch.<br/>7.17.x can be used but for Data Grid Audit only. | Compatible with Elastic Stack 8.17.3 and 9.1.3 for Environment Watch.<br/>9.1.3 is recommended. |

**Network Ports**: Ensure required ports are open for Elastic Stack communication. [Refer to the port diagram](docs/environment-watch/port-diagram.md) for full network configuration details.

---

> **Note**: Relativity Server 2025 and Relativity Server 2024 Patch 2 include visualization of the Environment Watch Catalog for easier monitoring and management. For more details, see the [Environment Watch Catalog](https://github.com/relativityone/server-relativity-docs/blob/main/environment-watch/environment-watch-catalog/readme.md).


## Getting Started

- **First-Time Installation**: [Click here for installation instructions](docs/environment_watch_installation.md).

- **Upgrade**: [Click here for upgrade instructions](docs/environment_watch_upgrade.md).

- **Troubleshooting**: [Click here for the troubleshooting guide](docs/environment_watch_troubleshooting.md).
