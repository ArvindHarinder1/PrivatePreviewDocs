# Generic LDAP Connector - AD LDS Server Sample

This tutorial describes the steps you needed to automatically provision and deprovision users from Azure AD into AD LDS using the generic LDAP connector. For important details on what this service does, how it works, and frequently asked questions, see [Automate user provisioning and deprovisioning to SaaS applications with Azure Active Directory](../app-provisioning/user-provisioning.md).

## Capabilities Supported
> [!div class="checklist"]
> * Create users in AD LDS
> * Remove users from AD LDS when they do not require access anymore
> * Keep user attributes synchronized between Azure AD and AD LDS

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

## Step 3. Setup your AD LDS environment

**Install AD LDS Role**

1. From **Server Manager**, choose **Manage** > **Add Roles and Features**.
2. Click **Next** until you get to **Select server roles**.
3. Place a check next to **Active Directory Lightweight Directory Services**. When prompted click **Add Features** and then click **Next**.
4. Click **Next** two more times.
5. Click **Install**.

**Configure AD LDS**

1. From **Server Manager** menu bar, click the yellow Notification.
2. Click the **Run the Active Directory Lightweight Directory Services Setup Wizard** link.

- This can also be launche- by running `adaminstall.exe` from the %systemroot%\adam folder.

3. Click **Next** to continue the setup wizard.
4. Chose the default setup option of **A unique instance**.
5. Use the default Instance name of **instance1** and click **Next**.
6. Use the default recommended ports of 389 and 636 on a member server and click **Next**.

**NOTE** It's easier to manage this on a domain member server, or a stand alone server in a workgroup rather than a domain controller.

7. On the *Application Directory Partition* screen select **Yes, create an application directory partition** and enter a name that starts with CN=<name>, like CN=Contoso. Then click **Next**.

![CreateAppPartition](/.attachments/AAD-Synchronization/343347/CreateAppPartition.jpg)

8. Click **Next** to use the default install locations.
9. On the *Service Account Selection* page, select **This account** and enter the domain user account that will be used as the AD LDS service account, then click **Next**.

**NOTE**: Local accounts in stand-alone server can be specified using servername\username while domain accounts can be added using domain\username.

![SelectSA](/.attachments/AAD-Synchronization/343347/SelectSA.jpg)

10. Use the default option of **Currently logged on user**: domain\username as the account that will have administrative access to the AD LDS instance, then click **Next**.
11. Do not select anything on the *Importing LDIF Files* screen. click **Next**.
12. Click **Next** one last time then click **Finish**.

**Extend the AD LDS Schema to Support ECMA Connector Imports**

This LDF file will add the Person object and include support of the userPrincipalName and EmployeeID attributes.

1. Download the *AddPerson.ldf* file from one of these locations:

