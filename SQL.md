

## Appendix G: Building a demo SQL environment for testing

1. Download SQL Server - [https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019](https://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019)

1. Download SSMS - [https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN&amp;view=sql-server-ver15](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN&amp;view=sql-server-ver15)

1. Open SQL Server management studio (SSMS)

1. Click on Browse for more, select the database under database engine (in my case 1110VM), and connect.

![](RackMultipart20210115-4-mlm6xl_html_5deb250b6bf40dc6.png)

1. Right click databases and create a new database and provide a name. Mine is &quot;ECMA2Host\_AppName&quot;.

1. Click new query and copy in the query from the employees file. Ensure that the table name matches as highlighted below.

![](RackMultipart20210115-4-mlm6xl_html_59e4119366bf37c7.png)

1. Open the sap.dsn file and update the server name (in my case it is &quot;SERVER=1110VM&quot;).

1. Ensure that the database name matches the name from step 6.

## Appendix H: Configuring the host using SQL

The following screenshots show you how to configure the host, if you are using the demo environment described above.

 
