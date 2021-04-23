# Ignore below
## Agents and Agent Groups 

Provisioning on-premises relies on the concept of agents and agent groups. Each agent is assigned to one or more agent groups and each app is assigned to one agent group.

![Agent](.\media\on-prem-app-prov-arch\agents1.png)

### Creating a new agent group 
 
```
POST ~/onPremisesPublishingProfiles/{publishingType}/agentGroups/{id}/agents 
{ 
    "displayName": "New Group" 
} 
```


### Adding an agent to an agent group 



### Removing an agent from an agent group 

```
DELETE /onPremisesPublishingProfiles/{publishingType}/agents/{id1}/agentGroups/{id2}/$ref 
```


### Deleting an agent group 

```
DELETE https://graph.microsoft.com/beta/onPremisesPublishingProfiles/provisioning/agentGroups/8832388F-3814-4952-B288-FFB62081FE25/ 
```

## Provisioning Agent with a proxy server for outbound HTTP communication

The Provisioning Agent supports use of outbound proxy. You can configure it by editing the agent config file **C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config**. 

Add the following lines into it, towards the end of the file just before the closing `</configuration>` tag. 

Replace the variables [proxy-server] and [proxy-port] with your proxy server name and port values. 

```xml 

    <system.net> 

          <defaultProxy enabled="true" useDefaultCredentials="true"> 

             <proxy 

                usesystemdefault="true" 

                proxyaddress="http://[proxy-server]:[proxy-port]" 

                bypassonlocal="true" 

             /> 

         </defaultProxy> 

    </system.net> 

``` 



## Next steps 


## Appendix F: Generating a self-signed certificate 

Navigating the the setting page of the ECMA Host will gnerate the self-signed certificate. You can also follow the following steps to manually generate a certificate. Note that this certificate will have only a 1-year validity period by default: 

Identify your hostname 

You can retrieve this by opening the command prompt, typing hostname, and hitting enter. 

Open Powershell as an administrator and run the following command. Replace “hostname” with the value received from step 1. 

 

New-SelfSignedCertificate -DnsName hostName -CertStoreLocation cert:\LocalMachine\My 

 

Search for “Manage computer certificates” in the Windows task bar. 

Identify the certificate you created under personal. 

Right click and copy your certificate.  

Navigate to the “Trusted root certificate authorities” folder, right click and paste the certificate.  

## ECMA Tutorial
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

