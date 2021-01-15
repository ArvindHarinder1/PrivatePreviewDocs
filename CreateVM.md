Build a VM for testing

The instructions below demonstrate how to build a Virtual Machine for testing the ECMA host. These steps are not required and you can use your own on-prem server.

1. Navigate to  **portal.azure.com**  \&gt;  **Virtual Machines**  \&gt;  **add machine.**

2. Configure your VM with the settings as described below.

![](RackMultipart20210115-4-mlm6xl_html_457c2e1654d13ee3.png)

3. Click review + create.

4. Set auto shutdown to off.

![](RackMultipart20210115-4-mlm6xl_html_f68363aceb3c4631.png)

5. Ensure the VM is started and click connect to download the RDP file.

![](RackMultipart20210115-4-mlm6xl_html_8c2407983bcce193.png)

6. When prompted sign in with the username / password you provided when creating the VM (you may need to wait for a few minutes before it is successful).

![](RackMultipart20210115-4-mlm6xl_html_ce411b559a6886a9.png)

7. Search for the &quot;Server manager&quot; in your task bar and click &quot;Add roles and features&quot;.

8. Select Active Directory Domain Services

![](RackMultipart20210115-4-mlm6xl_html_586072240ab2e0c5.png)

9. Select .net is selected

![](RackMultipart20210115-4-mlm6xl_html_12051c25428f268c.png)

10. Click next through the rest of the wizard (It will take a few minutes for the installation to complete).

11. Click on the flag at the top left of the server manager and select &quot;promote this server to a domain controller.&quot;

12. Select &quot;add a new forest&quot; and provide a root domain name (e.g. ecmaTest.onmicrosoft.com).

13. Click next through the rest of the wizard and click install.

14. Search for &quot;Dsa.msc&quot; \&gt; right click on user \&gt; add new

15. Check the boxes &quot;password never expires&quot; and &quot;don&#39;t have to provide new password on next login&quot;

16. Search for &quot;Dsa.msc &quot; in the windows task bar.

17. Right click on user and click &quot;add new.&quot;

![](RackMultipart20210115-4-mlm6xl_html_234a381dce6f461d.png)

18. Right click the following checkboxes

19. Password never expires.

20. Don&#39;t have to provide new password on next login.

21. Add the user to the domain admins group by right clicking on the user, clicking add group, typing administrator into the search box, and clicking OK.

![](RackMultipart20210115-4-mlm6xl_html_22a92499812348e2.png)
