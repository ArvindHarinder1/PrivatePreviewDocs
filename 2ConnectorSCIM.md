The Azure AD provisioning [SCIM](https://aka.ms/scimoverview) client can be used to automatically provision users into cloud or on-premises applications. This document outlines how you can use the Azure AD provisioning service to provision users into an on-premises application that is SCIM enabled. If you're looking to provision users into non-SCIM on-premises applications, such as an app relying on an LDAP directory or SQL database, please the respective tutorials. If you're looking to provisioning users into cloud apps such as DropBox, Atlassian, etc. please review the SaaS application [tutorials.](https://docs.microsoft.com/azure/active-directory/saas-apps/tutorial-list) 

![image](https://user-images.githubusercontent.com/36525136/110343793-b2427100-7fe1-11eb-9bc3-05d6825f8f81.png)

## Pre-requisites
* Azure AD Premium license
* Administrator role for installing the agent (one time effort) - Hybrid admin / Global admin 
* Administrator role for configuring the application in the cloud (Application admin, Cloud application admin, Global Administrator, Custom role with perms)
* Application supporting the SCIM standard

## Steps
1. Add the provisioning private preview test application from the [gallery](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal).
1. Navigate to your application > Provisioning > Download the provisioning agent.
1. Click on on-premises connectivity and download the provisioning agent (during the private preview, you should skip this step and use the agent provided in the SharePoint site).
2. Copy the agent onto the virtual machine or server that your SCIM endpoint is hosted on.
3. Open the provisioning agent installer, agree to the terms of service, and click install.
4. Open the provisioning agent wizard and select on-premises provisioning when prompted for the extension that you would like to enable. 
5. Provide credentials for an Azure AD Administrator when prompted to authorize (Hybrid administrator or Global administrator required). 
6. Click confirm to confirm the installation was successful. 
7. Navigate back to your application > on-premises connectivity.  
8. Select the agent that you installed from the dropdown list and click assign agent. 
9. Wait 10 minutes or restart the Azure AD Connect Provisioning agent service on your server / VM.
11. Provide URL for your SCIM endpoint in the tenant URL field (e.g. Https://localhost:8585/scim). ![image](https://user-images.githubusercontent.com/36525136/115437009-801b5600-a1c0-11eb-903b-303117e9eeca.png) ![Uploading image.png…]()
12. Click test connection and save the credentials.
13. Configure any [attribute mappings](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/customize-application-attributes) or [scoping](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/define-conditional-rules-for-provisioning-user-accounts) rules required for your application.  
14. [Assign users and groups](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users) to the application that you would like to be provisioned.
15. Test provisioning a few users [on-demand](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/provision-on-demand). 
16. Add additional users into scope by assigning them to your application. 
17. Navigate to the provisioning blade and hit start provisioning. 
18. Monitor using the [provisioning logs](https://docs.microsoft.com/en-us/azure/active-directory/reports-monitoring/concept-provisioning-logs). 

## Things to be aware of
* Ensure your [SCIM](https://aka.ms/scimoverview) implementation meets the [Azure AD SCIM requirements](https://docs.microsoft.com/en-us/azure/active-directory/app-provisioning/use-scim-to-provision-users-and-groups).
  * Azure AD offers open source [reference code](aka.ms/scimreferencecode) that developers can use to bootstrap their SCIM implementation (the code is as-is)
* Support the /Schemas endpoint to reduce the configuration required in the Azure Portal. 
