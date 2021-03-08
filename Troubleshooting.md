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
#### Test connector in MIM...

#### l
After (during) configuration, if you wish to further debug the generic SQL or generic LDAP connectors, then you can enable the Connector specific log file, following the instructions in the PowerShell script [https://raw.githubusercontent.com/microsoft/MIMPowerShellConnectors/master/src/LyncConnector/EventLogConfig/Register-EventSource.ps1](https://raw.githubusercontent.com/microsoft/MIMPowerShellConnectors/master/src/LyncConnector/EventLogConfig/Register-EventSource.ps1) and updating the system.diagnostics section of the file c:\program files\Microsoft ECMA2Host\Service\Microsoft.ECMA2Host.Service.exe.config as follows:

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


