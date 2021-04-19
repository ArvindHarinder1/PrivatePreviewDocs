# Azure AD Outbound Provisioning with the Azure AD ECMA Connector Host Private Preview

Microsoft Confidential – Not for distribution

This guide outlines how to validate that an Extensible Connectivity Management Agent (ECMA)-based connector, that today is used with FIM or MIM, will work with the upcoming Azure AD outbound provisioning via the Azure AD ECMA Connector Host, currently in private preview.

In you have feedback on this private preview, or if you have questions, please contact us at [msiga@microsoft.com](mailto:msiga@microsoft.com).  Please do not use other Internet forums for discussing this preview, as this preview is not a generally available or supported release.

Overview of the Azure AD outbound provisioning via Azure AD ECMA Connector Host preview

The purpose of this preview is to determine whether existing Extensible Connectivity Management Agent (ECMA) connectors implementing the [Microsoft Identity Manager (MIM)](https://docs.microsoft.com/microsoft-identity-manager/microsoft-identity-manager-2016) [Extensible Connectivity 2.2 Management Agent (ECMA2) API](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/hh859557(v=vs.100)?redirectedfrom=MSDN) can also be used for provisioning to target systems, independent of MIM.

ECMA [connectors](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/hh859557(v=vs.100)?redirectedfrom=MSDN) are used in FIM and MIM for inbound and outbound provisioning. (Azure AD Connect also uses the ECMA API internally for the generic LDAP connectors.)

In this preview, Azure AD outbound provisioning requests to create and update users are mapped into equivalent ECMA calls, by a new component, the Azure AD ECMA Connector Host.

Setting up this preview starts with your on-premises or IaaS-hosted VM, where you will set up the Azure AD ECMA Connector host with a new configuration for a connector, or import a configuration which was previously exported from a MIM sync service.  Then, you will configure in the Azure portal for Azure AD outbound provisioning to send SCIM operations to the Azure AD Connector host.

Once the setup described in this document is complete, then a user assignment to an application in Azure AD will result, shortly afterward, in an equivalent call to ECMA export APIs to provision user to the target system.  This is a similar model as with other SaaS applications.  Note that other MIM features such as password synchronization or full export are not possible with Azure AD.

![image](https://user-images.githubusercontent.com/36525136/115304670-10eb2680-a11a-11eb-8e95-3f01de6586ce.png)

This document provides the general steps for configuring on-premises application provisioning. For specific steps configuring the [SQL](input link) or LDAP (input link) connectors, please review the connector specific tutorials. 

## Prerequisites for this preview

This preview requires the following in the environment:

- A target system, such as a SQL database, or LDAP directory, in which users can be created, updated and deleted.

- An ECMA 2.0 or later connector for that target system, which supports export, schema retrieval, and optionally full import or delta import operations.  If you do not have an ECMA Connector ready during configuration, then you can still validate the end-to-end flow if you have a SQL Server in your environment and use the Generic SQL Connector.

- A Windows Server 2016 or later computer with an Internet-accessible TCP/IP address, connectivity to the target system, and with outbound connectivity to login.microsoftonline.com (for example, a Windows Server 2016 virtual machine hosted in Azure IaaS or behind a proxy). The server should have at least 3GB of RAM.

- An Azure AD tenant with Azure AD Premium P1 or Premium P2 (or EMS E3 or E5).  You can get a free developer test tenant [here](https://developer.microsoft.com/office/dev-program). The tenant where the preview is being configured cannot be located in one of the  **European Union** ,  **European Economic Area** , candidate for inclusion in the European Union,  **China** , or  **Switzerland ** country/regions.  Tenants that are deployed in Azure Government, China, or other specialized cloud are not currently available for use in this preview.  A list of country/regions that are not currently available for use in this preview as well as instructions to check the country or region that your tenant is in, is included in Appendix C.  (This restriction will be removed at a later date.)   The ECMA connector host preview is not intended for use with production target systems, this tenant should hold only simulated/test data.


## Step 1. Plan your provisioning deployment
1. Review the [on-prem provisioning architecture](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/3Architecture.md) 
2. Learn about [how the provisioning service works](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/how-provisioning-works).
3. Determine who will be in [scope for provisioning](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts).
4. Determine what data to [map between Azure AD and your target application](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes).
5. Review the [known limitations](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/known-issues) 


## Step 2. Download and install the provisioning agent and on-prem host
### Download and  install the provisioning agent / host
The provisioning agent and host are two separate windows services that are installed using one installer. They are expected to be deployed on the same server. 

1. Sign into the Azure Portal
2. Naviage to enterprise applications > Add a new application
3. Search for the provisioning private preview test application and add it to your tenant
4. Navigate to the provisioning blade
5. Click on on-premises connectivity
6. Download the agent installer in step 1 (you will need to copy it into the server that the app is hosted in if you're using 
7. Open the agent installer > agree to the terms of service > click install

![image](https://user-images.githubusercontent.com/36525136/115305138-be5e3a00-a11a-11eb-9453-e0f0d48b94e5.png)

### Configure the provisioning agent
1. Launch the provisioning agent wizard. A shortcut should be available in your desktop after completing the step above.
2. When prompted to select an extension, select the on-prem provisioning option.
3. Click authorize and provide Azure AD credentials. The credentials provided should be for a user that is a global administrator or hybrid administrator.  
4. Click confirm.

Note: You will see two steps about providing AD credentials and setting up gMSA. Those steps are not required and will automatically be skipped in the latest version of the provisioning agent. 

![image](https://user-images.githubusercontent.com/36525136/115305268-eb125180-a11a-11eb-83d8-e266b11efbd4.png)

## Step 3. Configure the ECMA host

1. Navigate to the start menu and identify the Microsoft ECMA Host application. **Open this as an administrator.** ![image](https://user-images.githubusercontent.com/36525136/115305371-11d08800-a11b-11eb-8f4f-bbde282eb1f7.png)

2. Generate the certificate for connectivity to the provisioning agent.
3. A new window will appear with a list of connectors. The first time this is run, no connector configurations will be present.  Click &quot;New Connector&quot;.
4. Pick and record the name of the connector, and a shared secret for use on the new connector properties page. The name of the connector should be exclusively ASCII letters and digits, the autosync timer should be 240 minutes, and the secret token a string of 10-20 ASCII letters and digits.    (You&#39;ll use the secret token later in configuring outgoing provisioning in Azure AD). For Connector, select from the drop-down the name DLL that was placed in the ECMA folder at the earlier step. Then click &quot;Next&quot;.
5. The left-hand side of the wizard window may change to add additional tabs, such as Connectivity, as required by the connector.  Click &quot;Next&quot; after completing each tab.
6. On the Partitions tab, if the connector does not have any partitions, leave the default partition selected, and click Next.
7. On the Run Profiles tab, select the run profiles which the connector is capable of being used in.  The export profile is mandatory, but full and delta import are optional.  Note that if neither full import nor delta import is configured, the ECMA Connector host will not be able to determine whether an object already exists in the target system prior to exporting it.
8. On the Export tab, specify the page size – default to 100.
9. On the Object Types tab, select which object type provided by the target system corresponds to an Azure AD user. Then, select the anchor attribute for that object type.
10. On the Schema Mapping tab, configure the unique identifier attribute for the Azure AD user.  The unique identifier will correspond to the primary key attribute that will be configured later in Azure AD portal. Then, add attribute mappings for each attribute in your target system&#39;s user to the corresponding Azure AD attribute, by clicking &quot;Add attribute mapping&quot; tab, selecting the attribute name of the connected system, and the corresponding SCIM attribute.  By default, the attribute names from SCIM are displayed; if you do not see a corresponding attribute, then type in the attribute name.  In the configuration wizard, custom attributes that are not displayed in the drop down must be written in the same format as in Azure AD portal, 
e.g.urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:CustomAttribute:User
11. On the Deprovisioning tab, configure the desired behavior when a user&#39;s account in Azure AD is disabled from being able to sign in, and the desired behavior when a user&#39;s account in Azure AD is deleted.  If you do not wish users to be removed from the target system, then select &quot;None&quot; for the delete flow. Then click &quot;Finish&quot;. If you would like to set a particular attribute value to true or false, please ensure that the desired attribute is a boolean and is not selected in the previous set attribute step. 
12. _Both deprovisioning flows use hardcoded SCIM attribute &quot;active&quot; which is available by default in Azure AD portal. Therefore, if you choose to Set attribute value in the Disable flow, the boolean value of &quot;active&quot; attribute will be copied to an attribute you select from the drop down list as follows:_

14. At this point you should now see the connector listed, and you can close the wizard window.

15. After configuration is complete, restart the Microsoft ECMA2Host service, if it is not running or paused, launch control panel, change to Services. In the list of services, right click on Microsoft ECMA2Host, and select either Start or Restart.

16. After a few seconds, the service should start. When it starts, for the connectors which support full import, the connector host will perform a full import. if the service does not start, or stops afterward, launch event viewer and check the log Applications and Services Log \&gt; Microsoft ECMA2Host Logs.

17. Test that you can make a request to the host by navigating to your browser and providing the endpoint for your host. If successful, you should receive a 405 message similar to the below screenshot. Please see the troubleshooting connectivity section for more details.

[https://hostname:8585/ecma2host\_connectorName/scim/users](https://hostname:8585/ecma2host_connectorName/scim/usersm)

Note: If you are using an LDAP connector, there is a virtual \&lt;dn\&gt; attribute that could be mapped to SCIM attribute for connectors that require DN calculation outside of connector logic. For example, using the Generic LDAP MA, you will be able to set up a mapping of _urn:ietf:params:scim:schemas:extension:ECMA:1.0:User:distinguishedName_ to \&lt;dn\&gt; in ECMA Connector Host Configuration Wizard and set up your custom expression in Azure AD to populate _urn:ietf:params:scim:schemas:extension:ECMA:1.0:User:distinguishedName_.

## Step 4. Configure provisioning in Azure AD
### Establish connectivity between Azure AD and the ECMA Host 

1. Check to ensure that the connector host Windows Service is running.  Click on the start menu, and type services. In the services list, scroll to &quot;Microsoft ECMA2Host&quot;.  Ensure that the status is &quot;Running&quot;.  If the status is blank, click &quot;Start&quot;.

1. Sign into Azure Portal as a global administrator in a tenant that has Azure AD Premium P1 or Premium P2 (EMS E3 or E5). This is required to be able to provision to an on-prem application. 

1. In the Azure Portal, change to the Azure Active Directory area, change to Enterprise Applications, and click on New Application.

1. Search for Provisioning Private Preview Test Application and add it to your tenant.  

1. Once the app has been created, click on the Provisioning item in the Manage section of the app. If you see an error when you click on the provisioning page, please wait and refresh.

1. Click get started.

1. Change the Provisioning Mode to Automatic. Additional settings will appear on that screen.

1. In the on-premises connectivity section, select the agent that you just deployed and click assign agent(s).

1. Before performing the next step,  **wait 10 minutes**  for the agent registration to complete. Test connection will not succeed until the agent registration is completed. You can also force the agent registration to complete by restarting the provisioning agent on your server by navigating to services, finding the provisioning agent service, and clicking restart. 

1. In the tenant URL field, enter the following URL, replacing the IP address with that of the connector host system, and adding the name of the connector to the before the &quot;/scim&quot; suffix.  For example, if your hostname is &quot;10.20.30.40&quot;, the port number is the default 8585, and the connector name is &quot;connector1&quot;, then provide the URL

In the tenant URL field, enter the following URL. Replace the hostname with the name of your VM and connectorName with the name of your connector. [https://hostname:8585/ecma2host\_connectorName/scim](https://hostname:8585/ecma2host_connectorName/scim)

1. Enter the secret token you created earlier during configuration in the field Secret Token.

1. Click Test Connection and wait one minute. If the error message &quot;We encountered an error&quot; or &quot;You appear to have entered invalid credentials&quot; appears, check the IP address or host name of the server is correct, that the server is reachable and look at the event log of the Windows Server, as described in the section Monitoring the Azure AD ECMA Connector Host service below, to see if there are any error messages.  If there are no messages, this may indicate that the firewall was unable to permit an incoming connection to the service, or that the connector name was not correctly specified.  **Please see the appendix for how to troubleshoot connectivity issues. **

1. Once the connection test is complete and the message &quot;The supplied credentials are authorized&quot; appears, click Save.

### Configure who is in scope for provisioning

Azure AD allows you to scope who should be provisioned based on assignment to an application and / or by filtering on a particular attribute. [Determine who should be in scope for provisioning](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts) and define your scoping rule as necessary. Most customers will stick with the default scope of "assigned users and groups," without doing any additional scoping configuration. 

### Assign users to your application

If you chose scoping based on assignment in the previous step, please [assign users and / or groups to your application](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users) if scoping is based on assignment to the application (recommended).

### Configure your attribute mappings
[Configure your attribute mappings.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes)

Also, you&#39;ll record the schema mappings between your connector&#39;s schema for the target system, the name of the attribute as transmitted in the SCIM protocol, and the name of the attribute in Azure AD.  Several examples are shown below.  _Note that the representation of attribute names in the Config Wizard and in the Azure AD portal is currently the same_, for extension attributes to SCIM.

| **Attribute name in target application**   | **Attribute name in SCIM**   | **Attribute name in Azure AD**   | 
| --- | --- | --- |
| Country  | urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:Country  | country  | 
| InternalGUID  |  urn:ietf:params:scim:schemas:extension:ECMA2Host:2.0:User:InternalGUID | objectId | 



1. Navigate to the provisioning page of your application.

1. Expand the Mappings section and click on Provision Azure Active Directory Users;.

1. Click on Add new mapping and add new rows for each Azure Active Directory attribute to be mapped to either a SCIM-defined attribute, or a custom attribute added at step 40.  Click OK after adding each row.

 ![](RackMultipart20210115-4-mlm6xl_html_33d9819eae0c88f0.png)

_Mapping to an application-required attributes in the Azure portal_

1. On the attribute mapping screen, click Save and click Yes to confirm.  Then close the attribute mapping page to return to the provisioning screen.

1. Change to the users and groups screen for this application and click Add user.  Select one user and assign them to the application. Ensure that they have the properties that you defined in the attribute mappings earlier.    You can retrieve those using [Graph explorer](https://developer.microsoft.com/graph/graph-explorer), however keep in mind that in Microsoft Graph, you need to ask for the attributes to be returned. Explicitly select the attributes like this:
 https://graph.microsoft.com/beta/users/abbie.spencer@fabrikamonline.com?$select=extension\_9d98ed114c4840d298fad781915f27e4\_employeeID,extension\_9d98ed114c4840d298fad781915f27e4\_division

### Test your configuration by provisioning users on demand

[Provision a user on-demand.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/provision-on-demand)

1. Navigate to the single sign-on blade and then back to the provisioning blade. From the new provisioning overview blade, click on on-demand provisioning to provision a user and test out a few users.

1. Test provisioning by provisioning a user on-demand as described [here](https://aka.ms/provisionondemanddocumentation).

1. Once on-demand provisioning is successful, change back to the provisioning configuration page. Ensure that the scope is set to only assigned users and group, turn provisioning On and click Save.

![](RackMultipart20210115-4-mlm6xl_html_cb1c08caacd4aa17.png)

_Setting the provisioning status and scope for an application in the Azure portal_

1. Wait several minutes for provisioning to start (it may take up to 40 minutes).  You can learn more about the provisioning service performance [here](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/application-provisioning-when-will-provisioning-finish-specific-user). After the provisioning job has been completed, as described in the next section, you can change the provisioning status to Off, and click Save. This will stop the provisioning service from running in the future.


### Validating user provisioning from Azure AD has been successful

Azure AD provisioning service will send a GET request with a test user to validate the tenant URL secret token and the format of the response.

Then, once the Azure AD provisioning service has been enabled, the Azure AD provisioning service will send SCIM GET, POST and PATCH operations to the connector host.   When the connector host receives one of these, it will begin an export connection, and export one or more changes to the connector.  Based on the export result code, the connector host will return success or an error indication to the Azure AD provisioning service.

For example, if a user is assigned to an application, then the Azure AD provisioning service will check if the user already exists, by querying the connector host.  If the user already exists, then Azure AD will either continue with the next user, or update that user with a PATCH with the latest attributes, if needed. However, if the user is not already present, then the Azure AD provisioning service will then send a POST request to create the user.  A POST request will be transformed by the connector host into a PutExportEntries call with an ObjectModificationType of Add, and the attributes named in the configuration of the connector.

If you are unsure if the Azure AD provisioning service has attempted to contact the connector host, start by checking the Azure AD provisioning logs..  Next, check for any  log messages on the ECMA Connector Host at the time of an update – if there are any errors or warnings they will indicate the connector host was unable to transform a request into an ECMA call, or the connector returned an exception, as described for the next section.


### Add additional users to your application
Assign additional users and groups to your application - https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/assign-user-or-group-access-portal


### Turn provisioning on
Click start to enable provisioning. The service will then synchronize all users and groups in scope. 

## Step 5. Monitor your deployment
1. Use the [provisioning logs](../reports-monitoring/concept-provisioning-logs.md) to determine which users have been provisioned successfully or unsuccessfully.
1. [Enable logging on the ECMA host](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Monitoring.md) for detailed diagnostics on-premises. 
1. Build customd alerts, dashboards, and queries using the [Azure Monitor integration] (https://docs.microsoft.com/azure/active-directory/app-provisioning/application-provisioning-log-analytics).
1. If the provisioning configuration seems to be in an unhealthy state, the application will go into quarantine. Learn more about quarantine states [here](../app-provisioning/application-provisioning-quarantine-status.md).  


## Next steps
* Review the [SQL](input link) and [LDAP](input link) specific tutorialsfor more detailed guidance


## Feedback

In you have feedback on this private preview, please report any issues at [https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyEYaD-36kVGmx4Eyyk10npUMUtDN0tLRlRVTkVOSzBGS0NDSlFYSVpLRC4u](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyEYaD-36kVGmx4Eyyk10npUMUtDN0tLRlRVTkVOSzBGS0NDSlFYSVpLRC4u) or if you have questions, please contact us at [msiga@microsoft.com](mailto:msiga@microsoft.com).


