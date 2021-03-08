---
title: How to monitor the ECMA connector host
description: This document describes how to monitor the ECMA connector host for on-premises app provisioning.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 08/31/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
---

Azure AD allows you monitor the provisioning service in the cloud as well as collect logs on-premises. The provisioning service emits logs for each user that was evaluated as part of the synchronization process. Those logs can be consumed through the Azure Portal UI, APIs, and log analytics. In addition, the ECMA host generates logs on-premises, showing each provisioning request received and the response sent to Azure AD.

# How to monitor the ECMA connector host for on-premises app provisioning

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

Requests made by Azure AD to the provisioning agent and connector host use the SCIM protocol. Requests made from the host to apps use the protocol the app support and the requests from the host to agent to azure ad rely on SCIM. You can learn more about our SCIM implementation here.  

Be aware that at the beginning of each provisioning cycle, before performing on-demand provisioning, and when doing the test connection the Azure AD provisioning service generally makes a get user call for a dummy user to ensure the target endpoint is available and returning SCIM compliant responses. 

The request generally looks like this:  

![Call](./media/how-to-monitor-ecma-connector-host/call1.png) 

And the expected response looks like this: 

![Response](./media/how-to-monitor-ecma-connector-host/response1.png) 

When querying for a user, we use the matching attribute. If the matching attribute is userName, the request will look like the above.

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

## Next Steps

Once the ECMA Connector host schema mapping has been configured, start the service so it will listen for incoming connections, if you haven&#39;t already, as described in step 29 above.  Then, monitor for incoming requests.

1. Click on the start menu, type &quot;event viewer&quot;, and click on Event Viewer.

1. In Event Viewer, expand Applications and Services Logs, and select ECMA2Host Logs.

1. As changes are received by the connector host, events will be written to the application log.


