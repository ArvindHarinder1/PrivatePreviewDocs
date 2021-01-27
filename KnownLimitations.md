## Updates and Known limitations of this preview

The September 2020 private preview build 1.6.83.0 has the following updates:

- HTTPS endpoint implemented. SSL certificate selection is now mandatory.
- Custom attributes syntax has changed to match Azure portal expression, e.g. urn:ietf:params:scim:schemas:extension:MyCustomExtensionName:2.0:MyCustomClassName:MyCustomAttribute
- Bug fixes, including handling of boolean attributes and fixes to deprovisioning flows.

This private preview build also has the following limitations and known issues.  If you experience an issue other than one of these listed below, please report it to us via the form or email address on the first page of this guide.


1. The Connector Host currently requires either SSL certificate to be trusted by Azure or the Provisioning Agent to be used. Certificate subject must match the host name ECMA Connector Host is installed on.

1. The Connector Host currently does not support anchor attribute changes (renames) or target systems which require multiple attributes to form an anchor.  \&lt;dn\&gt; rename operations are not yet possible.

1. Multi-valued attributes are not yet supported.

1. Reference attributes are not yet supported.

1. Complex anchors, e.g. ObjectTypeName+UserName, are not yet supported.

1. Provisioning passwords into on-premises applications

1. On-premises applications are sometimes not federated with Azure AD and require local passwords. The on-premises provisioning preview does not support provisioning 1-time passwords or synchronizing passwords between Azure AD and 3rd party applications. For security reasons, Microsoft is working to eliminate passwords and help customers transition to a passwordless environment.

1. User writeback from Azure AD to on-prem Azure AD is not supported

1. The Azure AD on-premises provisioning preview allows you to provision users into on-prem data stores such as an LDAP directory and a SQL database. However, this preview is  **NOT**  intended to be used to writeback a user from Azure AD to AD DS, for any domain in which Azure AD Connect has been enabled. Attempting to writeback users has the following limitations:

1. When a user is managed by Azure AD connect, the source of authority is on-prem AD. Therefore, user attributes cannot be changed in Azure AD. This preview does not change the source of authority for users managed by Azure AD Connect.

1. Using Azure AD Connect and the on-prem provisioning preview can lead to creation of a loop, where Azure AD Connect can overwrite a change was made by the provisioning service in the cloud. Provisioning to AD is not supported. Microsoft is working on a dedicated capability for user writeback. Please upvote the  UserVoice feedback [here](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/16887037-enable-user-writeback-to-on-premise-ad-from-azure) to track the status of the preview.

Note: as described in the Microsoft Online Services terms, located at [http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&amp;DocumentTypeId=46](http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&amp;DocumentTypeId=46), previews may employ lesser or different privacy and security measures than those typically present in the Online Services. Unless otherwise noted, Previews are not included in the SLA for the corresponding Online Service, and Customer should not use Previews to process Personal Data or other data that is subject to legal or regulatory compliance requirements.  In particular, the following terms in the section (&quot;Data Protection Terms&quot;) of the Online Services terms do not apply to Previews:  Processing of Personal Data; GDPR, Data Security, and HIPAA Business Associate.
