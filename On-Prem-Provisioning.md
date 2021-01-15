# Azure AD Outbound Provisioning
 with the Azure AD ECMA Connector Host
 Private Preview

# Test Lab Guide for version 1.6.83.3


 Last updated December 2020

Microsoft Confidential – Not for distribution

This guide outlines how to validate that an Extensible Connectivity Management Agent (ECMA)-based connector, that today is used with FIM or MIM, will work with the upcoming Azure AD outbound provisioning via the Azure AD ECMA Connector Host, currently in private preview.

In you have feedback on this private preview, or if you have questions, please contact us at [msiga@microsoft.com](mailto:msiga@microsoft.com).  Please do not use other Internet forums for discussing this preview, as this preview is not a generally available or supported release.

Contents

















Overview of the Azure AD outbound provisioning via Azure AD ECMA Connector Host preview

The purpose of this preview is to determine whether existing Extensible Connectivity Management Agent (ECMA) connectors implementing the [Microsoft Identity Manager (MIM)](https://docs.microsoft.com/microsoft-identity-manager/microsoft-identity-manager-2016) [Extensible Connectivity 2.2 Management Agent (ECMA2) API](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/hh859557(v=vs.100)?redirectedfrom=MSDN) can also be used for provisioning to target systems, independent of MIM.

ECMA [connectors](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/hh859557(v=vs.100)?redirectedfrom=MSDN) are used in FIM and MIM for inbound and outbound provisioning. (Azure AD Connect also uses the ECMA API internally for the generic LDAP connectors.)

In this preview, Azure AD outbound provisioning requests to create and update users are mapped into equivalent ECMA calls, by a new component, the Azure AD ECMA Connector Host.

Setting up this preview starts with your on-premises or IaaS-hosted VM, where you will set up the Azure AD ECMA Connector host with a new configuration for a connector, or import a configuration which was previously exported from a MIM sync service.  Then, you will configure in the Azure portal for Azure AD outbound provisioning to send SCIM operations to the Azure AD Connector host.

Once the setup described in this document is complete, then a user assignment to an application in Azure AD will result, shortly afterward, in an equivalent call to ECMA export APIs to provision user to the target system.  This is a similar model as with other SaaS applications.  Note that other MIM features such as password synchronization or full export are not possible with Azure AD.

![](RackMultipart20210115-4-mlm6xl_html_6f1d99f8a04ec528.png)

Note that the purpose of this preview is to verify API compatibility; it is not a supported production release.

Prerequisites for this preview

This preview requires the following in the environment:

- A target system, such as a database, or LDAP directory, in which users can be created, updated and deleted.  The ECMA connector host preview is not intended for use with production target systems, this system should hold only simulated/test data.

- An ECMA 2.0 or later connector for that target system, which supports export, schema retrieval, and optionally full import or delta import operations.  If you do not have an ECMA Connector ready during configuration, then you can still validate the end-to-end flow if you have a SQL Server in your environment and use the Generic SQL Connector.


- A Windows Server 2016 or later computer with an Internet-accessible TCP/IP address, connectivity to the target system, and with outbound connectivity to login.microsoftonline.com (for example, a Windows Server 2016 virtual machine hosted in Azure IaaS or behind a proxy). The server should have at least 3GB of RAM.

- An Azure AD tenant with Azure AD Premium P1 or Premium P2 (or EMS E3 or E5).  You can get a free developer test tenant [here](https://developer.microsoft.com/office/dev-program). The tenant where the preview is being configured cannot be located in one of the  **European Union** ,  **European Economic Area** , candidate for inclusion in the European Union,  **China** , or  **Switzerland ** country/regions.  Tenants that are deployed in Azure Government, China, or other specialized cloud are not currently available for use in this preview.  A list of country/regions that are not currently available for use in this preview as well as instructions to check the country or region that your tenant is in, is included in Appendix C.  (This restriction will be removed at a later date.)   The ECMA connector host preview is not intended for use with production target systems, this tenant should hold only simulated/test data.

- (optional) A MIM 2016 sync deployment (or FIM 2010 R2), in order to export an existing connector configuration file.

Updates and Known limitations of this preview

The September 2020 private preview build 1.6.83.0 has the following updates:

- HTTPS endpoint implemented. SSL certificate selection is now mandatory.
- Custom attributes syntax has changed to match Azure portal expression, e.g. urn:ietf:params:scim:schemas:extension:MyCustomExtensionName:2.0:MyCustomClassName:MyCustomAttribute
- Bug fixes, including handling of boolean attributes and fixes to deprovisioning flows.

This private preview build also has the following limitations and known issues.  If you experience an issue other than one of these listed below, please report it to us via the form or email address on the first page of this guide.


1. The Connector Host currently requires either SSL certificate to be trusted by Azure or the Provisioning Agent to be used. Certificate subject must match the host name ECMA Connector Host is installed on.

1. The Connector Host currently does not support anchor attribute changes (renames) or target systems which require multiple attributes to form an anchor.  \&lt;dn\&gt; rename operations are not yet possible.

1. Multi-valued attributes are not yet supported.

1. Reference attributes are not yet supported.

1. Complex anchors, e.g. ObjectTypeName+UserName, are not yet supported.

1. Provisioning passwords into on-premises applications

1. On-premises applications are sometimes not federated with Azure AD and require local passwords. The on-premises provisioning preview does not support provisioning 1-time passwords or synchronizing passwords between Azure AD and 3rd party applications. For security reasons, Microsoft is working to eliminate passwords and help customers transition to a passwordless environment.

1. User writeback from Azure AD to on-prem Azure AD is not supported

1. The Azure AD on-premises provisioning preview allows you to provision users into on-prem data stores such as an LDAP directory and a SQL database. However, this preview is  **NOT**  intended to be used to writeback a user from Azure AD to AD DS, for any domain in which Azure AD Connect has been enabled. Attempting to writeback users has the following limitations:

1. When a user is managed by Azure AD connect, the source of authority is on-prem AD. Therefore, user attributes cannot be changed in Azure AD. This preview does not change the source of authority for users managed by Azure AD Connect.

1. Using Azure AD Connect and the on-prem provisioning preview can lead to creation of a &#39;loop&#39; where Azure AD Connect can overwrite a change was made by the provisioning service in the cloud.

1. The on-prem provisioning capability can technically be used to provision to on-prem AD in the following scenarios, but is not recommended or supported:

1. The AD domain is not connected to Azure AD via Azure AD Connect. The users are created in the cloud and provisioned into AD.

1. The users that are being provisioned from AD to Azure AD are in a different OU than users being provisioned from Azure AD to AD.

1. Microsoft is working on a dedicated capability for user writeback. Please upvote the  UserVoice feedback [here](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/16887037-enable-user-writeback-to-on-premise-ad-from-azure) to track the status of the preview.

Note: as described in the Microsoft Online Services terms, located at [http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&amp;DocumentTypeId=46](http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&amp;DocumentTypeId=46), previews may employ lesser or different privacy and security measures than those typically present in the Online Services. Unless otherwise noted, Previews are not included in the SLA for the corresponding Online Service, and Customer should not use Previews to process Personal Data or other data that is subject to legal or regulatory compliance requirements.  In particular, the following terms in the section (&quot;Data Protection Terms&quot;) of the Online Services terms do not apply to Previews:  Processing of Personal Data; GDPR, Data Security, and HIPAA Business Associate.

Getting started

As you complete the following configuration steps, be sure to record the following parameters about the environment.

| **Parameter**   | **Determined in step**   | **Your value**   |
| --- | --- | --- |
| Hostname or public IP address of the Windows Server hosting the ECMA Connector Host  | 1  |   |
| Username of the account the Connector Host will run as  | 9  |   |
| Password of the account the Connector Host will run as  | 9  |   |
| Port number of the ECMA Connector Host  | Default  | 8585  |
| Connector Name  | 15  |   |
| Secret Token  | 15  |   |
| Primary key and query attribute  | 21  |   |

Also, you&#39;ll record the schema mappings between your connector&#39;s schema for the target system, the name of the attribute as transmitted in the SCIM protocol, and the name of the attribute in Azure AD.  Several examples are shown below.  _Note that the representation of attribute names in the Config Wizard and in the Azure AD portal is currently the same_, for extension attributes to SCIM.

| **Attribute name in connector**   | **Attribute name in SCIM as represented in the Config Wizard**   | **Attribute name in Azure AD custom schema mapping**   | **Attribute name in Azure AD**   |
| --- | --- | --- | --- |
| User  | urn:ietf:params:scim:schemas:
 extension:CustomExtensionName:
 2.0:CustomAttribute:User  | urn:ietf:params:scim:schemas:
extension:CustomExtensionName:
2.0:CustomAttribute:User    | employeeid  |
| displayName  | displayName  | displayName  | displayName  |
| email  | userName  | userName  | userPrincipalName  |
| ...  | ...  | ...  | ...  |



Note: If you have ECMA Connector Host previously installed:

1. Upgrade from older versions is not supported. You will need to uninstall the previous version before installing the new version. You will need to re-create all connectors in ECMA Connector Host.

1. You must also delete and re-create in Azure AD any apps used for provisioning.  The object IDs generated by earlier versions of ECMA Connector Host and stored in Azure AD are not compatible with this version of ECMA Connector Host. If you attempt to re-use an existing configuration, updates will fail.



Installing the Azure AD ECMA Connector Host

1. Prepare a Windows Server 2016 or later server with at least 3GB of RAM to run the Azure AD ECMA Connector Host.  One way to set up this server is by deploying an Azure Virtual Machine.  Note that this server requires Internet connectivity for incoming connections, either directly or via an HTTP proxy.  Record the IP address or hostname of that server.

1. Sign in as a user who is an administrator on that server.

1. Ensure Microsoft .NET 4.5.2 Framework or a later version of the .NET Framework is installed.  If using Windows Server 2016, launch Server Manager, click Add roles and features, and on the Features step of the wizard, ensure that &quot;.NET Framework 4.6&quot; is installed.

1. Create a folder on a local drive on the server that will contain the software, e.g., c:\temp.

1. Copy the ZIP file containing the Azure AD ECMA Connector Host software and unpack to that folder, so it creates a subfolder.

1. Generate a self-signed certificate with the hostname as the subject as described in the appendix.

1. Open an elevated command shell, by right clicking on Command Prompt on the Start menu and selecting &quot;Run as administrator&quot;.

1. In that elevated command shell, change to the subfolder where the software was unpacked.

cd \temp

1. Use msiexec to install the software, by typing


msiexec /log log.txt /i Microsoft.ECMA2Host.Setup.msi


1. When prompted, select a certificate to be used for HTTPS endpoint. You can use the steps outlined in the appendix to generate the certificate.

1. Provide the username and password of an account that is a local administrator. The user must be a member of the local Administrators group on the computer where the software is being installed.  If the account is a local machine account (not a domain account), provide the account name in the form &quot;hostname\username&quot;. _Otherwise, if the account is a domain account, provide the account name in the  &quot;domain\_fqdn\username&quot; form, including the full DNS domain name not just the NETBIOS name of the domain._ Then click Install, and when the install completes, click Finish.

Configuring a connector in the Azure AD ECMA Connector Host

1. After installation is complete, if you specified an account in step 9 above which is different than the account you&#39;re currently signed in as, sign out and switch to that account.

1. Change to the directory c:\program files\Microsoft ECMA2host\Service\ECMA and ensure there are one or more DLLs already present in that directory.  (Those DLLs correspond to Microsoft-delivered connectors).

cd &quot;c:\Program Files\Microsoft ECMA2Host\Service\ECMA&quot;

1. Copy the MA DLL for your connector, and any of its prerequisite DLLs, to that same ECMA subdirectory of the Service directory.   If you do not have a DLL for your connector, but have a SQL Server in your environment, then you can continue testing with the Generic SQL Connector.    Similarly, if you have an LDAP Server or PowerShell, you can use one of included Generic connectors.

Note: If you are using an LDAP connector, there is a virtual \&lt;dn\&gt; attribute that could be mapped to SCIM attribute for connectors that require DN calculation outside of connector logic. For example, using the Generic LDAP MA, you will be able to set up a mapping of _urn:ietf:params:scim:schemas:extension:ECMA:1.0:User:distinguishedName_ to \&lt;dn\&gt; in ECMA Connector Host Configuration Wizard and set up your custom expression in Azure AD to populate _urn:ietf:params:scim:schemas:extension:ECMA:1.0:User:distinguishedName_.

1. Change to the directory C:\program files\Microsoft ECMA2Host\Wizard and run as an administrator the program Microsoft.ECMA2Host.ConfigWizard.exe to set up the ECMA Connector Host configuration.

cd &quot;c:\Program Files\Microsoft ECMA2Host\Wizard&quot;
 Microsoft.ECMA2Host.ConfigWizard.exe


1. A new window will appear with a list of connectors. The first time this is run, no connector configurations will be present.  Click &quot;New Connector&quot;.

 ![](RackMultipart20210115-4-mlm6xl_html_51c835392a6b14a9.png)
_New connector properties page_

If you will be importing an existing MA configuration from MIM, then perform the steps in Appendix A before proceeding.

1. Pick and record the name of the connector, and a shared secret for use on the new connector properties page. The name of the connector should be exclusively ASCII letters and digits, the autosync timer should be 60 minutes, and the secret token a string of 10-20 ASCII letters and digits.    (You&#39;ll use the secret token later in configuring outgoing provisioning in Azure AD). For Connector, select from the drop-down the name DLL that was placed in the ECMA folder at the earlier step. Then click &quot;Next&quot;.

1. The left-hand side of the wizard window may change to add additional tabs, such as Connectivity, as required by the connector.  Click &quot;Next&quot; after completing each tab.

1. On the Partitions tab, if the connector does not have any partitions, leave the default partition selected, and click Next.

1. On the Run Profiles tab, select the run profiles which the connector is capable of being used in.  The export profile is mandatory, but full and delta import are optional.  Note that if neither full import nor delta import is configured, the ECMA Connector host will not be able to determine whether an object already exists in the target system prior to exporting it.

1. On the Export tab, specify the page size – default to 100.

1. On the Object Types tab, select which object type provided by the target system corresponds to an Azure AD user. Then, select the anchor attribute for that object type.

![](RackMultipart20210115-4-mlm6xl_html_a8b62338c0274b4d.png)_Object type and anchor attribute mapping for the Azure AD User object type_

1. On the Schema Mapping tab, configure the unique identifier attribute for the Azure AD user.  The unique identifier will correspond to the primary key attribute that will be configured later in Azure AD portal. Then, add attribute mappings for each attribute in your target system&#39;s user to the corresponding Azure AD attribute, by clicking &quot;Add attribute mapping&quot; tab, selecting the attribute name of the connected system, and the corresponding SCIM attribute.  By default, the attribute names from SCIM are displayed; if you do not see a corresponding attribute, then type in the attribute name.  In the configuration wizard, custom attributes that are not displayed in the drop down must be written in the same format as in Azure AD portal, e.g.

 urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:CustomAttribute:User

![](RackMultipart20210115-4-mlm6xl_html_f59f681d68cf102a.png)
_Schema Mappings tab for a user&#39;s attributes with a custom attribute_

1. On the Deprovisioning tab, configure the desired behavior when a user&#39;s account in Azure AD is disabled from being able to sign in, and the desired behavior when a user&#39;s account in Azure AD is deleted.  If you do not wish users to be removed from the target system, then select &quot;None&quot; for the delete flow. Then click &quot;Finish&quot;.

_Both deprovisioning flows use hardcoded SCIM attribute &quot;active&quot; which is available by default in Azure AD portal. Therefore, if you choose to Set attribute value in the Disable flow, the boolean value of &quot;active&quot; attribute will be copied to an attribute you select from the drop down list as follows:_

![](RackMultipart20210115-4-mlm6xl_html_68f5d9406d7483b8.png)

_Deprovisioning tab _

1. At this point you should now see the connector listed, and you can close the wizard window.

1. After configuration is complete, restart the Microsoft ECMA2Host service, if it is not running or paused, launch control panel, change to Services. In the list of services, right click on Microsoft ECMA2Host, and select either Start or Restart.

1. After a few seconds, the service should start. When it starts, for the connectors which support full import, the connector host will perform a full import. if the service does not start, or stops afterward, launch event viewer and check the log Applications and Services Log \&gt; Microsoft ECMA2Host Logs.

1. Test that you can make a request to the host by navigating to your browser and providing the endpoint for your host. If successful, you should receive a 405 message similar to the below screenshot. Please see the appendix for more details.

[https://hostname:8585/ecma2host\_connectorName/scim/users](https://hostname:8585/ecma2host_connectorName/scim/usersm)

![](RackMultipart20210115-4-mlm6xl_html_137c00ae78714e61.png)

Deploy the provisioning agent for connectivity

1. Download the provisioning agent, found in the zip file.

1. Install it on your virtual machine or on-premises server.

1. Select the extension to provision to on-prem applications (selecting this will skip over the step to provide AD credentials).

1. When prompted, you will need to provide credentials for a user that is a global administrator or hybrid administrator in Azure AD.

Configuring outbound provisioning in Azure AD

1. Check to ensure that the connector host Windows Service is running.  Click on the start menu, and type services. In the services list, scroll to &quot;Microsoft ECMA2Host&quot;.  Ensure that the status is &quot;Running&quot;.  If the status is blank, click &quot;Start&quot;.

1. Sign into Azure Portal as a global administrator in a tenant that has Azure AD Premium P1 or Premium P2 (EMS E3 or E5). This is required to be able to configure outbound provisioning via a non-gallery application.

1. In the Azure Portal, change to the Azure Active Directory area, change to Enterprise Applications, and click on New Application.

1. Search for &quot;Provisioning Private Preview Test Application&quot;. If the UI looks different than in the screenshot below, you can click the banner at the top of the screen to try the new preview experience.

![](RackMultipart20210115-4-mlm6xl_html_65d3bb0c9062ec2b.png)


_Adding a non-gallery application in the Azure portal_

1. Once the app has been created, click on the Provisioning item in the Manage section of the app. If you see an error when you click on the provisioning page, please wait and refresh.

1. Click get started.

1. Change the Provisioning Mode to Automatic. Additional settings will appear on that screen.

1. In the on-premises connectivity section, select the agent that you just deployed and click assign agent(s).

![](RackMultipart20210115-4-mlm6xl_html_38fc14246a9c85ce.png)

1. Assigning agents

1. Before performing the next step,  **wait about 30 minutes**  for the agent registration to complete. Test connection will not succeed until the agent registration is completed.

1. In the tenant URL field, enter the following URL, replacing the IP address with that of the connector host system, and adding the name of the connector to the before the &quot;/scim&quot; suffix.  For example, if your hostname is &quot;10.20.30.40&quot;, the port number is the default 8585, and the connector name is &quot;connector1&quot;, then provide the URL

In the tenant URL field, enter the following URL. Replace the hostname with the name of your VM and connectorName with the name of your connector. [https://hostname:8585/ecma2host\_connectorName/scim](https://hostname:8585/ecma2host_connectorName/scim)

1. Enter the secret token you created earlier during configuration in the field Secret Token.

1. Click Test Connection and wait one minute. If the error message &quot;We encountered an error&quot; or &quot;You appear to have entered invalid credentials&quot; appears, check the IP address or host name of the server is correct, that the server is reachable and look at the event log of the Windows Server, as described in the section Monitoring the Azure AD ECMA Connector Host service below, to see if there are any error messages.  If there are no messages, this may indicate that the firewall was unable to permit an incoming connection to the service, or that the connector name was not correctly specified.  **Please see the appendix for how to troubleshoot connectivity issues. **

1. Once the connection test is complete and the message &quot;The supplied credentials are authorized&quot; appears, click Save.

![](RackMultipart20210115-4-mlm6xl_html_fb1870091ae0d83f.png)

1. Expand the Mappings section and click on &quot;Provision Azure Active Directory Users&quot;.

1. Scroll to the bottom of the page, click &quot;Show advanced options&quot; and click &quot;Edit attribute list for scimOnPremises&quot;.

![](RackMultipart20210115-4-mlm6xl_html_ddd5a66aa21a3984.png)
_Advanced option for editing the list of application-required attributes in the Azure portal_

1. Add rows for each attribute which the connector host requires which is not already on the list.  Type the name of the attribute, specify the type as String, and click Add Attribute as described [here](https://docs.microsoft.com/azure/active-directory/app-provisioning/customize-application-attributes#provisioning-a-custom-extension-attribute-to-a-scim-compliant-application).  Do not use square brackets when typing the name; instead use a pattern with just colon characters as delimiters, as in

urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:CustomAttribute:User  

 ![](RackMultipart20210115-4-mlm6xl_html_c1c28f8fabf94e51.png)

_Editing the list of application-required attributes in the Azure portal (example 1)_

![](RackMultipart20210115-4-mlm6xl_html_b8487a057ad16a03.png)

_Editing the list of application-required attributes in the Azure portal (example 2)_

1. After all the attributes have been added, click Save, and return to the Attribute Mapping screen.

1.  You can delete mappings to attributes in the attribute mapping list which are not used in your connector.  For example, if you do not wish to send the mail nickname (which is not unique) to the target system, then remove the mapping of the mailNickname Azure AD attribute to the externalid customsso attribute.

![](RackMultipart20210115-4-mlm6xl_html_5b82dcb9c772fe7.png)
_Editing a mapping to an application-required attributes in the Azure portal_

1. Click on Add new mapping and add new rows for each Azure Active Directory attribute to be mapped to either a SCIM-defined attribute, or a custom attribute added at step 40.  Click OK after adding each row.

 ![](RackMultipart20210115-4-mlm6xl_html_33d9819eae0c88f0.png)

_Mapping to an application-required attributes in the Azure portal_

1. On the attribute mapping screen, click Save and click Yes to confirm.  Then close the attribute mapping page to return to the provisioning screen.

1. Change to the users and groups screen for this application and click Add user.  Select one user and assign them to the application. Ensure that they have the properties that you defined in the attribute mappings earlier.    You can retrieve those using [Graph explorer](https://developer.microsoft.com/graph/graph-explorer), however keep in mind that in Microsoft Graph, you need to ask for the attributes to be returned. Explicitly select the attributes like this:
 https://graph.microsoft.com/beta/users/abbie.spencer@fabrikamonline.com?$select=extension\_9d98ed114c4840d298fad781915f27e4\_employeeID,extension\_9d98ed114c4840d298fad781915f27e4\_division

1. Navigate to the single sign-on blade and then back to the provisioning blade. From the new provisioning overview blade, click on on-demand provisioning to provision a user and test out a few users.

1. Test provisioning by provisioning a user on-demand as described [here](https://aka.ms/provisionondemanddocumentation).

1. Once on-demand provisioning is successful, change back to the provisioning configuration page. Ensure that the scope is set to only assigned users and group, turn provisioning On and click Save.

![](RackMultipart20210115-4-mlm6xl_html_cb1c08caacd4aa17.png)

_Setting the provisioning status and scope for an application in the Azure portal_

1. Wait several minutes for provisioning to start (it may take up to 40 minutes).  You can learn more about the provisioning service performance [here](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/application-provisioning-when-will-provisioning-finish-specific-user). After the provisioning job has been completed, as described in the next section, you can change the provisioning status to Off, and click Save. This will stop the provisioning service from running in the future.

Validating user provisioning from Azure AD has been successful

Azure AD provisioning service will send a GET request with a test user to validate the tenant URL secret token and the format of the response.

Then, once the Azure AD provisioning service has been enabled, the Azure AD provisioning service will send SCIM GET, POST and PATCH operations to the connector host.   When the connector host receives one of these, it will begin an export connection, and export one or more changes to the connector.  Based on the export result code, the connector host will return success or an error indication to the Azure AD provisioning service.

For example, if a user is assigned to an application, then the Azure AD provisioning service will check if the user already exists, by querying the connector host.  If the user already exists, then Azure AD will either continue with the next user, or update that user with a PATCH with the latest attributes, if needed. However, if the user is not already present, then the Azure AD provisioning service will then send a POST request to create the user.  A POST request will be transformed by the connector host into a PutExportEntries call with an ObjectModificationType of Add, and the attributes named in the configuration of the connector.

If you are unsure if the Azure AD provisioning service has attempted to contact the connector host, start by checking the Azure AD provisioning logs..  Next, check for any  log messages on the ECMA Connector Host at the time of an update – if there are any errors or warnings they will indicate the connector host was unable to transform a request into an ECMA call, or the connector returned an exception, as described for the next section.

Monitoring the Azure AD ECMA Connector Host service

Once the ECMA Connector host schema mapping has been configured, start the service so it will listen for incoming connections, if you haven&#39;t already, as described in step 29 above.  Then, monitor for incoming requests.

1. Click on the start menu, type &quot;event viewer&quot;, and click on Event Viewer.

1. In Event Viewer, expand Applications and Services Logs, and select ECMA2Host Logs.

1. As changes are received by the connector host, events will be written to the application log.

Feedback

In you have feedback on this private preview, please report any issues at [https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyEYaD-36kVGmx4Eyyk10npUMUtDN0tLRlRVTkVOSzBGS0NDSlFYSVpLRC4u](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRyEYaD-36kVGmx4Eyyk10npUMUtDN0tLRlRVTkVOSzBGS0NDSlFYSVpLRC4u) or if you have questions, please contact us at [msiga@microsoft.com](mailto:msiga@microsoft.com).

Page Break

Appendix A: Creating and exporting a connector configuration in MIM Sync

You can import into Azure AD ECMA Connector Host a configuration for a specific connector from a FIM Sync or MIM Sync installation.  Note that the MIM Sync installation is only used for configuration, not for the ongoing synchronization from Azure AD.


If you already have MIM Sync with your ECMA connector already configured, then skip to step 10.

1. Prepare a Windows Server 2016 server, which is distinct from the server that will be used for running the Azure AD ECMA Connector Host.  This host server should either have a SQL Server 2016 database co-located, or have network connectivity to a SQL Server 2016 database.  One way to set up this server is by deploying an Azure Virtual Machine with the image &quot;SQL Server 2016 SP1 Standard on Windows Server 2016&quot;.  Note that this server does not need Internet connectivity, other than remote desktop access for setup purposes.

1. Create an account for use during the MIM Sync installation.  This can be a local account on that Windows Server.  To create a local account, launch control panel, open user accounts, and add a user account &quot;mimsync&quot;.

1. Add the account created in the previous step to the local Administrators group.

1. Give the account created earlier the ability to run a service.  Launch Local Security Policy, click on Local Policies, User Rights Assignment, and &quot;Log on as a service&quot;.  Add the account mentioned earlier.

1. Install MIM Sync on this host. If you do not have MIM Sync binaries, then you can install an evaluation by downloading the ZIP file from [https://www.microsoft.com/en-us/download/details.aspx?id=48244](https://www.microsoft.com/en-us/download/details.aspx?id=48244), mounting the ISO image, and copying the folder &quot;Synchronization Service&quot; to the Windows Server host.  Then run the setup program contained in that folder.   Note that evaluation software is time-limited and will expire, and is not intended for production use.

1. Once the installation of MIM Sync is complete, log out and log back in.

1. Install your connector on that same server as MIM Sync.  (For illustration purposes, this test lab guide will illustrate using one of the Microsoft-supplied connectors for download from [https://www.microsoft.com/en-us/download/details.aspx?id=51495](https://www.microsoft.com/en-us/download/details.aspx?id=51495) ).

1. Launch the Synchronization Service UI.  Click on &quot;Management Agents&quot;.  Click &quot;Create&quot;, and specify the connector management agent.  Be sure to select a connector management agent that is ECMA-based.

1. Give the connector a name, and configure the parameters needed to import and export data to the connector.  Be sure to configure that the connector can import and export single-valued string attributes of a user or person object type.

1. On the MIM Sync server computer, launch the Synchronization Service UI, if not already running.  Click on &quot;Management Agents&quot;.

1. Select the connector, and click &quot;Export Management Agent&quot;.  Save the XML file, as well as the DLL and related software for your connector, to the Windows Server which will be holding the ECMA Connector host.

At this point, the MIM Sync server is no longer needed.

1. Sign into the Windows Server as the account which the ECMA Connector Host will run as.

1. Change to the directory c:\program files\Microsoft ECMA2host\Service\ECMA and ensure there are one or more DLLs already present in that directory.  (Those DLLs correspond to Microsoft-delivered connectors).

1. Copy the MA DLL for your connector, and any of its prerequisite DLLs, to that same ECMA subdirectory of the Service directory.

1. Change to the directory C:\program files\Microsoft ECMA2Host\Wizard and run the program Microsoft.ECMA2Host.ConfigWizard.exe to set up the ECMA Connector Host configuration.

1. A new window will appear with a list of connectors. By default, no connectors will be present.  Click &quot;New connector&quot;.

1. Specify the management agent xml file that was exported from MIM earlier.  Continue with the configuration and schema mapping instructions from the section Configuring a connector above.

Page Break

Appendix B: Configuring the Generic SQL Connector for SQL Server

If you do not have a connector MA, but have SQL Server in your environment, then you can still validate the provisioning process, using the instructions in the Generic SQL Connector guide at [https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql) and  [https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step](https://docs.microsoft.com/en-us/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-genericsql-step-by-step)

After creating an ODBC file, you can then use that file when creating a new Connector.

Appendix C: Troubleshooting

After (during) configuration, if you wish to further debug the generic SQL or generic LDAP connectors, then you can enable the Connector specific log file, following the instructions in the PowerShell script [https://raw.githubusercontent.com/microsoft/MIMPowerShellConnectors/master/src/LyncConnector/EventLogConfig/Register-EventSource.ps1](https://raw.githubusercontent.com/microsoft/MIMPowerShellConnectors/master/src/LyncConnector/EventLogConfig/Register-EventSource.ps1) and updating the system.diagnostics section of the file c:\program files\Microsoft ECMA2Host\Service\Microsoft.ECMA2Host.Service.exe.config as follows:

\&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?\&gt;

\&lt;configuration\&gt;

    \&lt;startup\&gt;

        \&lt;supportedRuntime version=&quot;v4.0&quot; sku=&quot;.NETFramework,Version=v4.6&quot; /\&gt;

    \&lt;/startup\&gt;

    \&lt;appSettings\&gt;

      \&lt;add key=&quot;Debug&quot; value=&quot;true&quot; /\&gt;

    \&lt;/appSettings\&gt;

    \&lt;system.diagnostics\&gt;

      \&lt;sources\&gt;

    \&lt;source name=&quot;ConnectorsLog&quot; switchValue=&quot;Verbose&quot;\&gt;

          \&lt;listeners\&gt;

            \&lt;add initializeData=&quot;ConnectorsLog&quot; type=&quot;System.Diagnostics.EventLogTraceListener, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089&quot; name=&quot;ConnectorsLog&quot; traceOutputOptions=&quot;LogicalOperationStack, DateTime, Timestamp, Callstack&quot;\&gt;

              \&lt;filter type=&quot;&quot;/\&gt;

            \&lt;/add\&gt;

          \&lt;/listeners\&gt;

        \&lt;/source\&gt;

        \&lt;!-- Choose one of the following switchTrace:  Off, Error, Warning, Information, Verbose --\&gt;

        \&lt;source name=&quot;ECMA2Host&quot; switchValue=&quot;Verbose&quot;\&gt;

          \&lt;listeners\&gt;

            \&lt;add initializeData=&quot;ECMA2Host&quot; type=&quot;System.Diagnostics.EventLogTraceListener, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089&quot; name=&quot;ECMA2HostListener&quot; traceOutputOptions=&quot;LogicalOperationStack, DateTime, Timestamp, Callstack&quot; /\&gt;

            \&lt;remove name=&quot;Default&quot; /\&gt;

          \&lt;/listeners\&gt;

        \&lt;/source\&gt;

      \&lt;/sources\&gt;

    \&lt;/system.diagnostics\&gt;

  \&lt;runtime\&gt;

    \&lt;assemblyBinding xmlns=&quot;urn:schemas-microsoft-com:asm.v1&quot;\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;Microsoft.Owin&quot; publicKeyToken=&quot;31bf3856ad364e35&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;0.0.0.0-4.0.0.0&quot; newVersion=&quot;4.0.0.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;Microsoft.Owin.Hosting&quot; publicKeyToken=&quot;31bf3856ad364e35&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;0.0.0.0-4.0.0.0&quot; newVersion=&quot;4.0.0.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;System.IdentityModel.Tokens.Jwt&quot; publicKeyToken=&quot;31bf3856ad364e35&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;0.0.0.0-5.3.0.0&quot; newVersion=&quot;5.3.0.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;Newtonsoft.Json&quot; publicKeyToken=&quot;30ad4fe6b2a6aeed&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;0.0.0.0-10.0.0.0&quot; newVersion=&quot;10.0.0.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;Microsoft.MetadirectoryServicesEx&quot; publicKeyToken=&quot;31bf3856ad364e35&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;4.1.3.0-4.1.4.0&quot; newVersion=&quot;4.1.4.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

    \&lt;/assemblyBinding\&gt;

  \&lt;/runtime\&gt;

\&lt;/configuration\&gt;

If you want to troubleshoot configuration wizard related errors, then update the following sections of the file C:\Program Files\Microsoft ECMA2Host\Wizard\Microsoft.ECMA2Host.ConfigWizard.exe.config as follows:

\&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?\&gt;

\&lt;configuration\&gt;

  \&lt;configSections\&gt;

    \&lt;sectionGroup name=&quot;userSettings&quot; type=&quot;System.Configuration.UserSettingsGroup, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089&quot; \&gt;

      \&lt;section name=&quot;Microsoft.ECMA2Host.ConfigWizard.Properties.Settings&quot; type=&quot;System.Configuration.ClientSettingsSection, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089&quot; allowExeDefinition=&quot;MachineToLocalUser&quot; requirePermission=&quot;false&quot; /\&gt;

    \&lt;/sectionGroup\&gt;

  \&lt;/configSections\&gt;

  \&lt;startup\&gt;

    \&lt;supportedRuntime version=&quot;v4.0&quot; sku=&quot;.NETFramework,Version=v4.6&quot; /\&gt;

  \&lt;/startup\&gt;

  \&lt;system.diagnostics\&gt;

    \&lt;sources\&gt;

      \&lt;source name=&quot;ConnectorsLog&quot; switchValue=&quot;Verbose&quot;\&gt;

        \&lt;listeners\&gt;

          \&lt;add initializeData=&quot;ConnectorsLog&quot; type=&quot;System.Diagnostics.EventLogTraceListener, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089&quot; name=&quot;ConnectorsLog&quot; traceOutputOptions=&quot;LogicalOperationStack, DateTime, Timestamp, Callstack&quot;\&gt;

            \&lt;filter type=&quot;&quot;/\&gt;

          \&lt;/add\&gt;

        \&lt;/listeners\&gt;

      \&lt;/source\&gt;

      \&lt;!-- Choose one of the following switchTrace:  Off, Error, Warning, Information, Verbose --\&gt;

      \&lt;source name=&quot;ECMA2Host&quot; switchValue=&quot;Verbose&quot;\&gt;

        \&lt;listeners\&gt;

          \&lt;add initializeData=&quot;ECMA2Host&quot; type=&quot;System.Diagnostics.EventLogTraceListener, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089&quot; name=&quot;ECMA2HostListener&quot; traceOutputOptions=&quot;LogicalOperationStack, DateTime, Timestamp, Callstack&quot; /\&gt;

          \&lt;remove name=&quot;Default&quot; /\&gt;

        \&lt;/listeners\&gt;

      \&lt;/source\&gt;

    \&lt;/sources\&gt;

  \&lt;/system.diagnostics\&gt;

  \&lt;runtime\&gt;

    \&lt;assemblyBinding xmlns=&quot;urn:schemas-microsoft-com:asm.v1&quot;\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;Microsoft.Owin&quot; publicKeyToken=&quot;31bf3856ad364e35&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;0.0.0.0-4.0.0.0&quot; newVersion=&quot;4.0.0.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;System.IdentityModel.Tokens.Jwt&quot; publicKeyToken=&quot;31bf3856ad364e35&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;0.0.0.0-5.3.0.0&quot; newVersion=&quot;5.3.0.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;Newtonsoft.Json&quot; publicKeyToken=&quot;30ad4fe6b2a6aeed&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;0.0.0.0-10.0.0.0&quot; newVersion=&quot;10.0.0.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

      \&lt;dependentAssembly\&gt;

        \&lt;assemblyIdentity name=&quot;Microsoft.MetadirectoryServicesEx&quot; publicKeyToken=&quot;31bf3856ad364e35&quot; culture=&quot;neutral&quot; /\&gt;

        \&lt;bindingRedirect oldVersion=&quot;4.1.3.0-4.1.4.0&quot; newVersion=&quot;4.1.4.0&quot; /\&gt;

      \&lt;/dependentAssembly\&gt;

    \&lt;/assemblyBinding\&gt;

  \&lt;/runtime\&gt;

  \&lt;userSettings\&gt;

    \&lt;Microsoft.ECMA2Host.ConfigWizard.Properties.Settings\&gt;

      \&lt;setting name=&quot;DontShow&quot; serializeAs=&quot;String&quot;\&gt;

        \&lt;value\&gt;False\&lt;/value\&gt;

      \&lt;/setting\&gt;

    \&lt;/Microsoft.ECMA2Host.ConfigWizard.Properties.Settings\&gt;

  \&lt;/userSettings\&gt;

\&lt;/configuration\&gt;

Page Break

Appendix D: Country/Region for which this preview is not yet available

This preview is currently not available for tenants configured in any of the following country, region or Microsoft Online Services offering.  The country or region of a tenant can be viewed by logging into the Azure portal, switching to that tenant, and visiting the page [https://portal.azure.com/#blade/Microsoft\_AAD\_IAM/ActiveDirectoryMenuBlade/Properties](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties)

| **Country or Region or Offering where this preview is not available**   |
| --- |
| China  |
| United States - Azure Government (tenants in Azure Government cloud are not enabled for this preview)  |

Appendix E: Building a VM for testing

The instructions below demonstrate how to build a Virtual Machine for testing the ECMA host. These steps are not required and you can use your own on-prem server.

1. Navigate to  **portal.azure.com**  \&gt;  **Virtual Machines**  \&gt;  **add machine.**

1. Configure your VM with the settings as described below.

![](RackMultipart20210115-4-mlm6xl_html_457c2e1654d13ee3.png)

1. Click review + create.

1. Set auto shutdown to off.

![](RackMultipart20210115-4-mlm6xl_html_f68363aceb3c4631.png)

1. Ensure the VM is started and click connect to download the RDP file.

![](RackMultipart20210115-4-mlm6xl_html_8c2407983bcce193.png)

1. When prompted sign in with the username / password you provided when creating the VM (you may need to wait for a few minutes before it is successful).

![](RackMultipart20210115-4-mlm6xl_html_ce411b559a6886a9.png)

1. Search for the &quot;Server manager&quot; in your task bar and click &quot;Add roles and features&quot;.

1. Select Active Directory Domain Services

![](RackMultipart20210115-4-mlm6xl_html_586072240ab2e0c5.png)

1. Select .net is selected

![](RackMultipart20210115-4-mlm6xl_html_12051c25428f268c.png)

1. Click next through the rest of the wizard (It will take a few minutes for the installation to complete).

1. Click on the flag at the top left of the server manager and select &quot;promote this server to a domain controller.&quot;

1. Select &quot;add a new forest&quot; and provide a root domain name (e.g. ecmaTest.onmicrosoft.com).

1. Click next through the rest of the wizard and click install.

1. Search for &quot;Dsa.msc&quot; \&gt; right click on user \&gt; add new

1. Check the boxes &quot;password never expires&quot; and &quot;don&#39;t have to provide new password on next login&quot;

1. Search for &quot;Dsa.msc &quot; in the windows task bar.

1. Right click on user and click &quot;add new.&quot;

![](RackMultipart20210115-4-mlm6xl_html_234a381dce6f461d.png)

1. Right click the following checkboxes

1. Password never expires.

1. Don&#39;t have to provide new password on next login.

1. Add the user to the domain admins group by right clicking on the user, clicking add group, typing administrator into the search box, and clicking OK.

![](RackMultipart20210115-4-mlm6xl_html_22a92499812348e2.png)

Appendix F: Generating a self-signed certificate

Follow the steps below to create a certificate that can be used as part of the agent installation.  Note that this certificate will have only a 1-year validity period by default:

1. Identify your hostname

1. You can retrieve this by opening the command prompt, typing hostname, and hitting enter.

1. Open Powershell as an administrator and run the following command. Replace &quot;hostname&quot; with the value received from step 1.

New-SelfSignedCertificate -DnsName hostName -CertStoreLocation cert:\LocalMachine\My

1. Search for &quot;Manage computer certificates&quot; in the Windows task bar.

1. Identify the certificate you created under personal.

1. Right click and copy your certificate.

1. Navigate to the &quot;Trusted root certificate authorities&quot; folder, right click and paste the certificate.



Appendix G: Building a demo SQL environment for testing

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

Appendix H: Configuring the host using SQL

The following screenshots show you how to configure the host, if you are using the demo environment described above.

![](RackMultipart20210115-4-mlm6xl_html_8232d418efc1c061.png)

![](RackMultipart20210115-4-mlm6xl_html_a0e85cfddd4b37b5.png)

 

![](RackMultipart20210115-4-mlm6xl_html_82097051d9ac52ea.png)

 

 

![](RackMultipart20210115-4-mlm6xl_html_d2624c343f05d44d.png)

 

 

![](RackMultipart20210115-4-mlm6xl_html_3057f4282879489b.png)

 

 

![](RackMultipart20210115-4-mlm6xl_html_9c3250a0717d8552.png)

 

![](RackMultipart20210115-4-mlm6xl_html_518dfeac53735197.png)

 

 

![](RackMultipart20210115-4-mlm6xl_html_50f1b924dc423972.png)

 

![](RackMultipart20210115-4-mlm6xl_html_33434b3df4c6f8ad.gif)

 

 

![](RackMultipart20210115-4-mlm6xl_html_33434b3df4c6f8ad.gif)

![](RackMultipart20210115-4-mlm6xl_html_33434b3df4c6f8ad.gif)

 

Notice: The information contained in this document is Microsoft Confidential. This document is an overview and is intended to be utilized for informational purposes only. Microsoft makes no warranties, express or implied, in this document. Nothing in this document modifies any of the terms and conditions of Microsoft&#39;s written and signed agreements. This is not an offer and this document may be changed at any time by Microsoft.

Appendix I: Troubleshooting connectivity

Test connection is failing. How do I troubleshoot?

1. Ensure the agent and host services are running.

1. Search for services in your windows task bar and check to ensure the Microsoft Azure AD Connect provisioning agent service is running as well as the Microsoft ECMA connector host service

1. Test if a SCIM request works locally. If the request below returns a 405, you have the right endpoint and the host is receiving the request. If the request below does not return 405, you need to troubleshoot your host setup and ensure that you&#39;re using the right URL. See instructions below for the URL format. Make sure to add /users to the end of the request when testing in your browser (/users is not required when providing the URL in the cloud).

![](RackMultipart20210115-4-mlm6xl_html_33434b3df4c6f8ad.gif)

1. Ensure that the agent is active by navigating to your application in the azure portal \&gt; click on admin connectivity \&gt; click on the agent dropdown and ensuring your agent is active.

1. Check if the secret token provided is the same as the secret token on-prem (you will need to go on-prem and provide the secret token again and then copy it into the cloud).

1. Ensure that the URL provided matches the following pattern:

[https://hostname:8585/ecma2host\_connectorName/scim](https://hostname:8585/ecma2host_connectorName/scim)

1. Ensure that you are using a valid certificate. Please see Appendix F for instructions on how to create a valid certificate.

1. Restart the provisioning agent by navigating to the task bar on your VM \&gt; searching for the Microsoft Azure AD Connect provisioning agent \&gt; Right click stop and then start



Appendix J: Upgrade host to the latest version

1. Download the latest version of the host from the [SharePoint site](https://microsoft.sharepoint.com/:f:/t/AzureADIGAPreview/Et8GetPchJRNuE4MKN0wwRcB4KIsrrV3PPO5ZkyouIcpqA).

1. Navigate to control panel \&gt; programs \&gt; programs and features.

1. Find Microsoft ECMA2Host \&gt; right click \&gt; uninstall.

1. Uninstall does not work from Settings Apps and Features. You must uninstall from control panel.

![](RackMultipart20210115-4-mlm6xl_html_33434b3df4c6f8ad.gif)

1. Reboot the machine.

1. Open command prompt as an administrator to delete existing SSL certificates. Run the following commands one after the other.

netsh http delete sslcert ipport=0.0.0.0:8585

netsh http delete urlacl url= https://\*:8585/

1. Open command prompt and navigate to the folder where the host installer is located. If the folder is in the desktop, you will need to use a command like this (ensure the values with your file path):

cd C:\Users\Jane\Desktop\Azure AD ECMA Connector Host Private Preview 1.6.83.3\Azure AD ECMA Connector Host

1. Run the following command. Replace &quot;Microsoft.ECMA2Host.Setup.msi&quot; with the name of the installed in your folder.

msiexec /log log.txt /i Microsoft.ECMA2Host.Setup.msi
