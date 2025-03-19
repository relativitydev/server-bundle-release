# Relativity Alerts Troubleshooting

This guide will help you resolve common issues that may arise during the installation of Alert application. If you encounter difficulties, follow the steps outlined below to identify and fix the problem quickly. Before proceeding, ensure that your device meets the system requirements and that you have a stable internet connection.

* [During Installation](#during-installation)

* [After Installation](#after-installation)

### During Installation

### Issue 1: "Relativity Alert object data is unavailable or stale because Environment Watch is not currently enabled in this instance. Please see {doc page} for instructions on how to deploy and enable Environment Watch." error is displayed.
![ew-toggle-disabled-exception.png](../resources/relativity-alerts-troubleshooting-images/ew-toggle-disabled-exception.png)

#### This error occurs for the following reason:
    1. Relativity Server CLI First-Time Setup not executed properly.

#### Steps to troubleshoot:
    1. Please execute the Relativity Server CLI first time setup again to resolve this issue.

### Issue 2: "This operation cannot be completed because an HTTP request to the Elasticsearch server failed. Please try again. If the issue persists, ensure that the Elasticsearch server is accessible and the configuration is correct." error is displayed.
![elastic-setup-not-started-error.png](../resources/relativity-alerts-troubleshooting-images/elastic-kibana-not-started-error.png)

#### This error occurs for the following reasons:
    1. Missing Setup of Elastic/Kibana/APM Servers.
    2. Relativity Server CLI First-Time Setup not executed properly.

#### Steps to troubleshoot:
    1. Please verify Elastic/Kibana/APM Servers are up and running.
    2. You might need to execute the Relativity Server CLI first time setup again to resolve this issue.

### After Installation

### Issue 3: "Agent information doesn't exist." error is displayed.
![alert-agent-doesnot-exist1.png](../resources/relativity-alerts-troubleshooting-images/alert-agent-doesnot-exist1.png)
![alert-agent-doesnot-exist2.png](../resources/relativity-alerts-troubleshooting-images/alert-agent-doesnot-exist2.png)

#### Steps to troubleshoot:
    You might be missing the "Alert Manager" agent. Please follow below steps to verify and create a new agent if needed:
    Step 1: Verify the "Alert Manager" Agent
            a. Navigate to the Agents tab in Relativity.
            b. Filter the Name field to check if the 'Alert Manager' agent is present..
            c. If the agent is missing, proceed to create a new one by following the step2 below.
    Step 2: Create a New "Alert Manager" agent
            a. Navigate to the Agents tab in relativity.
            b. Click on the "New Agent" button.
            c. In the Agent Type dropdown, choose "Alert Manager".
            d. Enter 30 as the Run Interval (in seconds).
            e. Choose an active Agent Server that is assigned to one of the existing Resource Pools.
            f. Click on Save
![alert-agent-type.png](../resources/relativity-alerts-troubleshooting-images/alert-agent-type.png)
![testvm-hopper-instance.png](../resources/relativity-alerts-troubleshooting-images/testvm-hopper-instance.png)
![verifyfields.png](../resources/relativity-alerts-troubleshooting-images/verifyfields.png)
    Once these steps are completed, the "Alert Manager" agent should now be active and functioning correctly.

### Issue 4: "The Alert Manager agent is unable to retrieve current alert state information from Kibana. Please ensure that the Alert Manager agent is enabled. To see accurate alert state information please navigate to Kibana." error is displayed.
![alert-agent-disabled1.png](../resources/relativity-alerts-troubleshooting-images/alert-agent-disabled1.png)
![alert-agent-disabled2.png](../resources/relativity-alerts-troubleshooting-images/alert-agent-disabled2.png)

#### Steps to troubleshoot:
    Your "Alert Manager" agent might be disabled. Please follow below steps to verify and enable the agent if needed:
    Step 1: Verify the "Alert Manager" Agent
            a. Navigate to the Agents tab in Relativity.
            b. Filter the Name field to check 'Alert Manager' agent is present.
            c. CLick on 'Alert Manager' agent and check if the agent is disabled, if so, proceed to enable the agent by following the step2 below.
    Step 2: Enable the 'Alert Manager' agent
            a. Navigate to the Agents tab in Relativity.
            b. Filter the Name field to check 'Alert Manager' agent is present.
            b. Click on 'Alert Manager' agent and click on 'Enable Agent'
![alert-agent-enabled.png](../resources/relativity-alerts-troubleshooting-images/alert-agent-enabled.png)
    Once these steps are completed, the "Alert Manager" agent should now be Enabled and functioning correctly.

### Issue 5: "The Alert Manager agent interval is currently set to more than 15 minutes. If you want alert state information to be retrieved from Kibana more frequently, please update the agent interval." warning is displayed.
![alert-agent-interval-exceeds1.png](../resources/relativity-alerts-troubleshooting-images/alert-agent-interval-exceeds1.png)
![alert-agent-interval-exceeds2.png](../resources/relativity-alerts-troubleshooting-images/alert-agent-interval-exceeds2.png)

#### Steps to troubleshoot:
    Your "Alert Manager" agent might have exceeded the interval. Please follow below steps to verify and to update interval value if needed:
    Step 1: Verify the "Alert Manager" Agent
            a. Navigate to the Agents tab in Relativity.
            b. Filter the Name field to check 'Alert Manager' agent is present.
            c. CLick on 'Alert Manager' agent and check if the agent interval exceeds 900 seconds (15 minutes), if so, proceed to update agent interval value by following the step2 below.
    Step 2: Update Agent interval to a value less then 900 seconds (15 minutes)
            a. Navigate to the Agents tab in Relativity.
            b. Filter the Name field to check 'Alert Manager' agent is present.
            b. Click on 'Alert Manager' agent and Update Agent interval to a value less then 900 seconds (15 minutes). Recomended value is 30 (default).
![verify-alert-agent-interval.png](../resources/relativity-alerts-troubleshooting-images/verify-alert-agent-interval.png)

### Issue 6: "Unable to retrieve alert state information from Kibana. Please ensure that Elasticsearch and Kibana are functional." error is displayed.
![elastic-kibana-down1.png](../resources/relativity-alerts-troubleshooting-images/elastic-kibana-down1.png)
![elastic-kibana-down2.png](../resources/relativity-alerts-troubleshooting-images/elastic-kibana-down2.png)

#### This error occurs for the following reasons:
    1. Missing Setup of Elastic/Kibana/APM Servers.
    2. Relativity Server CLI First-Time Setup not executed properly.

#### Steps to troubleshoot:
    1. Ensure that the ElasticSearch, Kibana, and APM Servers are up and running. Verify that these services are properly started and accessible.
    2. Re-run the Relativity Server CLI first-time setup. It may be necessary to execute the setup again to resolve any configuration issues or missing components related to the integration.

### Issue 7: "The user does not have permission to perform this action." error is displayed.
![admin-permission-error.png](../resources/relativity-alerts-troubleshooting-images/admin-permission-error.png)

#### This error occurs for the following reason:
    1. User doesn't have System Administrator permissions.

#### Steps to troubleshoot:
    1. Please verify that user has the System Administrator permissions to perform the required actions.

### Issue 8: "Relativity Alert object data is unavailable or stale because Environment Watch is not currently enabled in this instance. Please see {doc page} for instructions on how to deploy and enable Environment Watch" error is displayed.
![ew-toggle-disabled-error.png](../resources/relativity-alerts-troubleshooting-images/ew-toggle-disabled-error.png)

#### This error occurs for the following reason:
    1. Relativity Server CLI First-Time Setup not executed properly.

#### Steps to troubleshoot:
    1. Please execute the Relativity Server CLI first time setup again to resolve this issue.
