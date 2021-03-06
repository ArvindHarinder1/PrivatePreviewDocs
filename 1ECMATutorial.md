# Azure AD Outbound Provisioning with the Azure AD ECMA Connector Host Private Preview

Microsoft Confidential – Not for distribution

This guide outlines how to validate that an Extensible Connectivity Management Agent (ECMA)-based connector, that today is used with FIM or MIM, will work with the upcoming Azure AD outbound provisioning via the Azure AD ECMA Connector Host, currently in private preview.

In you have feedback on this private preview, or if you have questions, please contact us at [msiga@microsoft.com](mailto:msiga@microsoft.com).  Please do not use other Internet forums for discussing this preview, as this preview is not a generally available or supported release.

The purpose of this preview is to determine whether existing Extensible Connectivity Management Agent (ECMA) connectors implementing the [Microsoft Identity Manager (MIM)](https://docs.microsoft.com/microsoft-identity-manager/microsoft-identity-manager-2016) [Extensible Connectivity 2.2 Management Agent (ECMA2) API](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/hh859557(v=vs.100)?redirectedfrom=MSDN) can also be used for provisioning to target systems, independent of MIM.

ECMA [connectors](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/hh859557(v=vs.100)?redirectedfrom=MSDN) are used in FIM and MIM for inbound and outbound provisioning. (Azure AD Connect also uses the ECMA API internally for the generic LDAP connectors.)

In this preview, Azure AD outbound provisioning requests to create and update users are mapped into equivalent ECMA calls, by a new component, the Azure AD ECMA Connector Host.

Setting up this preview starts with your on-premises or IaaS-hosted VM, where you will set up the Azure AD ECMA Connector host with a new configuration for a connector, or import a configuration which was previously exported from a MIM sync service.  Then, you will configure in the Azure portal for Azure AD outbound provisioning to send SCIM operations to the Azure AD Connector host.

Once the setup described in this document is complete, then a user assignment to an application in Azure AD will result, shortly afterward, in an equivalent call to ECMA export APIs to provision user to the target system.  This is a similar model as with other SaaS applications.  Note that other MIM features such as password synchronization or full export are not possible with Azure AD.

![image](https://user-images.githubusercontent.com/36525136/116119937-f8848a00-a68c-11eb-82c5-ddca1cad019d.png)

This document provides the general steps for configuring on-premises application provisioning. For specific steps configuring the [SQL](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/2ConnectorGSQLToSQLServer.md) or [LDAP](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericldap) connectors, please review the connector specific tutorials. 

## Prerequisites for this preview

This preview requires the following in the environment:

- A target system, such as a SQL database, or LDAP directory (excluding AD DS), in which users can be created, updated and deleted.

- An ECMA 2.0 or later connector for that target system, which supports export, schema retrieval, and optionally full import or delta import operations.  If you do not have an ECMA Connector ready during configuration, then you can still validate the end-to-end flow if you have a SQL Server in your environment and use the Generic SQL Connector.

- A Windows Server 2016 or later computer with an Internet-accessible TCP/IP address, connectivity to the target system, and with outbound connectivity to login.microsoftonline.com (for example, a Windows Server 2016 virtual machine hosted in Azure IaaS or behind a proxy). The server should have at least 3GB of RAM.

- NET Framework 4.7.1 on the machine the agent / host are installed on.

- An Azure AD tenant with Azure AD Premium P1 or Premium P2 (or EMS E3 or E5).  You can get a free developer test tenant [here](https://developer.microsoft.com/office/dev-program). The tenant where the preview is being configured cannot be located in one of the  **European Union** ,  **European Economic Area** , candidate for inclusion in the European Union,  **China** , or  **Switzerland ** country/regions.  Tenants that are deployed in Azure Government, China, or other specialized cloud are not currently available for use in this preview.  A list of country/regions that are not currently available for use in this preview as well as instructions to check the country or region that your tenant is in, is included in Appendix C.  (This restriction will be removed at a later date.)   The ECMA connector host preview is not intended for use with production target systems, this tenant should hold only simulated/test data.


## Step 1. Plan your provisioning deployment
1. Review the [on-prem provisioning architecture](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/3Architecture.md) 
2. Learn about [how the provisioning service works](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/how-provisioning-works).
3. Determine who will be in [scope for provisioning](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts).
4. Determine what data to [map between Azure AD and your target application](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes).
5. Review the [known limitations](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/known-issues).

## Step 2. Download and install the provisioning agent and on-prem host
### Download and  install the provisioning agent / host
The provisioning agent and ECMA host are two separate windows services that are installed using one installer, deployed on the same machine. They should have connectivity to the target application that you are looking to provision users into.   

1. Sign into the [Azure Portal](https://portal.azure.com)
2. Navigate to enterprise applications > Add a new application
3. Search for the "On-premises provisioning" application and add it to your tenant 
![image](https://user-images.githubusercontent.com/36525136/117818662-d404e200-b236-11eb-913b-63b22df0c1a8.png)
4. Navigate to the provisioning blade
5. Click on on-premises connectivity
6. Download the agent installer 
7. Open the agent installer > agree to the terms of service > click install
8. A shortcut should be created on your desktop to launch the Azure AD Connect Provisioning Agent Wizard. 

![image](https://user-images.githubusercontent.com/36525136/115305138-be5e3a00-a11a-11eb-9453-e0f0d48b94e5.png)

### Configure the provisioning agent
1. Launch the provisioning agent wizard from your desktop. 
2. When prompted to select an extension, select the "on-prem provisioning" option.
3. Click authorize and provide Azure AD credentials. The credentials provided should be for a user that is a global administrator or hybrid administrator.  
4. Click confirm.

Note: You will see two steps about providing AD credentials and setting up gMSA. Those steps are not required and will automatically be skipped in the latest version of the provisioning agent. 

![image](https://user-images.githubusercontent.com/36525136/115305268-eb125180-a11a-11eb-83d8-e266b11efbd4.png)

## Step 3. Configure the ECMA host

1. Navigate to the start menu and identify the Microsoft ECMA Host application. **Open this as an administrator.** ![image](https://user-images.githubusercontent.com/36525136/115305371-11d08800-a11b-11eb-8f4f-bbde282eb1f7.png)

2. Generate the certificate for connectivity to the provisioning agent. The auto-generated certificate will be self-signed / part of the trusted root and the SAN matches the hostname. 
![image](https://user-images.githubusercontent.com/36525136/115305574-607e2200-a11b-11eb-8c47-30c7acced3eb.png)

3. A new window will appear with a list of connectors. The first time this is run, no connector configurations will be present. Click New Connector.  
![image](https://user-images.githubusercontent.com/36525136/115305658-84416800-a11b-11eb-9917-3f1f408ad954.png)

4. Walk through the pages of the ECMA host to configure your connector. For more details, see the tutorials linked below:
   1. [SQL ECMA tutorial](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/2ConnectorGSQLToSQLServer.md)
   1. [Generic SQL MIM tutorial](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql)
   1. [Generic LDAP MIM tutorial](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericldap)

## Step 4. Configure provisioning in Azure AD
### Establish connectivity between Azure AD and the ECMA Host 

1. Check to ensure that the connector host Windows Service is running.  Click on the start menu, and type services. In the services list, scroll to Microsoft ECMA2Host.  Ensure that the status is Running.  If the status is blank, click Start.

1. Sign into Azure Portal as an application administrator in the tenant used to register the provisioning agent. The tenant should have Azure AD Premium P1 or Premium P2 (EMS E3 or E5). This is required to be able to provision to an on-prem application. 

1. In the Azure Portal, navigate to Azure Active Directory area, navigate to Enterprise Applications, and click on New Application.

1. Search for Provisioning Private Preview Test Application and add it to your tenant.  

1. Once the app has been created, click on the Provisioning page.

1. Click get started.

1. Change the Provisioning Mode to Automatic. Additional settings will appear on that screen.

1. In the on-premises connectivity section, select the agent that you just deployed and click assign agent(s).

1. Before performing the next step,  **wait 10 minutes**  for the agent registration to complete. Test connection will not succeed until the agent registration is completed. Alternatively, you can force the agent registration to complete by restarting the provisioning agent on your server. Navigating to your server > search for services in the windows search bar > identify the Azure AD Connect Provisioning Agent Service > right click on the service and restart. 

1. In the tenant URL field, enter the following URL, replacing "connectorName" with the name of the connector on the ECMA Host.   

[https://localhost:8585/ecma2host_connectorName/scim](https://localhost:8585/ecma2host_connectorName/scim)

1. Enter the secret token value that you defined in the ECMA Host in the field Secret Token.

1. Click Test Connection and wait one minute. If you reveive an error message, please review the [troubleshooting steps](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Troubleshooting.md). 

1. Once connection test is successful, click save.  

### Configure who is in scope for provisioning

Azure AD allows you to scope who should be provisioned based on assignment to an application and / or by filtering on a particular attribute. [Determine who should be in scope for provisioning](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts) and define your scoping rules as necessary. Most customers will stick with the default scope of "assigned users and groups," without doing any additional scoping configuration. 

### Assign users to your application

If you chose scoping based on assignment in the previous step (recommended), please [assign users and / or groups to your application](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users).

### Configure your attribute mappings
You will need to map the user attributes in Azure AD to the attributes in the target application. The Azure AD Provisioning service relies on the SCIM standard for provisioning and as a result, the attributes surfaced have the SCIM name space. The example below shows how you can map the country and objectId attributes in Azure AD to the Country and InternalGUID attributes in an application. Note, the default mapping contains userPrincipalName to an attribute name PLACEHOLDER. You will need to change the PLACEHOLDER attribute to one that is found in your application. To change the plpaceholder attribute, click on the userPrincipalName attribute to edit the row in the attribute mappings page. [Learn more](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes#editing-user-attribute-mappings) about configuring attribute mappings.

| **Attribute name in Azure AD**   | **Attribute name in SCIM**   | **Attribute name in target application**   | 
| --- | --- | --- |
| country  | urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:Country  | Country  | 
| objectId  |  urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:InternalGUID | InternalGUID | 


### Test your configuration by provisioning users on demand

[Provision a user on-demand.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/provision-on-demand)

1. Navigate to the single sign-on blade and then back to the provisioning blade. From the new provisioning overview blade, click on on-demand. 

1. Test provisioning a few users on-demand as described [here](https://aka.ms/provisionondemanddocumentation).

1. Once on-demand provisioning is successful, change back to the provisioning configuration page. Ensure that the scope is set to only assigned users and group, turn provisioning On, and click Save.

1. Wait several minutes for provisioning to start (it may take up to 40 minutes).  You can learn more about the provisioning service performance [here](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/application-provisioning-when-will-provisioning-finish-specific-user). After the provisioning job has been completed, as described in the next section, you can change the provisioning status to Off, and click Save. This will stop the provisioning service from running in the future.


### Validating user provisioning from Azure AD has been successful

Azure AD provisioning service will send a GET request with a test user to validate the tenant URL secret token and the format of the response.

Then, once the Azure AD provisioning service has been enabled, the Azure AD provisioning service will send SCIM GET, POST and PATCH operations to the connector host.   When the connector host receives one of these, it will begin an export connection, and export one or more changes to the connector.  Based on the export result code, the connector host will return success or an error indication to the Azure AD provisioning service.

For example, if a user is assigned to an application, then the Azure AD provisioning service will check if the user already exists, by querying the connector host.  If the user already exists, then Azure AD will either continue with the next user, or update that user with a PATCH with the latest attributes, if needed. However, if the user is not already present, then the Azure AD provisioning service will then send a POST request to create the user.  A POST request will be transformed by the connector host into a PutExportEntries call with an ObjectModificationType of Add, and the attributes named in the configuration of the connector.

If you are unsure if the Azure AD provisioning service has attempted to contact the connector host, start by checking the Azure AD provisioning logs..  Next, check for any  log messages on the ECMA Connector Host at the time of an update – if there are any errors or warnings they will indicate the connector host was unable to transform a request into an ECMA call, or the connector returned an exception, as described for the next section.


### Add additional users to your application
Assign additional users and groups to your [application](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/assign-user-or-group-access-portal
). 

### Turn provisioning on
Click start to enable provisioning. The service will then synchronize all users and groups in scope. 

## Step 5. Monitor your deployment
1. Use the [provisioning logs](../reports-monitoring/concept-provisioning-logs.md) to determine which users have been provisioned successfully or unsuccessfully.
1. Build custom alerts, dashboards, and queries using the [Azure Monitor integration](https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-log-analytics).
1. If the provisioning configuration seems to be in an unhealthy state, the application will go into quarantine. Learn more about quarantine states [here](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/application-provisioning-quarantine-status).  


## Next steps
* Review the [SQL](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/2ConnectorGSQLToSQLServer.md) and [LDAP](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericldap) specific tutorials for more detailed guidance
* Review the [troubleshooting tips](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Troubleshooting.md) to diagnose and resolve configuration issues.


## Feedback

In you have feedback on this private preview, please report any issues at [https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyEYaD-36kVGmx4Eyyk10npUMUtDN0tLRlRVTkVOSzBGS0NDSlFYSVpLRC4u](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyEYaD-36kVGmx4Eyyk10npUMUtDN0tLRlRVTkVOSzBGS0NDSlFYSVpLRC4u) or if you have questions, please contact us at [msiga@microsoft.com](mailto:msiga@microsoft.com).


