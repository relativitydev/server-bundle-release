# Environment Watch Troubleshooting

## Elastic Search

### Issue 1: Insufficient Memory

- Running the .\elasticsearch.bat command occasionally triggers errors related to insufficient memory.

**Steps to troubleshoot**

- Disk Space: 

   Ensure that there is sufficient disk space available. Elasticsearch requires adequate disk space for indexing and storing data. 

### Issue 2: Elastic search service issues

**Steps to troubleshoot**

- Verify if Elasticsearch running

   \- Open powershell and use the command '**Get-Service -Name elasticsearch'** to check the status of the Elasticsearch service.

      > Get-Service -Name elasticsearch

      | Status  | Name         | DisplayName                         |

      |-------- |--------------|-------------------------------------|

      | Running | elasticsearch| Elasticsearch (Elasticsearch)|

- Verify if Elasticsearch accessible from a given host:

   -Check if elasticsearch is accessible by accessing https://emttest:9200/

   ![](../resources/troubleshooting-images/verifiyelasticsearchinbrowser.png)

   \- If the service is not running, elasticsearch can be started using

   '***.\\elasticsearch-service.bat start Elasticsearch'***

- Verify the SSL/TLS URL

  a. Check if the URL is Correct:

    - Verify the correct URL format for accessing Elasticsearch with SSL enabled. The URL should start with https:// if SSL is configured.

  b.Check Elasticsearch elasticsearch.yml Configuration:
    - Verify that SSL/TLS settings are correctly defined.

![alt text](../resources/troubleshooting-images/elasticsslconfig.png)

### Issue 3: SSL certificate issue

   ![alt text](../resources/troubleshooting-images/sslissue.png)

**Steps to troubleshoot**

- While logging into ElasticSearch URL, if it shows not secure. Export that certificate of the highest hierarchy and save to your local directory.
- Go to that directory where you saved the certificate. Double click on the certificate -> Click Install Certificate

   ![alt text](../resources/troubleshooting-images/installcertificate.png)

- Select Local Machine

   ![alt text](../resources/troubleshooting-images/localmachine.png)

- Select Next, Click yes
- Select Place all the certificates in the following store
- Click on Browse, select Enterprise Trust
- Select Next, It should get imported
- Open MMC, go to files and click on Add/Remove Snap-IN

   ![alt text](../resources/troubleshooting-images/Add-removesnipin.png)

- Add Certificates

   ![alt text](../resources/troubleshooting-images/addcerts.png)

- Click Computer Account

   ![alt text](../resources/troubleshooting-images/clickcomputeraccount.png)

- Click Next -> Finish
- On the left side bar click the dropdown under Certificates
- Right Click on Trusted Root Certification Authorirties, All Task → Import

   ![alt text](../resources/troubleshooting-images/alltask-import.png)

- Select the Certificate which being saved few steps earlier under browse and click on Finish
- Import certificate for all the selected folders below.

   ![alt text](../resources/troubleshooting-images/importcerts.png)

- Close all search engine and re-login to ElasticSearch URL

   ![alt text](../resources/troubleshooting-images/sslenabled.png)

### Issue 4: Issues while extracting the `kibana-8.xx.x-windows-x86_64.zip`

**Steps to troubleshoot**

- Windows must be updated to support Long Paths to enable the Local Group Policy Editor

   a. Run the "gpedit.msc" to navigate into Local Group Policy Editor

    ![alt text](../resources/troubleshooting-images/gpedit.png)

   b. Select Computer Configuration → Administrative Template → System → Filesystem. 

   ![alt text](../resources/troubleshooting-images/administrativetemplate.png)

   c. Double click on enable the Long path. 

   ![](../resources/troubleshooting-images/kibanaextract.png)

### Issue 5: Kibana service issues

**Steps to troubleshoot**

***Verify Kibana Service Status:***

- Run the service and verify if service is up and running 

   ![alt text](../resources/troubleshooting-images/kibanaservice.png)

- If Windows service fails to start is likely due to a configuration or environmental problem

  Verify kibana configuration by following [ElasticSearch setup](elasticsearch_setup.md)
- Check if Kibana is running by navigating to Kibana URL

### Issue 6: Kibana authentication issue

**Steps to troubleshoot Kibana authentication issue:**

- update your Kibana config file with below.
- Generate encryption key by executing below command ./kibana-encryption-keys generate
- In the kibana.yml configuration file, add the xpack.encryptedSavedObjects.encryptionKey setting.

    ![alt text](../resources/troubleshooting-images/kibanaencryption.png)

- Restart Kibana

### Issue 7: APM service issues   

**Steps to troubleshoot**

**Step 1: Verify APM Service Status:**

\- Check if apm server is running using powershell command **Get-Service

-Name apm-server**

> Get-Service -Name apm-server

