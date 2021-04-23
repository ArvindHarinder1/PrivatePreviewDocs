# Generic SQL Connector - SQL Server Sample

This tutorial describes the steps you need to perform to automatically provision and deprovision users from Azure AD into a SQL server using the generic SQL connector. For important details on what this service does, how it works, and frequently asked questions, see [Automate user provisioning and deprovisioning to SaaS applications with Azure Active Directory](../app-provisioning/user-provisioning.md).

## Capabilities Supported
> [!div class="checklist"]
> * Create users in a SQL DB
> * Remove users from a SL DB when they do not require access anymore
> * Keep user attributes synchronized between Azure AD and the SQL DB

> [!Note]
> Notable known directories or features not supported: Microsoft Active Directory Domain Services (AD DS), Password Change Notification, Service(PCNS), provisioning group objects and memberships. 

## Step 1. Plan your provisioning deployment
1. Review the steps to configure [on-prem provisioning](https://linkToNewTutorial) 
2. Learn about [how the provisioning service works](../app-provisioning/user-provisioning.md).
3. Determine who will be in [scope for provisioning](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
4. Determine what data to [map between Azure AD and ServiceNow](../app-provisioning/customize-application-attributes.md). 
5. Review [known limitations](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/KnownLimitations.md).

## Step 2. Download and install the provisioning agent and ECMA host

You will need to download the provisioning agent and ECMA connector host to provide connectivity to your application as described in step 2 [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/1ECMATutorial.md).

## Step 3. Setup your SQL encironment

The steps below show you how to setup Microsoft SQL Server 2019. You can skip this tep if you already have a SQL DB setup. 

1. Download [SQL Server](https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019).
1. Download SQL Server management studio [SSMS](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN&amp;view=sql-server-ver15).
1. Search for SSMS in the Windows search bar and open SSMS.
1. Click on Browse for more, select the database under database engine (in my case 1110VM), and connect. ![image](https://user-images.githubusercontent.com/36525136/115303696-cc12c000-a118-11eb-931f-2dd0ac3257a6.png)
1. Right click databases and create a new database and provide a name. In the example below, the database is named ECMA2Host\_AppName.
1. Click new query and copy in the query from the employees file. Ensure that the table name matches as highlighted below. ![image](https://user-images.githubusercontent.com/36525136/115303773-e3ea4400-a118-11eb-9655-6e3e314c2b49.png)
1. Open the sap.dsn file and update the server name (in my case it is SERVER=1110VM;).
1. Ensure that the database name matches the name from step 6.

## Step 4. Configure the ECMA host
The following screenshots show you how to configure the host, if you are using the demo environment described above.

![image](https://user-images.githubusercontent.com/36525136/115303969-2ca1fd00-a119-11eb-9047-201ebcf0c2c6.png)

![image](https://user-images.githubusercontent.com/36525136/115303996-362b6500-a119-11eb-9a46-327b20cf500f.png)

![image](https://user-images.githubusercontent.com/36525136/115304025-404d6380-a119-11eb-9fb7-c45838953780.png)

![image](https://user-images.githubusercontent.com/36525136/115304044-480d0800-a119-11eb-8ca9-113ae5c30454.png)

![image](https://user-images.githubusercontent.com/36525136/115304068-522f0680-a119-11eb-97d8-fa9987771de9.png)

![image](https://user-images.githubusercontent.com/36525136/115304082-59eeab00-a119-11eb-842b-17f126c089b0.png)

![image](https://user-images.githubusercontent.com/36525136/115304110-65da6d00-a119-11eb-92fd-4b1863d14a6d.png)

![image](https://user-images.githubusercontent.com/36525136/115304249-96220b80-a119-11eb-88ef-3d7d2075d68f.png)

![image](https://user-images.githubusercontent.com/36525136/115304275-9e7a4680-a119-11eb-9bf5-aa47fa78765f.png)

![image](https://user-images.githubusercontent.com/36525136/115304299-a639eb00-a119-11eb-95ba-59e62b25ef80.png)

## Step 5. Configure provisioning in Azure AD and Monitor
Configure provisioning in Azure AD as described in step 4 and step 5 [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/1ECMATutorial.md).

## Troubleshooting tips

* Review the known limiataions defined [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/KnownLimitations.mdc)
* Review the troubleshooting tips defined [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Troubleshooting.md)

