# Relativity Datagrid Audit CLI Setup

## Summary
This guide provides step-by-step instructions for setting up the DataGrid using the CLI. It covers the prerequisites, setup process, verification steps to ensure a successful configuration.

## Prerequisites
- The telemetry backend software ([Elasticsearch, Kibana, and APM Server](/docs/elasticsearch_setup.md)) is installed
- Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2024/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting
- SQL Primary and Distributed access
- Relativity access
- The minimum Relativity major version and patch is installed on all servers in the environment. See the [release bundle](https://github.com/relativityone/server-environment-watch-releases/releases) requirements for the minimum version required.
- [Elasticsearch certificates](/docs/elasticsearch_certificates_setup.md) must be installed
- Elasticsearch mapper-size Plugin should be installed
- The minimum supported version of Elasticsearch is 7.17
- Command Terminal

## Setup

## Downloading the CLI
Download the Relativity.Server.Cli.exe (Relativity.Server.Cli.24.0.xxxx.zip) from [here](https://github.com/relativityone/server-environment-watch-releases/releases).

## Steps to execute the Relativity Datagrid Setup CLI
### 1: Open Command Terminal
- Launch Command Terminal v7 from the Start menu.

### Step 2: Navigate to the Directory
- Navigate to the directory where the `Relativity.Server.Cli.24.0.xxxx.zip` is downloaded and extract the ZIP file.

### 3: Run the Setup Command
- Execute the following setup command to configure the DataGrid:
  ```bash
  relsvr.exe setup
  
### 4: Select Setup Option
- When prompted, select "Datagrid" for setup and confirm the Datagrid setup.
  ![](/resources/SelectedOptionDatagrid.png)

### 5: Handle Existing Setup
- If the setup is not present on the machine, it will proceed directly. Otherwise, it will ask whether to "RerunSetup" or "Exit" to continue.
  ![](/resources/RerunSetupOptionDatagrid.png)

### 6: Provide Elasticsearch Inputs
- Ensure to provide the necessary inputs for Elasticsearch:
  - Elasticsearch admin username
  - Elasticsearch admin password
  - Elasticsearch primary server URL

### 7: Completion Confirmation
- Once the Datagrid setup is completed, refer to the following image for more information:
  ![](/resources/FirstTimeSetup-Datagrid.png)

### 8: Retry on Failure
- If incorrect credentials are provided, retry the DataGrid setup up to a maximum of 3 attempts. For more information, refer to the following image:
  ![](/resources/RetryDatagridSetup.png)

### 9: Maximum Retry Limit
- If all 3 attempts fail, a message indicating the maximum retry limit reached will be displayed. For more information, refer to the following image:
  ![](/resources/DatagridSetupFailedWithAllRetry.png)

### 10: Rerun Setup
- If the machine is already set up with Datagrid and you want to rerun the setup, select the "RerunSetup" option to reset the Datagrid. For more information, refer to the images:
  ![](/resources/RerunSetupOptionDatagrid.png)
  ![](/resources/ReRunDatagrid.png)

## Verification

### 1. Check the Kibana User Interface

#### a. Verify the Generated API Key for DataGrid
1. Navigate to the path: `/app/management/security/api_keys`
2. Ensure that the generated API key for DataGrid is visible.
3. For more information, refer to the image below:
   ![](/resources/Datagrid-Api-key.png)

#### b. Verify Logs for ElasticAPIKeyAuthenticationToggle
1. Navigate to the path: `Analytics/discover/logs`
2. Search for "elastic" to filter and view the logs.
3. Ensure that the log entry for `ElasticAPIKeyAuthenticationToggle` is enabled.
4. For more information, refer to the image below:
   ![](/resources/ElasticAPIKeyAuthenticationToggle-log.png)

### 2. Verify the Audit Dashboard

1. Navigate to the `Audit` tab.
2. Verify that the Audit dashboard graphs and data are loaded in the Relativity Instance.
3. For more information, refer to the image below:
   ![](/resources/Audit-Dashboard.png)

## Troubleshooting
- If any errors are encountered during the CLI setup process, please refer to the [Troubleshooting Guide](/docs/troubleshooting.md) for resolution.

## Next steps
Click [here](/docs/environmentwatch_installer.md) to Install Relativity Environment Watch installation
