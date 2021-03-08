# Generic SQL Connector technical reference

This tutorial describes the steps you need to perform to automatically provision and deprovision users from Azure AD into a SQL DB.  For important details on what this service does, how it works, and frequently asked questions, see [Automate user provisioning and deprovisioning to SaaS applications with Azure Active Directory](../app-provisioning/user-provisioning.md).

## Capabilities Supported
> [!div class="checklist"]
> * Create users in a SQL DB
> * Remove users from a SL DB when they do not require access anymore
> * Keep user attributes synchronized between Azure AD and the SQL DB

> [!Note]
> Notable known directories or features not supported: Microsoft Active Directory Domain Services (AD DS), Password Change Notification, Service(PCNS), Exchange provisioning, Delete of Active Sync Devices,Support for TDescurityDescriptor,Oracle Internet Directory (OID)

## Prerequisites

## Step 1. Plan your provisioning deployment
1. Review the steps to configure [on-prem provisioning](https://linkToNewTutorial) 
2. Learn about [how the provisioning service works](../app-provisioning/user-provisioning.md).
3. Determine who will be in [scope for provisioning](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
4. Determine what data to [map between Azure AD and ServiceNow](../app-provisioning/customize-application-attributes.md). 

## Step 2. Download and install the provisioning agent and on-prem host

You will need to download the provisioning agent and ECMA connector host to provide connectivity to your application. [Review the detailed steps for downloading and installing the components.]()

## Step 3. Setup your SQL encironment
Create SQL Database in Azure.
https://docs.microsoft.com/en-us/azure/azure-sql/database/single-database-create-quickstart

The steps below show you how to setup Microsoft SQL Server 2019. You can skip this tep if you already have a SQL DB setup. 

1. Download SQL Server - [https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019](https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019)

1. Download SSMS - [https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN&amp;view=sql-server-ver15](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN&amp;view=sql-server-ver15)

1. Open SQL Server management studio (SSMS)

1. Click on Browse for more, select the database under database engine (in my case 1110VM), and connect.

![](RackMultipart20210115-4-mlm6xl_html_5deb250b6bf40dc6.png)

1. Right click databases and create a new database and provide a name. Mine is &quot;ECMA2Host\_AppName&quot;.

1. Click new query and copy in the query from the employees file. Ensure that the table name matches as highlighted below.

![](RackMultipart20210115-4-mlm6xl_html_59e4119366bf37c7.png)

1. Open the sap.dsn file and update the server name (in my case it is &quot;SERVER=1110VM&quot;).

1. Ensure that the database name matches the name from step 6.

## Step 4. Configure the ECMA host


## Step 5. Configure provisioning in Azure AD
1. Assign the agents to your application (get steps from preview doc).
2. Provide the URL and secret token (get steps from preview doc). 
2. [Determine who should be in scope for provisioning](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts).
3. [Assign users to your application](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users) if scoping is based on assignment to the application (recommended).
4. [Configure your attribute mappings.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes)
5. [Provision a user on-demand.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/provision-on-demand)
6. Add additional users to your application.
7. Turn provisioning on.

## Step 6. Monitor your deployment
Once you've configured provisioning, use the following resources to monitor your deployment:

1. Use the [provisioning logs](../reports-monitoring/concept-provisioning-logs.md) to determine which users have been provisioned successfully or unsuccessfully.
2. Check the [progress bar](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) to see the status of the provisioning cycle and how close it is to completion.
3. If the provisioning configuration seems to be in an unhealthy state, the application will go into quarantine. Learn more about quarantine states [here](../app-provisioning/application-provisioning-quarantine-status.md).  

## Troubleshooting tips


## Known issues

* LDAP referrals between servers not supported (RFC 4511/4.1.10)


## Appendix G: Building a demo SQL environment for testing
https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step
https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql


## Appendix H: Configuring the host using SQL

The following screenshots show you how to configure the host, if you are using the demo environment described above.

 


## Appendix B: Configuring the Generic SQL Connector for SQL Server

If you do not have a connector MA, but have SQL Server in your environment, then you can still validate the provisioning process, using the instructions in the Generic SQL Connector guide at [https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql) and  [https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step)

After creating an ODBC file, you can then use that file when creating a new Connector.



