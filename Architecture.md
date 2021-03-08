---
title: 'On-premises application provisioning architecture | Microsoft Docs'
description: Describes overview of on-premises application provisioning architecture.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 08/31/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
---

# On-premises application provisioning architecture

There are three primary components to provisioning users into an on-premises application.

1. The **Provisioning agent** provides connectivity between Azure AD and your on-premises environment or virtual machine.
1. The **ECMA host** converts provisioning requests from Azure AD to requests made to your target application. It serves as a gateway between Azure AD and your application. It allows you to import existing ECMA2 connectors used with Microsoft Identity Manager. Note, the ECMA host is not required if you have built a SCIM application or SCIM gateway.
1. The **Azure AD provisioning service** serves as the synchronization engine.

Note: MIM Sync is not required. However, you can use MIM sync to build and test your ECMA connector before importing it into the ECMA host. 

The following diagram shows how the three components interact:

![image](https://user-images.githubusercontent.com/36525136/110339150-aef8b680-7fdc-11eb-8ddf-20e64573dfa2.png)



### High availability  

Microsoft currently supports active / passive high availability. You can configure more than one agent / host to connect to one application. However, you should only leave one agent active at a time, ensuring that all requests are sent to the corresponding host. When you need to failover, you can disable the active agent and enable one of the agents on stand-by. In the image below, you can see all three hosts enabled and agent 1 enabled / running. When you need to failover from agent 1 to agent 2, you can disable the agent 1 service and enable the agent 2 service. 

![image](https://user-images.githubusercontent.com/36525136/110338331-ba97ad80-7fdb-11eb-94c9-d5bfcad56689.png)

### Firewall requirements

The provisioning agents only use outbound connections to the provisioning service, which means that there is **no need to open firewall ports** for incoming connections. You also do not need a perimeter (DMZ) network because all connections are outbound and take place over a secure channel. 

## Agent best practices

- Ensure the auto update service is running. Auto update is required for Microsoft to support your deployment. 
- Place agents on different outbound connections to avoid a single point of failure. If agents use the same outbound connection, a network problem with the connection may impact all agents using it. 
- Avoid all forms of inline inspection on outbound TLS communications between agents and Azure. This type of inline inspection causes degradation to the communication flow. 
- Make sure to keep automatic updates running for your agents. This is required for Microsoft to support your configuration.  
- The agent has to communicate with both Azure and your application, so the placement of the agent affects the latency of those two connections.  You can minimize the latency of the end-to-end traffic by optimizing each network connection. Each connection can be optimized by: 
  - Reducing the distance between the two ends of the hop. 
  - Choosing the right network to traverse. For example, traversing a private network rather than the public Internet may be faster, due to dedicated links. 
 - [Learn more](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Troubleshooting.md) about troubleshooting agent issues. 

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
