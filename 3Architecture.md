# On-premises application provisioning architecture

There are three primary components to provisioning users into an on-premises application.

1. The **Provisioning agent** provides connectivity between Azure AD and your on-premises environment or virtual machine.
1. The **ECMA host** converts provisioning requests from Azure AD to requests made to your target application. It serves as a gateway between Azure AD and your application. It allows you to import existing ECMA2 connectors used with Microsoft Identity Manager. Note, the ECMA host is not required if you have built a SCIM application or SCIM gateway.
1. The **Azure AD provisioning service** serves as the synchronization engine.

Note: MIM Sync is not required. However, you can use MIM sync to build and test your ECMA connector before importing it into the ECMA host. 

The following diagram shows how the three components interact:

![image](https://user-images.githubusercontent.com/36525136/116119808-d2f78080-a68c-11eb-9bc9-659db9552ebe.png)

### Firewall requirements

The provisioning agents only use outbound connections to the provisioning service, which means that there is **no need to open firewall ports** for incoming connections. You also do not need a perimeter (DMZ) network because all connections are outbound and take place over a secure channel. 

## Agent best practices

- Ensure the auto Azure AD Connect Provisioning Agent Auto Update service is running. It is enabled by default when installing the agent. Auto update is **required for Microsoft to support your deployment**. 
- Avoid all forms of inline inspection on outbound TLS communications between agents and Azure. This type of inline inspection causes degradation to the communication flow.  
- The agent has to communicate with both Azure and your application, so the placement of the agent affects the latency of those two connections.  You can minimize the latency of the end-to-end traffic by optimizing each network connection. Each connection can be optimized by: 
  - Reducing the distance between the two ends of the hop. 
  - Choosing the right network to traverse. For example, traversing a private network rather than the public Internet may be faster, due to dedicated links. 
 - [Learn more](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Troubleshooting.md) about troubleshooting agent issues. 

### Provisioning Agent questions

#### What is the GA version of the Provisioning Agent?

Refer to [Azure AD Connect Provisioning Agent: Version release history](../app-provisioning/provisioning-agent-release-version-history.md) for the latest GA version of the Provisioning Agent.  

#### How do I know the version of my Provisioning Agent?

* Sign in to the Windows server where the Provisioning Agent is installed.
* Go to **Control Panel** -> **Uninstall or Change a Program** menu
* Look for the version corresponding to the entry **Microsoft Azure AD Connect Provisioning Agent**

  >[!div class="mx-imgBorder"]
  >![Azure portal](./media/workday-inbound-tutorial/pa_version.png)

#### Does Microsoft automatically push Provisioning Agent updates?

Yes, Microsoft automatically updates the provisioning agent if the  Windows service **Microsoft Azure AD Connect Agent Updater** is up and running. Ensuring that your agent is up to date is required for support to troubleshoot issues. 

#### Can I install the Provisioning Agent on the same server running Azure AD Connect or Microsoft Identity Manager (MIM)?

Yes, you can install the Provisioning Agent on the same server that runs Azure AD Connect or MIM, but they are not required.

#### How do I configure the Provisioning Agent to use a proxy server for outbound HTTP communication?

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

#### How do I ensure that the Provisioning Agent is able to communicate with the Azure AD tenant and no firewalls are blocking ports required by the agent?

You can also check whether all of the [required ports](../app-proxy/application-proxy-add-on-premises-application.md#open-ports) are open.


#### How do I uninstall the Provisioning Agent?

* Sign in to the Windows server where the Provisioning Agent is installed.
* Go to **Control Panel** -> **Uninstall or Change a Program** menu
* Uninstall the following programs:
  * Microsoft Azure AD Connect Provisioning Agent
  * Microsoft Azure AD Connect Agent Updater
  * Microsoft Azure AD Connect Provisioning Agent Package
