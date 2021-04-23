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

![image](https://user-images.githubusercontent.com/36525136/111520402-4c5a9580-8715-11eb-94f6-1311b66b900e.png)



### High availability  

Microsoft currently supports active / passive high availability. You can configure more than one agent / host to connect to one application. However, you should only leave one agent active at a time, ensuring that all requests are sent to the corresponding host. When you need to failover, you can disable the active agent and enable one of the agents on stand-by. In the image below, you can see all three hosts enabled and agent 1 enabled / running. When you need to failover from agent 1 to agent 2, you can disable the agent 1 service and enable the agent 2 service. 

![image](https://user-images.githubusercontent.com/36525136/110338331-ba97ad80-7fdb-11eb-94c9-d5bfcad56689.png)

#### Workflow
1. Set up Agent 1 and Host 1 on Server 1 and configure provisioning.
1. Set up Agent 2 and Host2 on Server2.  
1. Turn the service for Agent 2 off and then assign agent 2 to the application, same as was done with agent 1 in step 1. This will add agent 2 to the same agent group to process requests the next time it is activated. Host 2 is continuously doing full imports and refreshing the cache according to the schedule defined when setting up the host. However, it is not receiving any requests as Agent 2 is turned off. The cusotmer will have an architecture that looks like the screenshot below. 
![image](https://user-images.githubusercontent.com/36525136/115930218-14064f80-a457-11eb-8013-ba2338e8f94d.png)

It's time to take Server 1 down for maintainence and failover to Server / Agent / Host 2.   
1. Disable the service for Agent 1.   
1. Check the event viewer to ensure that the last full cycle for Host 2 has a timestamp that is later than the last full cycle for Host 1 (this ensures that the cache on both hosts is in sync).  
1. Enable the service for Agent 2. 

All requests going forward are sent to A2 / H2 on S2 and customer ends up with an architecture that looks like the below: 
![image](https://user-images.githubusercontent.com/36525136/115930827-1d43ec00-a458-11eb-998e-8209f10eb893.png)

### Firewall requirements

The provisioning agents only use outbound connections to the provisioning service, which means that there is **no need to open firewall ports** for incoming connections. You also do not need a perimeter (DMZ) network because all connections are outbound and take place over a secure channel. 

## Agent best practices

- Ensure the auto Azure AD Connect Provisioning Agent Auto Update service is running. It is enabled by default when installin the agent. Auto update is **required for Microsoft to support your deployment*. 
- Place agents on different outbound connections to avoid a single point of failure. If agents use the same outbound connection, a network problem with the connection may impact all agents using it. 
- Avoid all forms of inline inspection on outbound TLS communications between agents and Azure. This type of inline inspection causes degradation to the communication flow.  
- The agent has to communicate with both Azure and your application, so the placement of the agent affects the latency of those two connections.  You can minimize the latency of the end-to-end traffic by optimizing each network connection. Each connection can be optimized by: 
  - Reducing the distance between the two ends of the hop. 
  - Choosing the right network to traverse. For example, traversing a private network rather than the public Internet may be faster, due to dedicated links. 
 - [Learn more](https://github.com/ArvindHarinder1/PrivatePreviewDocs/blob/main/Troubleshooting.md) about troubleshooting agent issues. 
