# Troubleshooting connectivity

Test connection is failing. How do I troubleshoot?

1. Ensure the agent and host services are running.

1. Search for services in your windows task bar and check to ensure the Microsoft Azure AD Connect provisioning agent service is running as well as the Microsoft ECMA connector host service

1. Test if a SCIM request works locally. If the request below returns a 405, you have the right endpoint and the host is receiving the request. If the request below does not return 405, you need to troubleshoot your host setup and ensure that you&#39;re using the right URL. See instructions below for the URL format. Make sure to add /users to the end of the request when testing in your browser (/users is not required when providing the URL in the cloud).

![](RackMultipart20210115-4-mlm6xl_html_33434b3df4c6f8ad.gif)

1. Ensure that the agent is active by navigating to your application in the azure portal \&gt; click on admin connectivity \&gt; click on the agent dropdown and ensuring your agent is active.

1. Check if the secret token provided is the same as the secret token on-prem (you will need to go on-prem and provide the secret token again and then copy it into the cloud).

1. Ensure that the URL provided matches the following pattern:

[https://hostname:8585/ecma2host\_connectorName/scim](https://hostname:8585/ecma2host_connectorName/scim)

1. Ensure that you are using a valid certificate. Please see Appendix F for instructions on how to create a valid certificate.

1. Restart the provisioning agent by navigating to the task bar on your VM \&gt; searching for the Microsoft Azure AD Connect provisioning agent \&gt; Right click stop and then start
