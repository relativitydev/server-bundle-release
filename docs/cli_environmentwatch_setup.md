# CLI EnvironmentWatch setup

## Summary
This guide provides step-by-step instructions for setting up the Environment Watch using the CLI. Environment Watch enhances the Relativity Server product by modernizing observability features using Elasticsearch and Open Telemetry technology. It covers the prerequisites, setup process, verification steps to ensure a successful configuration. 

## Prerequisites
- The telemetry backend software ([Elasticsearch, Kibana, and APM Server](/docs/elasticsearch_setup.md)) is installed
- Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2024/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting
- SQL Primary and Distributed access
- Relativity access
- The minimum Relativity major version and patch is installed on all servers in the environment. See the [release bundle](https://github.com/relativityone/server-environment-watch-releases/releases) requirements for the minimum version required.
- [Elasticsearch certificates](/docs/elasticsearch_certificates_setup.md) must be installed
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
  ./relsvr.exe setup
  
### 4: Select Setup Option
- When prompted, select "Environment Watch" for setup and confirm the Environment Watch setup.
  ![](/resources/EnvironmentWatch.png)

### 5: Handle Existing Setup
- If the setup is not present on the machine, it will proceed directly. Otherwise, it will ask whether to Choose "Upgrade" or "Rerun Setup" or "Exit" to continue.  
  ![](/resources/EWRerun.png)

### 6: Provide Relativity Inputs
- Ensure to provide the necessary inputs for Relativity:
- Relativity admin username
- Relativity admin password
- Relativity URL   
  ![](/resources/EWRelativityInput.png)

### 7: Provide Elasticsearch Inputs
- Ensure to provide the necessary inputs for Elasticsearch:
- Elasticsearch admin username
- Elasticsearch admin password
- Elasticsearch primary server URL   
  ![](/resources/EWElasticInput.png)

### 8: Provide Elasticsearch APM Input
- Ensure to provide the necessary inputs for Elasticsearch APM:
- Elasticsearch APM server URL   
  ![](/resources/EWAPMInput.png)

### 9: Provide Elasticsearch Kibana Input
- Ensure to provide the necessary inputs for Elasticsearch Kibana:
- Elasticsearch Kibana server URL   
  ![](/resources/EWKibanaInput.png)

### 10: Completion Confirmation
- Once the Environment watch setup is completed, refer to the following image for more information:   
  ![](/resources/EWCompletion.png)

### 11: Retry on Failure
- If incorrect credentials are provided, retry the Environment watch setup up to a maximum of 3 attempts. For more information, refer to the following image:   
  ![](/resources/EWRelativityRetryAttempt.png)
  ![](/resources/EWElasticRetryAttempt.png)
  ![](/resources/EWAPMRetryAttempt.png)
  ![](/resources/EWKibanaRetryAttempt.png)

### 12: Maximum Retry Limit
- If all 3 attempts fail, a message indicating the maximum retry limit reached will be displayed. For more information, refer to the following image:     
  ![](/resources/EWRelativityMaxAttempts.png)
  ![](/resources/EWElasticMaxAttempts.png)
  ![](/resources/EWAPMMaxAttempts.png)
  ![](/resources/EWKibanaMaxAttempts.png)

### 13: Rerun Setup
- If the machine is already set up with Environment Watch and you want to rerun the setup, select the "Rerun Setup" option to reset the Environment Watch. For more information, refer to the images:    
  ![](/resources/EWRerun.png)
  ![](/resources/EWCompletion.png)

### 14: Upgrade Setup
- If the machine is already set up with Environment Watch and you want to upgrade the setup, select the "Upgrade" option to reset the Environment Watch. For more information, refer to the images:   
  ![](/resources/EWUpgrade.png)
  ![](/resources/EWUpgradeCompletion.png)
  
## Verification

### 1. Check the Kibana User Interface

#### a. Verify the Generated API Key for Environment Watch
1. Navigate to the path: `/app/management/security/api_keys`
2. Ensure that the generated API key for Alerts and Infrawatch are visible.
3. For more information, refer to the image below:
   ![](/resources/EWVerification.png)

## Troubleshooting
- If any errors are encountered during the CLI setup process, please refer to the [Troubleshooting Guide](/docs/troubleshooting.md) for resolution.

## Next steps
Click [here](/docs/environmentwatch_installer.md) to Install Relativity Environment Watch installation
