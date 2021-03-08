## Build a VM for testing

The instructions below demonstrate how to build a Virtual Machine for testing the ECMA host. These steps are not required and you can use your own on-prem server.

1. Navigate to  **portal.azure.com** > **Virtual Machines** > **add machine.**

2. Configure your VM with the settings as described below.

![image](https://user-images.githubusercontent.com/36525136/110341848-a3f35580-7fdf-11eb-9fd7-c3ac4038275a.png)


3. Click review + create.

4. Set auto shutdown to off.

![image](https://user-images.githubusercontent.com/36525136/110341897-b1a8db00-7fdf-11eb-967f-4eefe1ac18dd.png)


5. Ensure the VM is started and click connect to download the RDP file.

![image](https://user-images.githubusercontent.com/36525136/110341968-c1282400-7fdf-11eb-994b-08cc9274c250.png)


6. When prompted sign in with the username / password you provided when creating the VM (you may need to wait for a few minutes before it is successful).

![image](https://user-images.githubusercontent.com/36525136/110342025-d0a76d00-7fdf-11eb-9249-0d07b718a8b5.png)


7. Search for the &quot;Server manager&quot; in your task bar and click &quot;Add roles and features&quot;.

8. Select Active Directory Domain Services

![image](https://user-images.githubusercontent.com/36525136/110342065-dac96b80-7fdf-11eb-9df8-a3e6c1a3e385.png)

9. Select .net

![image](https://user-images.githubusercontent.com/36525136/110342109-e61c9700-7fdf-11eb-8cb2-4bc82adb6cab.png)


10. Click next through the rest of the wizard (It will take a few minutes for the installation to complete).

11. Click on the flag at the top left of the server manager and select &quot;promote this server to a domain controller.&quot;

12. Select &quot;add a new forest&quot; and provide a root domain name (e.g. ecmaTest.onmicrosoft.com).

13. Click next through the rest of the wizard and click install.

14. Search for &quot;Dsa.msc&quot; \&gt; right click on user \&gt; add new

15. Check the boxes &quot;password never expires&quot; and &quot;don&#39;t have to provide new password on next login&quot;

16. Search for &quot;Dsa.msc &quot; in the windows task bar.

17. Right click on user and click &quot;add new.&quot;

![image](https://user-images.githubusercontent.com/36525136/110342162-f2a0ef80-7fdf-11eb-9fa9-49c3daaa2450.png)

18. Right click the following checkboxes

19. Password never expires.

20. Don&#39;t have to provide new password on next login.

21. Add the user to the domain admins group by right clicking on the user, clicking add group, typing administrator into the search box, and clicking OK.

![image](https://user-images.githubusercontent.com/36525136/110342207-fdf41b00-7fdf-11eb-81c6-3e3cc92f9cf8.png)