| Status  | Name       | DisplayName  |
| ------- | ---------- | ------------ |
| Running | apm-server | {.BeatName \ |

\-- If the service is not running, start service using

'**Start-Service -Name apm-server\'*

a. Check if apm is running by navigating to APM URL http://emttest:8200/ in any supported Web browser.

![](../resources/troubleshooting-images/verifyapminbrowser.png)

If Windows service fails to start is likely due to a configuration or environmental problem

     Verify apm configuration by following [ElasticSearch setup](elasticsearch_setup.md)

b. Verify the "publish_ready" property is true/false.

   If the value of publish_ready is still false and elasticsearch is configured to use https, perform the below changes under "Elasticsearch output"

- Update protocol to https and uncomment the line
- Update ssl.enabled to true and uncomment the line
- Update ssl.verification_mode to none and uncomment the line
- Restart the apm service ({.BeatName | title})

### Relativity Server CLI - Environment Watch Setup

### Issue 1: Unauthorized issue:
  ![](../resources/EWRelativityUnauthorized.png)
  ![](../resources/EWElasticUnauthorized.png)
#### Steps to troubleshoot
- Verify your Relativity admin username and password, and provide valid credentials.
- Verify your Elasticsearch admin username and password, and provide valid credentials.

### Issue 2: Server URLs are incorrect:
  ![](../resources/EWRelativityUrlIncorrect.png)
  ![](../resources/EWElasticUrlIncorrect.png)
  ![](../resources/EWAPMUrlIncorrect.png)
  ![](../resources/EWKibanaUrlIncorrect.png)
#### Steps to troubleshoot
- Verify your Relativity/Elastic/APM/kibana URL, and provide valid URLs.

### Issue 3: ElasticSearch server credentials are incorrect

![](../resources/troubleshooting-images/invalidelasticcreds.png)

**Steps to troubleshoot**

The user should verify the Elasticsearch admin username and password and provide valid credentials

### Issue 4: Retry limit reached:
  ![](../resources/EWRelativityMaxAttempts.png)
  ![](../resources/EWElasticMaxAttempts.png)
  ![](../resources/EWAPMMaxAttempts.png)
  ![](../resources/EWKibanaMaxAttempts.png)
#### Steps to troubleshoot
- The user reached the maximum number of attempts. Please rerun the Relativity.Server.Cli with the setup command `relsvr.exe setup` using Command Terminal.

### Relativity Server CLI - DataGrid/Audit Setup

### Issue 1: Unauthorized issue:
  ![](../resources/Issue1-Unauthorized.png)
#### Steps to troubleshoot
- Verify your Elasticsearch admin username and password, and provide valid credentials.

### Issue 2: Elasticsearch server credentials are incorrect:
  ![](../resources/Issue2-ElasticUrlCredentialsWrong.png)
#### Steps to troubleshoot
- Verify your Elasticsearch admin username, password, and URL, and provide valid credentials.

### Issue 3: Retry limit reached:
  ![](../resources/Issue3-RetryLimit-Reached.png)
#### Steps to troubleshoot
- The user reached the maximum number of attempts. Please rerun the Relativity.Server.Cli with the setup command `relsvr.exe setup` using Command Terminal.

### Troubleshooting Environment Watch installer on Windows

### Issue 1: Product cannot be installed because user is not added in Local security policy

![](../resources/troubleshooting-images//user-not-added-in-local-security.png)

**Steps to troubleshoot**

**Step 1:** Add user to Local security policy.

![](../resources/troubleshooting-images/useraddedtolocalsecurity.png)

### Issue 2: Product cannot be installed because relativity secret store is not accessible.

![](../resources/troubleshooting-images/installer-secret-store-error1.png)

![](../resources/troubleshooting-images/installer-secret-store-error2.png)

**Steps to troubleshoot**

**Step 1:** Start the Relativity Secret store service 

**Step 2:** : To view whitelisted machines, run this command:

**.\secretstore whitelist read**

If server is not whitelisted, whitelist servers on secretstore to grant permission by running the below command:

  **.\secretstore whitelist write (server-name)**

  For more details refer: https://help.relativity.com/Server2024/Content/System_Guides/Secret_Store/Secret_Store.htm

### Issue 3: Product cannot be installed, once or more secrets are invalid 

**Steps to troubleshoot :**

**Step 1**: Verify if the one-time setup using the

\`relsvr.exe\` CLI was executed properly and ensure that it has been

completed as required.

![](../resources/troubleshooting-images/one-or-more-secrets-invalid.png)

### Issue 4: Product cannot be installed because Elasticsearch service is not running

(#issue-2-elastic-search-service-issues)

**Steps to troubleshoot** 

Verify elasticsearch is running or not by following steps under [Elastic search service]

### Issue 5: Product cannot be installed because Kibana server is not running

![](../resources/troubleshooting-images/kibana-service-stopped.png)

**Steps to troubleshoot** 

Verify kibana is running or not by following steps under [kibana service](#issue-5-kibana-service-issues)

### Issue 6: Product cannot be installed because APM server is not running

**Steps to troubleshoot**

Verify apm server is running or not by following steps under [apm service](#issue-7-apm-service-issues)

### Issue 7: Relativity service is inaccessible

![](../resources/troubleshooting-images/relativity-service-inaccessible-error.png)

**Steps to troubleshoot**

- Verify user is able to successfully login to Relativity
- If unable to login to relativity, Check the Relativity services in IIS and start them.

![](../resources/troubleshooting-images/start-relativity.png)

- Verify whether the version matches the minimum required version as per the Environment Watch release. Also verify Service Host is running within the web server

If there are any issues, refer https://help.relativity.com/Server2024/Content/System_Guides/Service_Host_Manager.htm: