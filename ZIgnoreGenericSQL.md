# Generic SQL Connector 

This document describes concenpts required to provision a user from Azure AD into a SQL data base. For a step by step tutorial to provision a user into a SQL database, review the documentation [here.](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/2ConnectorGSQLToSQLServer.md) 

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



### Ports and protocols
For the ports required for the ODBC driver to work, consult the database vendor's documentation.

## Create a new Connector
To Create a Generic SQL connector, in **Synchronization Service** select **Management Agent** and **Create**. Select the **Generic SQL (Microsoft)** Connector.

![CreateConnector](./media/microsoft-identity-manager-2016-connector-genericsql/createconnector.png)

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

## Troubleshooting the Generic SQL connector
### Identify the query the ECMA host made to the SQL DB
1. Enable verbose logging as described here.
2. Review the event logs to identify the  request made. 


### Test the SQL query directly in your database
Sometimes, importing users from the SQL database may fail. Verify that the request the ECMA host is making to the SQL database works when made directly in the SQL database. To accomplish this, identify the query the ECMA host made, as described above. Navigate to SSMS to make the request yourself. 

## Configuring the Generic SQL Connector for SQL Server

If you do not have a connector MA, but have SQL Server in your environment, then you can still validate the provisioning process, using the instructions in the Generic SQL Connector guide at [https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql) and  [https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step)

After creating an ODBC file, you can then use that file when creating a new Connector.



