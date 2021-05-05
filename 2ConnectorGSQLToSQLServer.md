# Generic SQL Connector - SQL Server Sample

This tutorial describes the steps you needed to automatically provision and deprovision users from Azure AD into a SQL server using the generic SQL connector. For important details on what this service does, how it works, and frequently asked questions, see [Automate user provisioning and deprovisioning to SaaS applications with Azure Active Directory](../app-provisioning/user-provisioning.md).

## Capabilities Supported
> [!div class="checklist"]
> * Create users in a SQL DB
> * Remove users from a SQL DB when they do not require access anymore
> * Keep user attributes synchronized between Azure AD and the SQL DB

> [!Note]
> Notable known directories or features not supported: Microsoft Active Directory Domain Services (AD DS), Password Change Notification, Service(PCNS), provisioning group objects, and group memberships. 

## Step 1. Plan your provisioning deployment
1. Review the steps to configure [on-prem provisioning](https://linkToNewTutorial) 
2. Learn about [how the provisioning service works](../app-provisioning/user-provisioning.md).
3. Determine who will be in [scope for provisioning](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
4. Determine what data to [map between Azure AD and ServiceNow](../app-provisioning/customize-application-attributes.md). 
5. Review [known limitations](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/KnownLimitations.md).

## Step 2. Download and install the provisioning agent and ECMA host

You will need to download the provisioning agent and ECMA connector host to provide connectivity to your application as described [here in step 2.](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/1ECMATutorial.md)

## Step 3. Setup your SQL environment

The steps below show you how to setup Microsoft SQL Server 2019. You can skip this step if you already have a SQL DB setup. 

1. Download [SQL Server](https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019).
1. Download SQL Server management studio [SSMS](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN&amp;view=sql-server-ver15).
1. Search for SSMS in the Windows search bar and open it.
1. Click on Browse for more, select the database under database engine (in my case 1110VM), and connect. ![image](https://user-images.githubusercontent.com/36525136/115303696-cc12c000-a118-11eb-931f-2dd0ac3257a6.png)
1. Right click databases and create a new database and provide a name. In the example below, the database is named ECMA2Host\_AppName.
1. Click new query and copy in the query from the employees file (found in the same zip file as the provisioning agent). Ensure that the table name matches as highlighted below. ![image](https://user-images.githubusercontent.com/36525136/115303773-e3ea4400-a118-11eb-9655-6e3e314c2b49.png)
1. Open the sap.dsn file (found in the same zip file as the provisioning agent) in notepad and update the server name (in this example it is SERVER=1110VM).
1. Ensure that the database name matches the name from step 6.

## Step 4. Configure the ECMA host
The following screenshots show you how to configure the host, if you are using the demo environment described above.

Enter a **Name** for the connector, and a **Shared Token**.

**IMPORTANT**: Be sure to record the **Name** and **Shared Token** for use later on when configuring outgoing provisioning in Azure AD.

- **Name**: This is the name of the ECMA Connector and will be part of the URL.
- **Autosync timer (minutes)**: Minimum allowed is `120` minutes.
- **Secret Token**: `123456` [This must be a string of 10-20 ASCII letters and/or digits.]
- **Extension DLL**: Click the drop-down and select the **Microsoft.ISM.Connector.GenericSql.dll**.
- Click **Next** to move to the **Connectivity** tab.

**NOTE**: Depending on the connector DLL selected, the left hand side of the wizard may change to add additional tabs, such as *Connectivity*, as required by the connector. Click **Next** after completing each tab.
![image](https://user-images.githubusercontent.com/36525136/115303969-2ca1fd00-a119-11eb-9047-201ebcf0c2c6.png)


- **User Name**: This must be in the form of `hostname\sqladminaccount` for standalone servers, or `domain\sqladminaccount` for domain member servers.
- Provide the DSN file that can be found in the [sharepoint site](https://aka.ms/onpremprovisioning) and update the server name in the file to match yours.
- Click **Next** to proceed through the *Schema Detection* tabs.

**NOTE**: Unless the customer's environment is known to require these settings, leave **DN is Anchor** and **Export Type:Object Replace** deselected.

![image](https://user-images.githubusercontent.com/36525136/115303996-362b6500-a119-11eb-9a46-327b20cf500f.png)

- **Object type detection method**: This should be set to `Fixed Value`.
    - **Fixed value list/Table/View/SP**: This should contain `User`.
    - Click **Next** to move to the next schema tab.

![image](https://user-images.githubusercontent.com/36525136/115304025-404d6380-a119-11eb-9fb7-c45838953780.png)

- **User:Attribute Detection**: This should be set to `Table`.
    - **User:Table/View/SP**: This should contain `Employees`.
    - Click **Next** to move to the next schema tab.

![image](https://user-images.githubusercontent.com/36525136/115304044-480d0800-a119-11eb-8ca9-113ae5c30454.png)

- Place a check next to `User:SAPLogin`.
- **Select DN attribute for users** should then be set to `UPN`.
- Click **Next** to move to the next schema tab.

![image](https://user-images.githubusercontent.com/36525136/115304068-522f0680-a119-11eb-97d8-fa9987771de9.png)

- **Delta Strategy:**: This should default to `None`*`
- **Water Mark Query**: This should default to `*`SELECT GETDATE()`*`
- **Data Source Time Zone**: This can be set to the desired time zone, or use the default of `(UTC) Coordinated Universal Time`.
- **Data Source Date Time Format**: This should be set to the format desired. If prompted to populate this field, the default is `yyyy-MM-dd HH:mm:ss`.
- **Operation Methods**: This should default to `Password extension`.
- Click **Next** to move to the **Partitions** tab.  This also triggers a validation check of the that were credentials entered.

**NOTE**: No other parameters need to be set.
![image](https://user-images.githubusercontent.com/36525136/115304082-59eeab00-a119-11eb-842b-17f126c089b0.png)


![image](https://user-images.githubusercontent.com/36525136/115304110-65da6d00-a119-11eb-92fd-4b1863d14a6d.png)

- **Operation Method**: This should be set to `Table`.
- **Table/View/SP**: This should contain `users`.
- **Page Size**: Scroll to the very bottom of the page. This should be set to `100`.
- Click **Next** to move to what will be the **FullImport** tab.
**NOTE**: Everything else can be blank.
![image](https://user-images.githubusercontent.com/36525136/115304249-96220b80-a119-11eb-88ef-3d7d2075d68f.png)

* **Anchor** - this attribute should be uniqie in the target system. The Azure AD provisioning service will query the ECMA host using this attribute after the initial cycle. This anchor value should be the same as the anchor value in schema 3. 
* **Query attribute** - used by the ECMA host to query the in-memory cache. This attribute should be unique. 
* **DN** - The autogenerate option should be selected in most cases. If deselected, ensure that the DN attribute is mapped to an attribute in Azure AD that stores the DN in this format: CN = anchorValue, Object = objectType

![image](https://user-images.githubusercontent.com/36525136/116597748-fcfeac00-a8f3-11eb-88fa-9b1d28b48393.png)

* The ECMA host discovers the attributes supported by the target system. You can choose which of those attributes you would like to expose to Azure AD. These attributes can then be configured in the Azure Portal for provisioning. 

![image](https://user-images.githubusercontent.com/36525136/116597847-1acc1100-a8f4-11eb-954c-4eea166f299d.png)


![image](https://user-images.githubusercontent.com/36525136/115304299-a639eb00-a119-11eb-95ba-59e62b25ef80.png)

## Step 5. Configure provisioning in Azure AD and Monitor
Configure provisioning in Azure AD as described in step 4 and step 5 [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/1ECMATutorial.md).

## Troubleshooting tips

* Review the known limitations defined [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/KnownLimitations.mdc)
* Review the troubleshooting tips defined [here](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Troubleshooting.md)

