---
title: Generic LDAP Connector | Microsoft Docs
description: This article describes how to configure Microsoft's Generic LDAP Connector.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 984beeb0-4d91-4908-ad81-c19797c4891b
ms.reviewer: davidste
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.prod: microsoft-identity-manager
ms.date: 03/01/2020
ms.author: billmath
---

# Generic LDAP Connector technical reference

This tutorial describes the steps you need to perform to automatically provision and deprovision users from Azure AD into an [LDAP v3 server](https://LinkToSectionBelowWithFullList) (e.g. Apache DS, OpenLDAP, or ODSEE). This tutorial should not be used to [write users into Active Directory](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/16887037-enable-user-writeback-to-on-premise-ad-from-azure). Click [here](https://sectionLaterInTheDoc) for a list of tested LDAP directories. For important details on what this service does, how it works, and frequently asked questions, see [Automate user provisioning and deprovisioning to SaaS applications with Azure Active Directory](../app-provisioning/user-provisioning.md).

## Capabilities Supported
> [!div class="checklist"]
> * Create users in a LDAP directory
> * Remove users from a LDAP directory when they do not require access anymore
> * Keep user attributes synchronized between Azure AD and the LDAP directory
> * Discover schema from the LDAP directory (RFC3673 and RFC4512/4.2). Supports structural classes, aux classes, and extensibleObject object class (RFC4512/4.3)
> * Full import and export are supported for all directories. Delta imports are supported for [specified directories]().

> [!Note]
> Notable known directories or features not supported: Microsoft Active Directory Domain Services (AD DS), Password Change Notification, Service(PCNS), Exchange provisioning, Delete of Active Sync Devices,Support for TDescurityDescriptor,Oracle Internet Directory (OID)

## Prerequisites
Before you use the Connector, make sure you have the following on the synchronization server:

* Microsoft .NET 4.5.2 Framework or later

* The connector uses the port number specified in the configuration, which by default is 389 for LDAP and 636 for LDAPS. For LDAPS, you must use SSL 3.0 or TLS. SSL 2.0 is not supported and cannot be activated.

## Step 1. Plan your provisioning deployment
1. Review the steps to configure [on-prem provisioning](https://linkToNewTutorial) 
2. Learn about [how the provisioning service works](../app-provisioning/user-provisioning.md).
3. Determine who will be in [scope for provisioning](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
4. Determine what data to [map between Azure AD and your LDAP directory](../app-provisioning/customize-application-attributes.md). 

## Step 2. Download and install the provisioning agent and on-prem host

You will need to download the provisioning agent and ECMA connector host to provide connectivity to your application. [Review the detailed steps for downloading and installing the components.]()

## Step 3. Configure the host
### Connectivity
#### Detecting the LDAP server
The Connector relies upon various techniques to detect and identify the LDAP server. The Connector uses the Root DSE, vendor name/version, and it inspects the schema to find unique objects and attributes known to exist in certain LDAP servers. This data, if found, is used to pre-populate the configuration options in the Connector.

#### Connected Data Source permissions
To perform import and export operations on the objects in the connected directory, the connector account must have sufficient permissions. The connector needs write permissions to be able to export, and read permissions to be able to import. Permission configuration is performed within the management experiences of the target directory itself. On the Connectivity page, you must specify the Host, Port, and Binding information. Depending on which Binding is selected, additional information might be supplied in the following sections.

![Connectivity](./media/microsoft-identity-manager-2016-connector-genericldap/connectivity.png)

* The Connection Timeout setting is only used for the first connection to the server when detecting the schema.
* If Binding is Anonymous, then neither username / password nor certificate are used.
* For other bindings, enter information either in username / password or select a certificate.
* If you are using Kerberos to authenticate, then also provide the Realm/Domain of the user.

The **attribute aliases** text box is used for attributes defined in the schema with RFC4522 syntax. These attributes cannot be detected during schema detection and the Connector needs help to identify those attributes. For example the following must be entered in the attribute aliases box to correctly identify the userCertificate attribute as a binary attribute:

`userCertificate;binary`

The following is an example for how this configuration could look like:

![Connectivity](./media/microsoft-identity-manager-2016-connector-genericldap/connectivityattributes.png)

Select the **include operational attributes in schema** checkbox to also include attributes created by the server. These include attributes such as when the object was created and last update time.

Select **Include extensible attributes in schema** if extensible objects (RFC4512/4.3) are used and enabling this option allows every attribute to be used on all object. Selecting this option makes the schema very large so unless the connected directory is using this feature the recommendation is to keep the option unselected.

### Global Parameters
On the Global Parameters page, you configure the DN to the delta change log and additional LDAP features. The page is pre-populated with the information provided by the LDAP server.

![Connectivity](./media/microsoft-identity-manager-2016-connector-genericldap/globalparameters.png)

The top section shows information provided by the server itself, such as the name of the server. The Connector also verifies that the mandatory controls are present in the Root DSE. If these controls are not listed, a warning is presented. Some LDAP directories do not list all features in the Root DSE and it is possible that the Connector works without issues even if a warning is present.

The **supported controls** checkboxes control the behavior for certain operations:

* With tree delete selected, a hierarchy is deleted with one LDAP call. With tree delete unselected, the connector does a recursive delete if needed.
* With paged results selected, the Connector does a paged import with the size specified on the run steps.
* The VLVControl and SortControl is an alternative to the pagedResultsControl to read data from the LDAP directory.
* If all three options (pagedResultsControl, VLVControl, and SortControl) are unselected then the Connector imports all object in one operation, which might fail if it is a large directory.
* ShowDeletedControl is only used when the Delta import method is USNChanged.

The change log DN is the naming context used by the delta change log, for example **cn=changelog**. This value must be specified to be able to do delta import.

The following is a list of default change log DNs:

| Directory | Delta change log |
| --- | --- |
| Microsoft AD LDS and AD GC |Automatically detected. USNChanged. |
| Apache Directory Server |Not available. |
| Directory 389 |Change log. Default value to use: **cn=changelog** |
| IBM Tivoli DS |Change log. Default value to use: **cn=changelog** |
| Isode Directory |Change log. Default value to use: **cn=changelog** |
| Novell/NetIQ eDirectory |Not available. TimeStamp. The Connector uses last updated date/time to get added and updated records. |
| Open DJ/DS |Change log.  Default value to use: **cn=changelog** |
| Open LDAP |Access log. Default value to use: **cn=accesslog** |
| Oracle DSEE |Change log. Default value to use: **cn=changelog** |
| RadiantOne VDS |Virtual directory. Depends on the directory connected to VDS. |
| Sun One Directory Server |Change log. Default value to use: **cn=changelog** |

The password attribute is the name of the attribute the Connector should use to set the password in password change and password set operations.
This value is by default set to **userPassword** but can be changed when needed for a particular LDAP system.

In the additional partitions list, it is possible to add additional namespaces not automatically detected. For example, this setting can be used if several servers make up a logical cluster, which should all be imported at the same time. Just as Active Directory can have multiple domains in one forest but all domains share one schema, the same can be simulated by entering the additional namespaces in this box. Each namespace can import from different servers and is further configured on the Configure Partitions and Hierarchies page. Use Ctrl+Enter to get a new line.

### Configure Provisioning Hierarchy
This page is used to map the DN component, for example OU, to the object type that should be provisioned, for example organizationalUnit.

![Provisioning Hierarchy](./media/microsoft-identity-manager-2016-connector-genericldap/provisioninghierarchy.png)

By configuring provisioning hierarchy, you can configure the Connector to automatically create a structure when needed. For example, if there is a namespace dc=contoso,dc=com and a new object cn=Joe, ou=Seattle, c=US, dc=contoso, dc=com is provisioned, then the Connector can create an object of type country for US and an organizationalUnit for Seattle if those are not already present in the directory.

### Configure Partitions and Hierarchies
On the partitions and hierarchies page, select all namespaces with objects you plan to import and export.

![Partitions](./media/microsoft-identity-manager-2016-connector-genericldap/partitions.png)

For each namespace, it is also possible to configure connectivity settings that would override the values specified on the Connectivity screen. If these values are left to their default blank value, the information from the Connectivity screen is used.

It is also possible to select which containers and OUs the Connector should import from and export to.

When performing a search this is done across all containers in the partition. In cases where there are large numbers of containers this behavior leads to performance degradation.

> [!NOTE]
> Starting in the March 2017 update to the Generic LDAP connector searches can be limited in scope to only the selected containers. This can be done by selecting the checkbox 'Search only in selected containers' as shown in the image below.

![Search only selected containers](./media/microsoft-identity-manager-2016-connector-genericldap/partitions-only-selected-containers.png)

### Configure Anchors
This page does always have a preconfigured value and cannot be changed. If the server vendor has been identified, then the anchor might be populated with an immutable attribute, for example the GUID for an object. If it has not been detected or is known to not have an immutable attribute, then the connector uses dn (distinguished name) as the anchor.

![anchors](./media/microsoft-identity-manager-2016-connector-genericldap/anchors.png)


The following is a list of LDAP servers and the anchor being used:

| Directory | Anchor attribute |
| --- | --- |
| Microsoft AD LDS and AD GC |objectGUID |
| 389 Directory Server |dn |
| Apache Directory |dn |
| IBM Tivoli DS |dn |
| Isode Directory |dn |
| Novell/NetIQ eDirectory |GUID |
| Open DJ/DS |dn |
| Open LDAP |dn |
| Oracle ODSEE |dn |
| RadiantOne VDS |dn |
| Sun One Directory Server |dn |

## References
This section provides information of aspects that are specific to this Connector or for other reasons are important to know.

#### Tested LDAP servers
The Connector is supported with all LDAP v3 servers (RFC 4510 compliant). It has been tested with the following: 
* Microsoft Active Directory Lightweight Directory Services (AD LDS)
* Microsoft Active Directory Global Catalog (AD GC)
* 389 Directory Server
* Apache Directory Server
* IBM Tivoli DS
* Isode Directory
* NetIQ eDirectory
* Novell eDirectory
* Open DJ
* Open DS
* Open LDAP (openldap.org)
* Oracle (previously Sun) Directory Server Enterprise Edition
* RadiantOne Virtual Directory Server (VDS)
* Sun One Directory Server

#### Delta import
* The delta watermark in Open LDAP is UTC date/time. For this reason, the clocks between FIM Synchronization Service and the Open LDAP must be synchronized. If not, some entries in the delta change log might be omitted.For directories with a delta change log that is based on date/time, it is highly recommended to run a full import at periodic times. This process allows the sync engine to find and dissimilarities between the LDAP server and what is currently in the connector space.

* Delta import is only available when a support directory has been detected. The following methods are currently used:

  * LDAP Accesslog. See [http://www.openldap.org/doc/admin24/overlays.html#Access Logging](http://www.openldap.org/doc/admin24/overlays.html#Access%20Logging)
  * LDAP Changelog. See [http://tools.ietf.org/html/draft-good-ldap-changelog-04](http://tools.ietf.org/html/draft-good-ldap-changelog-04)
  * TimeStamp. For Novell/NetIQ eDirectory, the Connector uses last date/time to get created and updated objects. Novell/NetIQ eDirectory does not provide an equivalent means to retrieve deleted objects. This option can also be used if no other delta import method is active on the LDAP server. This option is not able to import deleted objects.
  * USNChanged. See: [https://msdn.microsoft.com/library/ms677627.aspx](https://msdn.microsoft.com/library/ms677627.aspx)

Supported Directories for Delta import:

* Microsoft Active Directory Lightweight Directory Services (AD LDS)
* Microsoft Active Directory Global Catalog (AD GC)
* 389 Directory Server
* Apache Directory Server
  * Does not support delta import since this directory does not have a persistent change log
* IBM Tivoli DS
* Isode Directory
* Novell eDirectory and NetIQ eDirectory
  * Supports Add, Update, and Rename operations for delta import
  * Does not support Delete operations for delta import
  * The delta import does not detect any object deletes. It is necessary to run a full import periodically to find all deleted objects.
* Open DJ
* Open DS
* Open LDAP (openldap.org)
* Oracle (previously Sun) Directory Server Enterprise Edition
* RadiantOne Virtual Directory Server (VDS)
  * Must be using version 7.1.1 or higher
* Sun One Directory Server

## Step 4. Configure provisioning in Azure AD
1. Assign the agents to your application (get steps from preview doc).
2. Provide the URL and secret token (get steps from preview doc). 
2. [Determine who should be in scope for provisioning](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts).
3. [Assign users to your application](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users) if scoping is based on assignment to the application (recommended).
4. [Configure your attribute mappings.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes)
5. [Provision a user on-demand.](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/provision-on-demand)
6. Add additional users to your application.
7. Turn provisioning on.

## Step 5. Monitor your deployment
Once you've configured provisioning, use the following resources to monitor your deployment:

1. Use the [provisioning logs](../reports-monitoring/concept-provisioning-logs.md) to determine which users have been provisioned successfully or unsuccessfully.
2. Check the [progress bar](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) to see the status of the provisioning cycle and how close it is to completion.
3. If the provisioning configuration seems to be in an unhealthy state, the application will go into quarantine. Learn more about quarantine states [here](../app-provisioning/application-provisioning-quarantine-status.md).  

## Troubleshooting tips
Ensure that you have TCP connectivity from the system to your LDAP server.  One way to check this is to open a PowerShell window on the same computer as where ECMA Connector Host is installed, and type 
  
```Powershell
Test-NetConnection -ComputerName <valid IP address> -port 389 -InformationLevel "Detailed" 
```
  
(replacing &lt;valid IP address&gt; with the IP address of your LDAP server, and a different port number if not 389) 

If the output's last line is  

`
TcpTestSucceeded        : False 
`

then this indicates there is a network connectivity problem to the LDAP server from that system, which will need to be addressed prior to continuing with the configuration of the ECMA Connector Host. 

For reference, see the test-netconnection documentation [here](https://docs.microsoft.com/powershell/module/nettcpip/test-netconnection?view=win10-ps).


## Known issues

* LDAP referrals between servers not supported (RFC 4511/4.1.10)

# Content removed from previous version
This article describes the Generic LDAP Connector. The article applies to the following products:

* Microsoft Identity Manager 2016 (MIM2016)
* Forefront Identity Manager 2010 R2 (FIM2010R2)
  * Must use hotfix 4.1.3671.0 or later [KB3092178](https://support.microsoft.com/kb/3092178).

For MIM2016 and FIM2010R2, the Connector is available as a download from the [Microsoft Download Center](http://go.microsoft.com/fwlink/?LinkId=717495).

When referring to IETF RFCs, this document is using the format (RFC [RFC number]/[section in RFC document]), for example (RFC 4512/4.3).
You can find more information at [https://tools.ietf.org/](https://tools.ietf.org/). In the left panel, enter an RFC number in the **Doc fetch** dialog box and test it to make sure it is valid.

Certain operations and schema elements, such as those needed to perform delta import, are not specified in the IETF RFCs. For these operations, only LDAP directories explicitly specified are supported.

For connecting to the directories, we test using the root/admin account.  To use a different account to apply more granular permissions, you may need to review with your LDAP directory team.

To Create a Generic LDAP connector, in **Synchronization Service** select **Management Agent** and **Create**. Select the **Generic LDAP (Microsoft)** Connector.

![CreateConnector](./media/microsoft-identity-manager-2016-connector-genericldap/createconnector.png)



* The following LDAP controls/features must be available on the LDAP server for the connector to work properly:  
`1.3.6.1.4.1.4203.1.5.3` True/False filters

The True/False filter is frequently not reported as supported by LDAP directories and might show up on the **Global Page** under **Mandatory Features Not Found**. It is used to create **OR** filters in LDAP queries, for example when importing multiple object types. If you can import more than one object type, then your LDAP server supports this feature.

If you use a directory where a unique identifier is the anchor the following must also be available (For more information, see the [Configure Anchors](#configure-anchors) section):  
  `1.3.6.1.4.1.4203.1.5.1` All operational attributes

If the directory has more objects than what can fit in one call to the directory, then it is recommended to use paging. For paging to work, you need one of the following options:

**Option 1:**  
`1.2.840.113556.1.4.319` pagedResultsControl

**Option 2:**  
`2.16.840.1.113730.3.4.9` VLVControl  
`1.2.840.113556.1.4.473` SortControl

If both options are enabled in the connector configuration, pagedResultsControl is used.

`1.2.840.113556.1.4.417` ShowDeletedControl

ShowDeletedControl is only used with the USNChanged delta import method to be able to see deleted objects.

      The connector tries to detect the options present on the server. If the options cannot be detected, a warning is present on the Global page in the connector properties. Not all LDAP servers present all controls/features they support and even if this warning is present, the connector might work without issues.


* For information on how to enable logging to troubleshoot the connector, see the [How to Enable ETW Tracing for Connectors](http://go.microsoft.com/fwlink/?LinkId=335731).
