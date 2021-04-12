## Test connection is failing. How do I troubleshoot?

1. Verify that the agent is running:
   1. On the server with the agent installed, open **Services** by either navigating to it or by going to **Start** > **Run** > **Services.msc**.
   1. Under **Services**, make sure **Microsoft Azure AD Connect Agent Updater** and **Microsoft Azure AD Connect Provisioning Agent** are there and their status is *Running*.

![image](https://user-images.githubusercontent.com/36525136/110372252-eaa67700-8002-11eb-968e-a6ea61a435ea.png)

1. Test if a SCIM request works locally. If the request below returns a 405, you have the right endpoint and the host is receiving the request. If the request below does not return 405, you need to troubleshoot your host setup and ensure that you&#39;re using the right URL. See instructions below for the URL format. Make sure to add /users to the end of the request when testing in your browser (/users is not required when providing the URL in the cloud).

![](RackMultipart20210115-4-mlm6xl_html_33434b3df4c6f8ad.gif)

1. Ensure that the agent is active by navigating to your application in the azure portal \&gt; click on admin connectivity \&gt; click on the agent dropdown and ensuring your agent is active.

1. Check if the secret token provided is the same as the secret token on-prem (you will need to go on-prem and provide the secret token again and then copy it into the cloud).

1. Ensure that the URL provided matches the following pattern:

[https://hostname:8585/ecma2host\_connectorName/scim](https://hostname:8585/ecma2host_connectorName/scim)

1. Ensure that you are using a valid certificate. Please see Appendix F for instructions on how to create a valid certificate.

1. Restart the provisioning agent by navigating to the task bar on your VM \&gt; searching for the Microsoft Azure AD Connect provisioning agent \&gt; Right click stop and then start


## How do I troubleshoot configuration of the ECMA host? 

#### The following issues can be resolved by running the ECMA host as an admin:

* I get an error when opening the ECMA host wizard 
![image](https://user-images.githubusercontent.com/36525136/114464200-4af87d80-9b9a-11eb-96f8-f05bebaa0113.png)

* I've been able to configure the ECMA host wizard, but am not able to start the ECMA host service
![image](https://user-images.githubusercontent.com/36525136/114463488-58613800-9b99-11eb-85d8-e0b598a37995.png)

* I'm unabled to start the ECMA host service
![image](https://user-images.githubusercontent.com/36525136/114463772-bbeb6580-9b99-11eb-9252-9a3934aa83a1.png)


#### Test connector in MIM...


##### Reviewing events in the event viewer

Once the ECMA Connector host schema mapping has been configured, start the service so it will listen for incoming connections.  Then, monitor for incoming requests. To do this, do the following:

  1. Click on the start menu, type **event viewer**, and click on Event Viewer. 
  2. In **Event Viewer**, expand **Applications and Services** Logs, and select **Microsoft ECMA2Host Logs**.     
  3. As changes are received by the connector host, events will be written to the application log. 

|Xpath|Example|
|-----|-----|
|Xpath for an event that contains a particular user|input sample expression|
|Xpath expression for filtering on a certain date-time range|input sample expression| 
|Exporting the xpath events for support|input sample expression| 


## Understanding incoming SCIM requests 

Requests made by Azure AD to the provisioning agent and connector host use the SCIM protocol. Requests made from the host to apps use the protocol the app support and the requests from the host to agent to azure ad rely on SCIM. You can learn more about our SCIM implementation [here](https://docs.microsoft.com/azure/active-directory/app-provisioning/use-scim-to-provision-users-and-groups).  

Be aware that at the beginning of each provisioning cycle, before performing on-demand provisioning, and when doing the test connection the Azure AD provisioning service generally makes a get user call for a [dummy user](https://docs.microsoft.com/azure/active-directory/app-provisioning/use-scim-to-provision-users-and-groups#request-3) to ensure the target endpoint is available and returning SCIM compliant responses. 



#### l
After (during) configuration, if you wish to further debug the generic SQL or generic LDAP connectors, then you can enable the Connector specific log file, following the instructions in the PowerShell script [https://raw.githubusercontent.com/microsoft/MIMPowerShellConnectors/master/src/LyncConnector/EventLogConfig/Register-EventSource.ps1](https://raw.githubusercontent.com/microsoft/MIMPowerShellConnectors/master/src/LyncConnector/EventLogConfig/Register-EventSource.ps1) and updating the system.diagnostics section of the file c:\program files\Microsoft ECMA2Host\Service\Microsoft.ECMA2Host.Service.exe.config as follows:

## Turning on verbose logging 

If you wish to further debug the generic SQL or generic LDAP connectors, then you can enable the Connector specific log file, following the instructions in the PowerShell script https://raw.githubusercontent.com/microsoft/MIMPowerShellConnectors/master/src/LyncConnector/EventLogConfig/Register-EventSource.ps1 and updating the system.diagnostics section of the file c:\program files\Microsoft ECMA2Host\Service\Microsoft.ECMA2Host.Service.exe.config as follows:

```
      <add key="WriteVerboseLog" value="true" /> 
      </appSettings>  
<system.diagnostics> 
       <sources> 
       <source name="ConnectorsLog" switchValue="Verbose"> 
         <listeners> 
           <add initializeData="ConnectorsLog" type="System.Diagnostics.EventLogTraceListener, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" name="ConnectorsLog" traceOutputOptions="LogicalOperationStack, DateTime, Timestamp, Callstack"> 
             <filter type=""/> 
           </add> 
         </listeners> 
       </source> 
         <source name="Microsoft ECMA2Host" switchValue="Verbose"> 
           <listeners> 
             <add initializeData="Microsoft ECMA2Host" type="System.Diagnostics.EventLogTraceListener, System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" name="ConnectorsLogListener" traceOutputOptions="LogicalOperationStack, DateTime, Timestamp, Callstack" /> 
             <remove name="Default" /> 
           </listeners> 
         </source> 
       </sources> 
     </system.diagnostics> 
```

## How do I troubleshoot the provisioning agent?
#### Verify the port
Verify that Azure is listening on port 443 and that your agent can communicate with it.
![image](https://user-images.githubusercontent.com/36525136/110371023-44a63d00-8001-11eb-9dfc-83407452c991.png)


#### Agent failed to start

You might receive an error message that states:

**Service 'Microsoft Azure AD Connect Provisioning Agent' failed to start. Verify that you have sufficient privileges to start the system services.** 

This problem is typically caused by a group policy that prevented permissions from being applied to the local NT Service log-on account created by the installer (NT SERVICE\AADConnectProvisioningAgent). These permissions are required to start the service.

To resolve this problem, follow these steps.

1. Sign in to the server with an administrator account.
1. Open **Services** by either navigating to it or by going to **Start** > **Run** > **Services.msc**.
1. Under **Services**, double-click **Microsoft Azure AD Connect Provisioning Agent**.
1. On the **Log On** tab, change **This account** to a domain admin. Then restart the service. 

This test verifies that your agents can communicate with Azure over port 443. Open a browser, and go to the previous URL from the server where the agent is installed.

#### Agent times out or certificate is invalid

You might get the following error message when you attempt to register the agent.

![image](https://user-images.githubusercontent.com/36525136/110371601-0eb58880-8002-11eb-8594-bb95ce2ebf92.png)

This problem is usually caused by the agent being unable to connect to the Hybrid Identity Service and requires you to configure an HTTP proxy. To resolve this problem, configure an outbound proxy. 

The provisioning agent supports use of an outbound proxy. You can configure it by editing the agent config file *C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config*.
Add the following lines into it, toward the end of the file just before the closing `</configuration>` tag.
Replace the variables `[proxy-server]` and `[proxy-port]` with your proxy server name and port values.

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
#### Agent registration fails with security error

You might get an error message when you install the cloud provisioning agent.

This problem is typically caused by the agent being unable to execute the PowerShell registration scripts due to local PowerShell execution policies.

To resolve this problem, change the PowerShell execution policies on the server. You need to have Machine and User policies set as *Undefined* or *RemoteSigned*. If they're set as *Unrestricted*, you'll see this error. For more information, see [PowerShell execution policies](/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-6). 

### Log files

By default, the agent emits minimal error messages and stack trace information. You can find these trace logs in the folder **C:\ProgramData\Microsoft\Azure AD Connect Provisioning Agent\Trace**.

To gather additional details for troubleshooting agent-related problems, follow these steps.

1.  Install the AADCloudSyncTools PowerShell module as described [here](reference-powershell.md#install-the-aadcloudsynctools-powershell-module).
2. Use the `Export-AADCloudSyncToolsLogs` PowerShell cmdlet to capture the information.  You can use the following switches to fine tune your data collection.
      - SkipVerboseTrace to only export current logs without capturing verbose logs (default = false)
      - TracingDurationMins to specify a different capture duration (default = 3 mins)
      - OutputPath to specify a different output path (default = User’s Documents)


---------------------



Azure AD allows you monitor the provisioning service in the cloud as well as collect logs on-premises. The provisioning service emits logs for each user that was evaluated as part of the synchronization process. Those logs can be consumed through the [Azure Portal UI, APIs, and log analytics](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs#ways-of-interacting-with-the-provisioning-logs). In addition, the ECMA host generates logs on-premises, showing each provisioning request received and the response sent to Azure AD.


## Next Steps

Once the ECMA Connector host schema mapping has been configured, start the service so it will listen for incoming connections, if you haven&#39;t already, as described in step 29 above.  Then, monitor for incoming requests.

1. Click on the start menu, type &quot;event viewer&quot;, and click on Event Viewer.

1. In Event Viewer, expand Applications and Services Logs, and select ECMA2Host Logs.

1. As changes are received by the connector host, events will be written to the application log.



