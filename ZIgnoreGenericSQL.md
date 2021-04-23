# Generic SQL Connector 

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
5. Review [known limitations](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/KnownLimitations.md).

## Overview of the Generic SQL Connector

The Generic SQL Connector enables you to integrate the synchronization service with a database system that offers ODBC connectivity.  

From a high-level perspective, the following features are supported by the current release of the connector:

| Feature | Support |
| --- | --- |
| Connected data source |The Connector is supported with all 64-bit ODBC drivers*. It has been tested with the following: <li>Microsoft SQL Server & SQL Azure</li><li>IBM DB2 10.x</li><li>IBM DB2 9.x</li><li>Oracle 10 & 11g</li><li>Oracle 12c and 18c</li><li>MySQL 5.x</li> |
| Scenarios |<li>Object Lifecycle Management</li><li>Password Management</li> |
| Operations |<li>Full Import and Delta Import, Export</li><li>For Export: Add, Delete, Update, and Replace</li><li>Set Password, Change Password</li> |
| Schema |<li>Dynamic discovery of objects and attributes</li> |

> [!NOTE]
> *Connections to data sources not listed above are currently limited to read-only scenarios.

## Pre-requisites
Before you use the Connector, make sure you have the following on the synchronization server:

* Microsoft .NET 4.5.2 Framework or later
* 64-bit ODBC client drivers
* If you are using the connector to communicate with Oracle 12c, this requires Oracle Instant Client 12.2.0.1 or newer with the ODBC package.
* If you are using the connector to communicate with Oracle 18c, this requires Oracle Instant Client 18.3.0.0 or newer with ODBC Package, and the NLS_LANG system variable to be set to support UTF8 characters.

### Permissions in connected data source
To create or perform any of the supported tasks in Generic SQL connector, you must have:

* db_datareader
* db_datawriter

### Ports and protocols
For the ports required for the ODBC driver to work, consult the database vendor's documentation.

## Create a new Connector
To Create a Generic SQL connector, in **Synchronization Service** select **Management Agent** and **Create**. Select the **Generic SQL (Microsoft)** Connector.

![CreateConnector](./media/microsoft-identity-manager-2016-connector-genericsql/createconnector.png)

## Prepare the sample database
On a server running SQL Server, run the SQL script found in [Appendix A](#appendix-a). This script creates a sample database with the name GSQLDEMO. The object model for the created database looks like this picture:  
![Object Model](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/objectmodel.png)

Also create a user you want to use to connect to the database. In this walkthrough, the user is called FABRIKAM\SQLUser and located in the domain.

## Create the ODBC connection file
The Generic SQL Connector is using ODBC to connect to the remote server. First we need to create a file with the ODBC connection information.

1. Start the ODBC management utility on your server:  
   ![ODBC](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc.png)
2. Select the tab **File DSN**. Click **Add...**.  
   ![ODBC1](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc1.png)
3. The out-of-box driver works fine, so select it and click **Next>**.  
   ![ODBC2](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc2.png)
4. Give the file a name, such as **GenericSQL**.  
   ![ODBC3](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc3.png)
5. Click **Finish**.  
   ![ODBC4](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc4.png)
6. Time to configure the connection. Give the data source a good description and provide the name of the server running SQL Server.  
   ![ODBC5](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc5.png)
7. Select how to authenticate with SQL. In this case, we use Windows Authentication.  
   ![ODBC6](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc6.png)
8. Provide the name of the sample database, **GSQLDEMO**.  
   ![ODBC7](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc7.png)
9. Keep everything default on this screen. Click **Finish**.  
   ![ODBC8](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc8.png)
10. To verify everything is working as expected, click **Test Data Source**.  
    ![ODBC9](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc9.png)
11. Make sure the test is successful.  
    ![ODBC10](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc10.png)
12. The ODBC configuration file should now be visible in File DSN.  
    ![ODBC11](./media/microsoft-identity-manager-2016-connector-genericsql-step-by-step/odbc11.png)

We now have the file we need and can start creating the Connector.

## Step 2. Download and install the provisioning agent and on-prem host

You will need to download the provisioning agent and ECMA connector host to provide connectivity to your application. [Review the detailed steps for downloading and installing the components.]()

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

## Step 5. Configure provisioning in Azure AD
1. Add the application from the app gallery > navigate to the provisioning page
1. Assign the agents to your application (get steps from preview doc).
1. Provide the URL and secret token (get steps from preview doc). 
1. Click test connection and save. (note you have to wait **10 minutes** between assigning the agents to the application and testing connection) 
1. [Determine who should be in scope for provisioning](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts).
1. [Assign users to your application](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users) if scoping is based on assignment to the application (recommended).
1. [Configure your attribute mappings.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes)
1. [Provision a user on-demand.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/provision-on-demand)
1. Add additional users to your application.
1. Turn provisioning on.

## Step 6. Monitor your deployment
Once you've configured provisioning, use the following resources to monitor your deployment:

1. Use the [provisioning logs](../reports-monitoring/concept-provisioning-logs.md) to determine which users have been provisioned successfully or unsuccessfully.
2. Check the [progress bar](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) to see the status of the provisioning cycle and how close it is to completion.
3. If the provisioning configuration seems to be in an unhealthy state, the application will go into quarantine. Learn more about quarantine states [here](../app-provisioning/application-provisioning-quarantine-status.md).  

## Troubleshooting tips

* Review the known limiataions defined [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/KnownLimitations.mdc)
* Review the troubleshooting tips defined [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Troubleshooting.md)


## Building a demo SQL environment for testing
https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step
https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql


## Configuring the Generic SQL Connector for SQL Server

If you do not have a connector MA, but have SQL Server in your environment, then you can still validate the provisioning process, using the instructions in the Generic SQL Connector guide at [https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql) and  [https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step)

After creating an ODBC file, you can then use that file when creating a new Connector.