* CSS Wiki [here](/.attachments/AAD-Synchronization/343347/AddPerson.ldf).
* CSS Azure Identity - User Provisioning Health site has the file [here](https://microsoft.sharepoint.com/:u:/t/CloudIdentity2/EY-zML4xrk1JsMTDq6PMoaEBUX8d2sQvy4NhTo99GqkNXw?e=grfQsL)

2. Open an Administrator Command Prompt on the Windows AD LDS server.

3. Import the LDF file into the AD LDS instance using the account that has administrative rights to the AD LDS instance. Run this command from the directory where the LDF file is located.

```
ldifde -i -u -f AddPerson.ldf -s server:port -b username domain password -j . -c "cn=Configuration,dc=X" #configurationNamingContext
```

* Using an example domain account.

```
ldifde -i -u -f AddPerson.ldf -s deWeb:389 -b <username> <domainname> <password> -j . -c "cn=Configuration,dc=X" #configurationNamingContext
```

**NOTE**: If the server is a stand-alone in a Workgroup and not joined to a domain, enter the name of the computer where the domain name would go.

# Configure ECMA Connector for LDAP

To learn more about which connector DLLs are available with the ECMA connector installer see the [Configuration](https://supportability.visualstudio.com/AzureAD/_wiki/wikis/AzureAD?pageID=343348&anchor=configuration) section of the "Azure AD ECMA Connector" CSS wiki page.

1. Right-click the **Microsoft ECMA2Host Configuration Wizard** shortcut that was created on the desktop and choose **Run as administrator**.

- If the wizard has been run before a list of connectors with the options to **Edit** or **Delete**.
- If the debug tracing of the Wizard needs to be collected, see [Wizard Debug Logging](https://supportability.visualstudio.com/AzureAD/_wiki/wikis/AzureAD?pageID=326861&anchor=wizard-debug-logging) for guidance.

2. Click **New Connector**

This demo shows minimal settings needed and likely will not match your customer's.

## Properties tab

Be sure to record the *Name* and *Shared Token* for use later on when configuring outgoing provisioning in Azure AD. Also, customers can click **Import connector** to restore an exported configuration.

- **Name**: This is the name of the ECMA Connector and will be part of the URL.
- **Autosync timer (minutes)**: 120
- **Secret Token**: 123456 [This must be a string of 10-20 ASCII letters and/or digits.]
- **Extension DLL**: Click the drop-down and select the **Microsoft.IAM.Connector.GenericLdap.dll**.
- Click **Next** to move to the **Connectivity** tab.

**NOTE**: Depending on the connector DLL selected, the left hand side of the wizard may change to add additional tabs, such as *Connectivity*, as required by the connector. Click **Next** after completing each tab.

![FirsttimeSetupLDAP](/.attachments/AAD-Synchronization/343347/FirsttimeSetupLDAP.jpg =791x529)

## Connectivity tab

The LDAP connector DLL presents a **Connectivity** tab to define how to bind to the AD LDS instance.

- **Host**: Should contain the FQDN of the server. Run sysdm.cpl to get the *Full computer name*.
- **Port**: Should contain the TCP port that the AD LDS instance is listening on. The default port is `389` on a member server and the SSL port would be `636`.
- **Binding**: Select **Kerberos** from the drop-down for domain joined computers.
- **User Name**: Contains the name of the domain administrator that installed the AD LDS instance.
- **Password**: Contains the password of the domain administrator that installed the AD LDS instance.
- **Realm/Domain**: Contains the NetBIOS or FQDN of the Active Directory (AD) domain that the administrator account is located in.
* Click **Next** to proceed through the **Global** tab.

![ConnectivityToADLDS](/.attachments/AAD-Synchronization/343347/ConnectivityToADLDS.jpg =791x529)

## Global tab

This view lists the server name and supported SASL Mechanisms when Kerberos is selected. It also shows a summary of **Supported Controls** and **Mandatory Features Not Found**.

**NOTE**: On an AD LDS server **Mandatory Features Not Found** will show *(1.3.6.1.4.1.4203.1.5.3) True/False filters*. This is the Feature OID for LDAP_FEATURE_ABSOLUTE_FILTERS is not supported in rootdse. This can be ignored.

- Click **Next** to proceed through the **Partitions** tab. This also triggers a validation check of the that were credentials entered.

![GlobalCheck](/.attachments/AAD-Synchronization/343347/GlobalCheck.jpg =792x530)

## Partitions tab

This tab lists all partitions that are available to accept user accounts in in the AD LDS instance. If only one is present it will be selected.  If a connector does not have any partitions, leave the default partition selected. Select the partition where the users should be created.

* Click **Next** to move to the **Run Profiles** tab.

![SelectPartition](/.attachments/AAD-Synchronization/343347/SelectPartition.jpg =791x529)

## Run Profiles tab

Select the run profiles that the connector is capable of using. The export profile is mandatory, but *Full import* and *Delta import* are optional.  The default settings should suffice.

**NOTE**: If neither *Full import* nor *Delta import* is configured, the ECMA Connector host will not be able to determine whether an object already exists in the target system prior to exporting. This could result in users being exported more than once and appearing as duplicates in the target DB.

- Click **Next** to move to the **Export** tab. 

![RunProfiles](/.attachments/AAD-Synchronization/343347/RunProfiles.jpg =791x529)

## Export tab

- **Page Size** should be set to `100`
- Click **Next** to move to the **FullImport** tab.

![ExportPageSize](/.attachments/AAD-Synchronization/343347/ExportPageSize.jpg =791x529)

## FullImport tab

- **Page Size** should be set to `1000`.
- Click **Next** to move to the **Object Types** tab.

![FullImportPageSize](/.attachments/AAD-Synchronization/343347/FullImportPageSize.jpg =791x529)

## Object Types tab

The admin specifies the type of object that the target system will create that corresponds to the Azure AD user when the import takes place. Set the following:

- **Target object**: *Person*
- **Anchor**: *objectGUID*
- **Query attribute**: *displayName*
- **DN**: *displayName*
    - Check **Autogenerated** to have ECMA Connector autogenerate the DN of the user.
- Click **Next** to move to the **Select Attributes** tab.

![ObjectTypesLDAP](/.attachments/AAD-Synchronization/343347/ObjectTypesLDAP.jpg =791x529)

## Select Attributes tab

Add attribute mappings for each of the attribute that should be configured on the target system's user account. These will map to Azure AD user attributes being sent by SCIM to be added to the imported account.

The attribute names from the target database will be in the drop-down list.

- Type the name of the *Attribute* or select from the drop-down list.
- Click the **+** sign to add it to the list.
- repeat this for each attribute that needs to be added.
- Click **Next** to move to the **Deprovisioning** tab.

![SchemaMappingLDAP](/.attachments/AAD-Synchronization/343347/SchemaMappingLDAP.jpg =791x529)

## Deprovisioning tab

The default values for **Disable flow** and **Delete flow** are **None**. This should remain unchanged.

![Deprovisioning](/.attachments/AAD-Synchronization/343347/Deprovisioning.jpg =791x529)

# Verify Connectivity

Testing Inbound connectivity.

See [Verify Inbound TCP Port 8585 is Open]() for more information.

# Verify User Import to ECMA Connector Works

A PowerShell script can be used to create a test user using the ECMA connector endpoint. This check can be done before an Azure AD application is configured to Provision accounts.

The [Verify User Import to ECMA Connector Works]() section of the "Azure AD ECMA Connector" CSS wiki has a PowerShell script that calls to the ECMA connector endpoint and imports a test user. 

# Examine Users Imported to AD LDS

These steps allow the AD LDS admin to view the users imported in the AD LDS instance.

1. Sign into Windows with the account that has administrative rights to the AD LDS instance.
2. Press **Window**+**R** keys to launch the **Run** command.
3. Type **ldp.exe** and click **Run**.
4. In LDP, click **Connection** > **Connect...** in the menu at the top.
5. Type **localhost** as the server and the port should be 389. click **OK** to proceed.
6. Press the **CTRL**+**B** keys. Leave the **Bind type** set to **Bind as currently logged on user** and click **OK**.
7. Press the **CRTL**+**N** keys to clear the screen.
8. Press the **CTRL**+**T** keys to bring up te **Tree View** option.
* Select the **CN=Contoso** naming context from the drop-down list and click **OK**.
9. Click the **+** icon to the left of **CN=Contoso** and the Imported users should appear directly below as **CN=username@contoso.com,CN=Contoso**.


