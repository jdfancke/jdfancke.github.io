---
layout: post
title: 'How to automate (Azure) Analysis Services processing from a remote PC'
date: 2018-12-14 20:56:47.000000000 +01:00
summary: 'A local SQL Server agent is unable to easily execute process commands against a remote Azure Analysis Services instance. This post details how to overcome this difficulty while also not having server administration rights.'
mermaid: true
tag: PBI
image: '/attachments/Analysis-Service.png'
---
There are many blog posts showing how to automate processing of a SQL Server Analysis Services model using either the SQL server agent or SSIS.

There are also many posts out there showing different ways of processing an Azure Analysis Services database through the use of Azure Functions, automation accounts, and runbooks. ([SQLDusty.com](https://sqldusty.com/2017/06/21/how-to-automate-processing-of-azure-analysis-services-models), [technet](https://blogs.technet.microsoft.com/uktechnet/2018/06/22/how-to-automate-processing-your-azure-analysis-services-models), [Azure blog](https://azure.microsoft.com/en-au/blog/automating-azure-analysis-services-processing-with-azure-functions), [official AAS docs](https://docs.microsoft.com/en-us/sql/analysis-services/instances/schedule-ssas-administrative-tasks-with-sql-server-agent?view=sql-server-2017), [byobi.com](http://byobi.com/2016/11/processing-an-azure-as-database))

All of the above require some level of server administration privileges or Azure resource deployment.

What if you don't have server admin rights or can't create Azure resources?

What if all you've got is the "process" rights granted to a role that you belong to?

If that's the case, then this post is for you!

The strategy is relatively simple and uses PowerShell in combination with Task Scheduler.  
You will require:

*   administration rights over a local machine in order to enable PowerShell scripting execution
*   an internet connection in order to send the processing command to the server
*   an account with processing rights over the database

Here are the steps that you need to follow:

**1\. Change the execution policy to allow you to execute PowerShell scripts.** Run the below script as an administrator:

```powershell
Set-ExecutionPolicy remotesigned
```
**2\. Make sure you have the SQLServer PowerShell module installed.** Information can be found [here](https://docs.microsoft.com/en-us/sql/powershell/download-sql-server-ps-module?view=sql-server-2017):

```powershell
Install-Module -Name SqlServer -force -allowclobber
```

**3\. You may now want to test that you can process the database via PowerShell by executing the following code.** Information can be found [here](https://docs.microsoft.com/en-us/powershell/module/sqlserver/invoke-processasdatabase?view=sqlserver-ps):

```powershell
Invoke-ProcessASDatabase -Server "YourServerName" -DatabaseName "YourDatabaseName" -RefreshType "RefreshTypeHere"
```

Here's an example:

```powershell
Invoke-ProcessASDatabase -Server "asazure://region.asazure.windows.net/databaseinstance1" -DatabaseName "Database1" -RefreshType "Full"
```

You should be prompted for your login credentials, enter the account that has access to process the model.

**4\. To automate the script, you should first create an encrypted password for your account.**

To do this, execute the script at the end of this step. It will ask you for your username and password credentials.

Use the credentials for the account that has access to process the database, e.g. domain\\username if on SQL Server Analysis Services, or the UPN (i.e. username@domain.com) for Azure Analysis Services.

The script will then create an encrypted password text file for the account where you ran the code from.

Make sure you place this encrypted password somewhere that it can be accessed by an unattended account, ideally just in a new folder on the C: drive. For this example I have stored it in C:\\Scripts\\

Further information can be found [here](https://interworks.com/blog/trhymer/2013/07/08/powershell-how-encrypt-and-store-credentials-securely-use-automation-scripts):

```powershell
$credential = Get-Credential

$credential.Password | ConvertFrom-SecureString | Set-Content c:scriptsencrypted\_password1.txt\
```

**5\. Next you will want to create the script that processes the model using your credentials (username and encrypted password).** Again, make sure that your encrypted password file is accessible by an unattended account and is not in any specific user folders. If you run the below script it should execute without any prompts.

```powershell
$username = "username@domain.com"

$encrypted = Get-Content "C:\\Scripts\\scriptsencrypted\_password.txt" | ConvertTo-SecureString

$cred = New-Object System.Management.Automation.PsCredential($username, $encrypted)

Invoke-ProcessASDatabase -Server "asazure://domain.asazure.windows.net/databaseinstance1" -DatabaseName "database1" -RefreshType "Full" -Credential $cred\
```

**6\. The final step is to configure Task Scheduler to run the script.** First you should create a new task, name it whatever you want (i.e. Process Analysis Services). Select the user that will have access to run the script, it can be the same account that you’re using to set up this task. You can also select “Run whether user is logged on or not”, and “run with highest privileges”.

<div align='center'><img src = '/attachments/Task-Scheduler-1.png' width='400'></div>

Next create a new trigger according to your own required schedule. I’ll leave this for you to work out.

Finally, create a new action, When creating a new action, you want to run the PowerShell executable, it should be C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe

<div align='center'><img src = '/attachments/Task-Scheduler-2.png' width='400'></div>

You can test this by opening Run and the path above in and seeing if PowerShell opens.

<div align='center'><img src = '/attachments/Task-Scheduler-3.png' width='400'></div>

In the arguments section you should add the -file argument followed by the path to your local PowerShell script. Note that the PowerShell script must be located somewhere that the account can access it from. It is probably best to also put this in your C drive next to your encrypted password file.

**7\. After you’ve set up the above, you can test to see if it processes!** Note that because the task is set to run whether the user is logged on or not, you will not see the PowerShell window open, it will instead run in the background.

