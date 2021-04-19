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


Appendix F: Generating a self-signed certificate 

Navigating the the setting page of the ECMA Host will gnerate the self-signed certificate. You can also follow the following steps to manually generate a certificate. Note that this certificate will have only a 1-year validity period by default: 

Identify your hostname 

You can retrieve this by opening the command prompt, typing hostname, and hitting enter. 

Open Powershell as an administrator and run the following command. Replace “hostname” with the value received from step 1. 

 

New-SelfSignedCertificate -DnsName hostName -CertStoreLocation cert:\LocalMachine\My 

 

Search for “Manage computer certificates” in the Windows task bar. 

Identify the certificate you created under personal. 

Right click and copy your certificate.  

Navigate to the “Trusted root certificate authorities” folder, right click and paste the certificate.  

 