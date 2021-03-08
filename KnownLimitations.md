## Known limitations

1. Applications and directories not yet supported
   1. This feature is **not supported for writeback to AD DS**.

      1. When a user is managed by Azure AD connect, the source of authority is on-prem AD. Therefore, user attributes cannot be changed in Azure AD. This preview does not change the source of authority for users managed by Azure AD Connect.

      1. Attempting to use Azure AD Connect and the on-prem provisioning to provision groups / users into AD DS can lead to creation of a loop, where Azure AD Connect can overwrite a change was made by the provisioning service in the cloud. Microsoft is working on a dedicated capability for group / user writeback. Please upvote the  UserVoice feedback [here](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/16887037-enable-user-writeback-to-on-premise-ad-from-azure) to track the status of the preview.

   1. Connectors other than generic LDAP and SQL
      1. The ECMA host is officially supported for the GLDAP and GSQL connectors. While it is possible to use other connectors such as the web services connector or custom ECMA connectors, it is not yet supported. We plan to support for all ECMA2 connectors, but have not yet announced support for connectors other than GSQL and GLDAP. 

   1. Azure Active Directory is not supported as a target system
      1. On-prem provisioning allows you to take a user in Azure AD and provision them into a third party application. It does not allow you to bring a user into the directory from a third party application. Customers will need to rely on our native HR integrations, Azure AD Connect, or MIM to bring users intot he directory.  

1. Attributes and objects not yet supported
   1. Multi-valued attributes
   1. Reference attributes (e.g. manager).
   1. Groups 
   1. Complex anchors (e.g. ObjectTypeName+UserName).
   1. On-premises applications are sometimes not federated with Azure AD and require local passwords. The on-premises provisioning preview **does not support provisioning 1-time   passwords or synchronizing passwords** between Azure AD and 3rd party applications. For security reasons, Microsoft is working to eliminate passwords and help customers transition to a passwordless environment.

1. The Connector Host currently requires either SSL certificate to be trusted by Azure or the Provisioning Agent to be used. Certificate subject must match the host name ECMA Connector Host is installed on.

1. The Connector Host currently does not support anchor attribute changes (renames) or target systems which require multiple attributes to form an anchor.  \&lt;dn\&gt; rename operations are not yet possible.

Note: as described in the Microsoft Online Services terms, located at [http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&amp;DocumentTypeId=46](http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&amp;DocumentTypeId=46), previews may employ lesser or different privacy and security measures than those typically present in the Online Services. Unless otherwise noted, Previews are not included in the SLA for the corresponding Online Service, and Customer should not use Previews to process Personal Data or other data that is subject to legal or regulatory compliance requirements.  In particular, the following terms in the section (&quot;Data Protection Terms&quot;) of the Online Services terms do not apply to Previews:  Processing of Personal Data; GDPR, Data Security, and HIPAA Business Associate.
