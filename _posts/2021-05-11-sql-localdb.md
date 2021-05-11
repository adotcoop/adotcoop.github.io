---
layout: post
title:  "SQL Server Express LocalDB"
date:   2021-05-11 10:41:52 +0100
--- 

SQL Server Express LocalDB is a strange offering from Microsoft. A cut down version of SQL Server Express intended for developers that doesn't get security updates. Wait, what? No security updates? From the documentation at [docs.microsoft.com](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/sql-server-express-localdb?view=sql-server-ver15)

> LocalDB cannot be patched beyond Service Packs. CUs and Security Updates cannot be applied manually and will not be applied via Windows Update, Windows Update for Business, or other methods.

But we’re getting ahead of ourselves. What exactly is LocalDB, and how is it different from other versions of SQL Server? And, more importantly, if we have applications where LocalDB is a prerequisite, what is the best way of deploying it? If it can't be patched, should we be deploying it?

This article is pretty long, so it's divided into three sections. Firstly, we'll cover what LocalDB is, how to obtain it and how to update it. Secondly, we will look at the security implications of running this software. Finally we'll cover deployment and configuration in an enterprise environment.

In the interests of full disclosure I am not a SQL Server expert. I did go on a SQL Server 6.5 course once and passed the SQL Server 7.0 exam in 1999 as part of my MCSE in NT 4.0,  but that is a very, very long time ago. My interest in the product is now solely as a dependency for other applications being deployed on end user devices. LocalDB can be used in production on servers - Citrix uses it for the Local Host Cache on its Virtual Apps and Desktops product, and you have probably seen it if you use Azure AD Connect - but the main focus here is on the needs of the IT admin who has to deploy this software and needs to comply with regular security audits. 

* TOC
{:toc}

## A user mode SQL Server

Most people in IT are aware of the two main versions of SQL Server available

- SQL Server (Standard, Enterprise etc)
- SQL Server Express

SQL Server Express is a cut down version of the full SQL Server. It has some hardcoded limits on database sizes, RAM usage and CPU but can be used for production workloads. LocalDB is a further subset of Express.

There is a concise description of LocalDB on the [download page for SQL Server Express 2014](https://www.microsoft.com/en-us/download/details.aspx?id=42299)

> LocalDB is a lightweight version of Express that has all its programmability features, yet runs in user mode and has a fast, zero-configuration installation and short list of pre-requisites. Use this if you need a simple way to create and work with databases from code. It can be bundled with Application and Database Development tools like Visual Studio and or embedded with an application that needs local databases.

The main difference between LocalDB and the normal SQL Server Express is that LocalDB does not run as a service.

If LocalDB is just for developers, why should anyone other than developers care? Well, in a blog post from 2011, [Introducing LocalDB, an improved SQL Express](https://docs.microsoft.com/en-gb/archive/blogs/sqlexpress/introducing-localdb-an-improved-sql-express), we are told

> if the simplicity (and limitations) of LocalDB fit the needs of the target application environment, developers can continue using it in production, as LocalDB makes a pretty good embedded database too.

And, yes, some developers decided to use this in production and bundle it with their installers. 

And if you've packaged applications for any length of time, you've probably noticed this being installed. If not, you've probably noticed it when your security team audits your endpoints!

To compound things, LocalDB will often appear as a *required* pre-requisite of another application, sometimes without an option to prevent its installation.

If we are packaging an app that requires LocalDB we need to know some things before releasing it to production
- What version of SQL Server LocalDB is bundled with the installer?
- Is this version supported by Microsoft?
- Are there any updates we can apply to LocalDB?
- What are the security implications of using this, and does our organisation accept these risks?

## A supported unpatchable SQL Server?

It would appear so. Attempting to install a SQL Server service pack or cumulative update will fail if only LocalDB is installed. The only way to update LocalDB is to install a newer version over the top of an existing version. This appears to be [a very longstanding problem with localDb](https://sqlblog.org/2012/09/06/sqllocaldb-vs-cumulative-updates). 

There is a workaround, and that is to install the full SQL Server Express. Updates to SQL Server Express appear to update LocalDB as well. However, most software that installs LocalDB as a prerequisite do not install the full version of SQL Server Express, only the LocalDB component. 

To understand the risk that LocalDB poses to your environment (and what is the latest version you can apply) we need to understand SQL Server patching.

## SQL patches explained

SQL Server patches are pretty confusing. Unlike other apps you may encounter Microsoft still support *a lot* of different versions of SQL Server at the time of writing. As is sadly more often the case these days, the best resource for learning what products are supported, and what patches are required, is not Microsoft. Instead head to the quite excellent [sqlserverupdates.com](https://sqlserverupdates.com/) for a list of each major version, when support ends, and how to get current. 

![alt text](/assets/localdb_sqlserverupdates.png "sqlserverupdates.com web page")

Some of the acronyms used for SQL patches were unfamiliar to me. I couldn't find a good page from Microsoft that explained everything but luckily Aaron Bertrand details everything you need to know (and more) about SQL Server acronyms 
[in this blog post](https://www.sentryone.com/blog/aaronbertrand/back-to-basics-release-acronyms).

To really understand what's going on, read Aaron's blog, but for our purposes we don't need to know all of the acronyms - the following should be enough

| Acronym | Explanation                  | Notes                                                                   |
| :------ | :--------------------------- | :---------------------------------------------------------------------- |
| SP      | Service Pack                 |                                                                         |
| CU      | Cumulative Update            | A collection of updates much like a Service Pack                        |
| GDR     | General Distribution Release | Kind of like a hotfix but widely available                              |
| QFE     | Quick-Fix Engineering        | Kind of like a hotfix but not widely available and for a specific issue |

One thing to note with the Cumulative Updates (CU) is that they apply to a specific Service Pack level. When a new Service Pack is released the CU counter returns to 1.

For example, [SQL Server 2014 Service Pack 2 ](https://support.microsoft.com/en-us/help/3171021/kb3171021-sql-server-2014-service-pack-2-release-information)
	
> includes hotfixes that were included in SQL Server 2014 SP1 CU1 to SQL Server 2014 SP1 CU7

and [SQL Server 2014 Service Pack 3](https://support.microsoft.com/en-us/help/4022619/kb4022619-sql-server-2014-service-pack-3-release-information)

> includes hotfixes that were included in SQL Server 2014 Cumulative Update 1 (CU1) through SQL Server 2014 SP2 CU13

Returning to the [sqlserverupdates.com](https://sqlserverupdates.com/) screenshot above, we can see where there might be issues with deploying this software into production.


| Version | Latest LocalDB Version | Missing from LocalDB |
| :------ | :--------------------- | :------------------- |
| 2019    | RTM                    | CU8                  |
| 2017    | RTM                    | CU22                 |
| 2016    | SP2                    | CU15                 |
| 2014    | SP3                    | CU4+GDR              |
| 2012    | SP4                    | GDR                  |


The table gives a good feel for what updates (security or otherwise) will be missing from a LocalDB install. For most organisations it'll be up to their information security team to evaluate whether the missing updates are justification to block deployment. Later on we'll see how LocalDB mitigates some of the potential security issues by running in user mode. 

## The modern servicing model
After getting over the initial shock that *Security Updates cannot be applied to LocalDB*, with a bit of investigation things don't look as bad. LocalDB gets updated when a Service Pack is released. As long as Service Packs are released on a regular basis we can probably get LocalDB *current enough* to be secure. 

Unfortunately with the introduction of the modern servicing model there will be no more Service Packs

> Starting with SQL Server 2017, we are adopting a simplified, predictable mainstream servicing lifecycle: SPs will no longer be made available. Only CUs, and GDRs when needed.

[Announcing the Modern Servicing Model](https://docs.microsoft.com/en-gb/archive/blogs/sqlreleaseservices/announcing-the-modern-servicing-model-for-sql-server)

Unless there is a change to this policy it would appear that the risk of running 2017 or 2019 LocalDB will increase over time as there will be no way to patch it. 

Assuming Microsoft stick to this policy it becomes even more important to understand what the implications of LocalDB are to the security of your endpoints.

## Security context 

I started writing this article after finishing up our annual Cyber Essentials Plus audit. For the unaware, Cyber Essentials is a UK government backed scheme (see [NCSC](https://www.ncsc.gov.uk/cyberessentials/overview) for details) that aims to validate that you are doing the bare minimum of cyber security for your organisation. Limiting the use of admin rights, configuring things correctly, making sure operating systems and applications are regularly patched (and patchable), that sort of thing.  So I was a bit concerned to see the following line in a nessus scan of our environment

![alt text](/assets/localdb_nessus.png "MS15-058: Vulnerabilities in SQL Server Could Allow Remote Code Execution")

It turned out that this was true - we had endpoints running an old unpatched version of LocalDB in our environment. Installing a newer version improved things and removed the risk of this particular vulnerability, but it got me thinking about what the real risk of running this software could be.

### Understanding the risk of user mode SQL Server

For most people there is not much option but to deploy this. Some very large, expensive, critical pieces of software have LocalDB as a prerequisite, so the best you can do is evaluate the risk. 

To understand the risk LocalDB poses to your environment, the docs article has this to say

> An instance of SQL Server Express LocalDB is an instance created by a user for their use.

Furthermore

> LocalDB always runs under the users security context; that is, LocalDB never runs with credentials from the local Administrator's group. This means that all database files used by a LocalDB instance must be accessible using the owning user's Windows account, without considering membership in the local Administrators group.

Does this make running LocalDB any safer? Normal SQL Server runs as a service and, depending on the accounts used to run the service, can lead to total compromise of a system if a vulnerability is exploited. Since LocalDB runs in the context of a user then this mitigates the threat. Obviously if your users have admin rights then all bets are off. 

We can verify that LocalDB runs in user context by running Process Explorer when LocalDB launches

![alt text](/assets/localdb_processexplorer.png "Process explorer SQL Server process in user context")

Compare this to a full install of SQL Server, and pay particular attention to the Autostart Location and Parent process

![alt text](/assets/localdb_processexplorer_fullsql.png "Process explorer SQL Server process running as a service")

The main things to take from this are that under normal circumstances the LocalDB process is only ever launched from another _user mode_ process in the _context of the logged on user_. The full SQL Server is run _as a service_ and runs under a _service account_. 

It should be clear here that LocalDB and the full SQL Server pose very different security threats to your environment.

The other thing to note is the Autostart location is missing under LocalDB - it is not launched automatically. In this example I've launched it from SQL Server Management Studio (smss.exe). So, unlike standard SQL Server, it does not run all the time - it only ever starts when the application that has LocalDB as a dependency launches. Again, this helps mitigate the risk of LocalDB in your environment.

### SQL Server VSS Writer Service

Despite being a user mode application, SQLLocalDB does install one service, the SQL Server VSS Writer. This is the SQL Server implementation of the [Volume Shadow Copy Service](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) and is documented [here](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/sql-writer-service)

![alt text](/assets/localdb_services_SQLWriter.png "SQL Server VSS Writer Service")

This complicates the landscape slightly. The docs explain the security implications of this service

> The SQL Writer service must run under the Local System account. The SQL Writer service uses the NT Service\SQLWriter login to connect to SQL Server. Using the NT Service\SQLWriter login allows the SQL Writer process to run at a lower privilege level in an account designated as no login, which limits vulnerability.

Whilst there is an attempt to limit the impact of this service, it still runs in the context of Local System. The article goes on to explain that we can safely disable this service in most cases

> If neither SQL Server, the system it runs on, nor the host system (in the event of a virtual machine), need to use anything besides Transact-SQL backup, then the SQL Writer service can be safely disabled and the login removed.

You should only ever need to keep this service if the application you are deploying requires LocalDB VSS snapshots.

### LocalDB running as a service

All of the above is based on LocalDB running as a user mode process. There is nothing stopping an application that runs under a different security context from executing LocalDB. At the start of this article I mentioned that Citrix use LocalDB for the Local Host Cache feature of Virtual Apps and Desktops (the product formerly known as XenDesktop).

Citrix run LocalDB under the *NT AUTHORITY\NETWORK SERVICE* account. This obviously changes the security impact of the product dramatically. 

If the application you are deploying runs LocalDB as a service then speak to your information security team to evaluate the threat, especially if it runs under a privileged account such as *NETWORK AUTHORITY\NETWORK SERVICE*

### Exploiting xp_cmdshell

Aside from the issue of data leakage, a compromised SQL Server can be problematic for another reason - xp_cmdshell.

Disabled by default, xp_cmdshell allows an attacker to run commands on the machine hosting SQL Server. You can imagine the problem here if you can run a command such as ```net user username password /add``` from a SQL command. 

If we enable this feature using the following commands we can see that it does run, and runs under the context of the SQL Server (in this case in user mode)

```EXEC sp_configure 'show advanced option', '1'```

```RECONFIGURE```

```EXEC sp_configure 'xp_cmdshell', 1```

```RECONFIGURE```

```EXEC xp_cmdshell, 'net user'```

![alt text](/assets/localdb_xpcmdshell_netuser.png "xp_cmdshell running in localdb")

Is there much risk here? I would say no, but your local infosec person will be far better placed to explain your organisations appetite for risk than I am. 

In most cases, this is running under the context of the logged in user and, considering xp_cmdshell is disabled by default, I don't see much issue here. It would be a lot easier to trick a user into running the commands themselves than getting them to spin up a LocalDB instance and running some SQL.

### User mode SQL Server and multi-session Windows (WVD)

The fact that this product runs in user mode makes it interesting from a multi-user point of view. If we use RDS or Windows 10 multi-session then having standard SQL Express might cause us some concerns. 

However, the instance of SQL runs in the user context and, as we will see, the database files are stored in the users profile. This provides nice separation between users. 

A user can share their instance via named pipes using the command ```sqllocaldb share```, although I am yet to see any application take advantage of this. 

From my perspective, as long as LocalDB is being run as a non-admin user the risk is very low. Again, if you have doubts, flag this up to you security team.

## Downloading the latest version of LocalDB

Looking at the patching model for LocalDB the best we can do is get the latest service pack version of it and install that. And just when you think SQL Server can't get any more confusing, try to download it. 

For some reason, there is no direct link to the SqlLocalDB.msi available via Microsoft for 2016, 2017 and 2019. To obtain it you need to download and run a downloader which will then download the relevant MSI. 

In the following example, I've downloaded the SQL Server Express installer. If you click Download Media, you can then choose to download the LocalDB installer. This is all a bit convoluted. 

![alt text](/assets/localdb_setup1.png "SQL Server Express Installer")

![alt text](/assets/localdb_setup2.png "SQL Server Express Installer")

Earlier versions are only marginally better, forcing you through a web downloader instead.

![alt text](/assets/localdb_webdownload1.png  "SQL Server Express Web Downloader")

![alt text](/assets/localdb_webdownload2.png  "SQL Server Express Web Downloader")

Thankfully Scott Hanselman has a blog post that works around this madness. Head to [downloadsqlserverexpress.com](http://downloadsqlserverexpress.com) for direct links to the latest versions of LocalDB.

He doesn't have links to versions earlier than 2016, so here are the other versions still in support

- SQL Server 2014 SP3 Express
	- [SQL Server 2014 SP3 Express LocalDB x64](https://download.microsoft.com/download/3/9/F/39F968FA-DEBB-4960-8F9E-0E7BB3035959/ENU/x64/SqlLocalDB.msi)
	- [SQL Server 2014 SP3 Express LocalDB x86](https://download.microsoft.com/download/3/9/F/39F968FA-DEBB-4960-8F9E-0E7BB3035959/ENU/x86/SqlLocalDB.msi)
- SQL Server 2012 SP4 Express
	- [SQL Server 2012 SP4 Express LocalDB x64](https://download.microsoft.com/download/B/D/E/BDE8FAD6-33E5-44F6-B714-348F73E602B6/ENU/x64/SqlLocalDB.msi)
	- [SQL Server 2012 SP4 Express LocalDB x86](https://download.microsoft.com/download/B/D/E/BDE8FAD6-33E5-44F6-B714-348F73E602B6/ENU/x86/SqlLocalDB.msi)


## Packaging for deployment

### The basic silent install

As we have seen, there is no single place where all the documentation for LocalDB is kept. For silent install parameters we have to go to (another) 2011 blog post [Announcing SQL Server 2012 Express LocalDB RC0](https://docs.microsoft.com/en-gb/archive/blogs/sqlexpress/announcing-sql-server-2012-express-localdb-rc0)

```msiexec /i SqlLocalDB.msi /qn IACCEPTSQLLOCALDBLICENSETERMS=YES```

I would alter this to include my standard parameters to create a log file in case we need to troubleshoot later on

```msiexec /i SqlLocalDB.msi /qn IACCEPTSQLLOCALDBLICENSETERMS=YES /l*v "C:\WINDOWS\TEMP\SQLLocalDB_2014_x64.log"```

The article also makes an important point about running LocalDB in 32-bit and 64-bit Windows

> LocalDB doesn't support installing 32-bit version on 64-bit Windows, so you will need to distribute both versions of LocalDB with your application and they both have the same file name (SqlLocalDB.msi).

The uninstall command is a standard msiexec with the /x parameter. To find out the exact command look in the registry under the following path

```HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall```

In the following example we have SQL Server 2017 LocalDB installed

![alt text](/assets/localdb_uninstall.png "Windows Registry Uninstall Key")

Note also that, as is commonly the case with this key, the uninstall string is probably not what you want. I would change the command from 

```MsiExec.exe /I{216778FC-CC9A-4D47-AF5E-8223A37626D4}```

to look like


```msiexec /x {216778FC-CC9A-4D47-AF5E-8223A37626D4} /qn /l*v "C:\WINDOWS\TEMP\SQLLocalDB_2017_x64_uninstall.log"```


### Replacing an out of date version with the latest service pack or CU

If you have an install package that has the SQLlocalDB.msi file present, you might be able to just replace this in the source files with a more up to date version. Some caveats apply here

- Make sure the vendor is happy for you to do this. You might find that with a new version although your copy of LocalDB is now supported, the application you pay for is not. Some vendors will only support you on a specific set of prerequisites
- Often it is better to install the prerequisite separately instead of as part of an install chain from another package. A separate install means you can manage the exit codes and verify the prereq is already installed before the main application
- The prerequisite detection method for your application installer may be coded to look for a very specific version. You may need to use an app compat shim to prevent the main app installer from attempting to install the old version of LocalDB 

As a quick example let's look at an app distributed as an InstallShield package. Using the standard trick of looking in the TEMP folder while the installer is running, we can see a number of extracted files

![alt text](/assets/localdb_rwb_tempfiles.png "InstallShield tempfiles")

The file that contains the detection logic is ```SQL Server 2016 LocalDB.prq```

<script src="https://gist.github.com/adotcoop/d192645ee92a5ea10a67c087f549de86.js"></script>

Examining the contents of the file, the detection method is the presence of the registry key ```HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL13E.LOCALDB\MSSQLServer\CurrentVersion```

Looking at this file we can have confidence that we can just install the newer version of LocalDB before running the InstallShield installer. As long as that key is populated, the older version of LocalDB will not be installed. 

### Replacing the version (eg 2014 -> 2017)

Depending on the application you might be able to replace the version of LocalDB with a more up to date version. While you might be able to do this, check with the application vendor if this is a supported configuration. 

This will only work if the application hasn't hardcoded the version of SQL Server either in the path to the SQL Server executables or in the version that is required. 

### Is installation order important?

You might have to install multiple versions of LocalDB on a device. For example, you might have apps with dependencies on 2012, 2014 and 2017. 

Do you need to install these in a certain order?

From my testing, no. But I don’t have an extensive library of apps with LocalDB dependencies to test against, so your mileage may vary. 

If you only have a single version of LocalDB installed you can run all the executables without being explicit about the path or the version you require.

![alt text](/assets/localdb_path.png "LocalDB Paths")

In testing I haven’t seen any issues with running the wrong version of sqlloclalcmd to start or stop a different version of sqlserver.exe. If you have multiple bits of software with different version dependencies this is one that _should_ be ok, but needs checked before you deploy.


## Where are the databases kept?

Assuming our organisation is happy with the risk of running SQL Server LocalDB, and assuming the package installs correctly, the final thing to worry about is where do the databases sit?

Again, the old sqlexpress blog comes to the rescue with another post [LocalDB: Where is My Database?](https://docs.microsoft.com/en-gb/archive/blogs/sqlexpress/localdb-where-is-my-database). System databases are stored

> deep inside the "hidden" AppData folder in the user profile

The users databases are stored

> in the root of the user profile. On most machines it is located in C:\Users\user-name folder

To define these more properly the system databases (master, model, msdb and tempdb) are stored in 

``` C:\Users\username\AppData\Local\Microsoft\Microsoft SQL Server Local DB\Instances\```

The user databases are stored in 

```C:\Users\username```

Here's an example with LocalDB 2017 installed. This is what the users AppData looks like with the software installed but no database running


![alt text](/assets/localdb_userprofile_before.png "User profile before")

and this is what it looks like once the database has been called


![alt text](/assets/localdb_userprofile_after.png "User profile after")

## Managing the database using the SqlLocalDB utility

LocalDB doesn't ship with any graphical management tools, but Microsoft provide a command line utility ```SqlLocalDB.exe``` which you can use to verify a successful install and manage databases.

The tool is documented [here](https://docs.microsoft.com/en-us/sql/tools/sqllocaldb-utility)

Lets start with the basics

- ```sqllocaldb info```
- ```sqllocaldb versions```

These commands can be shortened to 

- ```sqllocaldb i```
- ```sqllocaldb v```

and this is how you'll probably see them mentioned on the web.

![alt text](/assets/localdb_sqllocaldbcmd_1.png "SqlLocalDb info and version commands")

Now we know that it is installed we can create a database and start it using the following command

![alt text](/assets/localdb_sqllocaldbcmd_2.png "sqllocaldb create 'EXAMPLEDB' 14.0 -s")

We could create the database with just

- ```sqllocaldb create "EXAMPLEDB"```

but the addition of ```14.0``` means we will definitely start a SQL 2017 instance (useful if there is more than one version installed) and the ```-s``` tells the command to start the database. We can verify that the process has been started by looking at process explorer

![alt text](/assets/localdb_procexp_sqlserverexe.png "sqlserver.exe shown running in Process Explorer")

## Managing the database using SQL Server Management Studio

For most packaging jobs the above detail is all you need. In some cases you might need to test that the database is functional and the easiest way to do that is by installing SQL Server Management Studio (SSMS). 

For our purposes SSMS is good because you don't need to match the SSMS version to the SQL version. This means you can install the latest version of SSMS and it will work with all supported versions of SQL Server 2008 onward.

### Installing SQL Server Management Studio

It doesn't seem right in a packaging article for me to install this interactively, so lets package this up too.

This link [aka.ms/ssmsfullsetup](https://aka.ms/ssmsfullsetup)
points to the latest download.

*Also note that (at the time of writing) there is no 64-bit version of this app available. All versions of SQL Server Management Studio are 32-bit only.*

Usual packaging procedure here - I launched the install to see if it gave away any clues about its origin. In this case I used Process Explorer after launching ```SMSS-Setup-ENU.exe```. If you hover over the secondary process you can see the command line parameter   contains the text ```-burn.unelevated BurnPipe.{GUID}```  and this is all we need to know that it was created with Wix Burn.

![alt text](/assets/localdb_ssms_wixidentification.png "Viewing SSMS install process in Process Explorer")

If you're interested in how to deal with those installers, see my Wix Burn article.

As I usually do with Wix Burn installers I've decompiled the ```SMSS-Setup-ENU.exe``` file using ```dark.exe``` and extracted both the ```manifest.xml``` file and the ```BootstrapperApplicationData.xml``` file. We can see additional command line options at the bottom of the bootstrapper file

![alt text](/assets/localdb_ssms_wixxml.png "Wix File Decompiled")


So the command line is a standard Wix silent install with our additional parameter

```SMSS-Setup-ENU.exe /silent DONOTINSTALLAZUREDATASTUDIO=1```

### Using SQL Server Management Studio to verify the LocalDB install 

Launch SQL Server Management Studio (SSMS) and enter the database details 

```(localdb)\MSSQLLOCALDB```

and click Connect.

![alt text](/assets/localdb_ssms_connect.png "SSMS Connection Dialog Box")


## Conclusion

During a recent audit nessus flagged up a very out of date copy of SQL Server running on some of our machines. Attempting to apply the latest service pack didn't work - it failed to detect the installed localDB. 

If run as intended, as a user mode process, LocalDB is a low risk. It does install a service running as LocalSystem - SQL Server VSS Writer Service - but this can be disabled or removed in most circumstances. 

LocalDB can be run as a service, and this obviously increases the risk. For these scenarios, the full version of SQL Server Express might be a better option as that can be patched.

There's a number of things here that could do with further investigation. Could App-V be used to mitigate some of the risk here? I haven't tried it, but since LocalDB is a user mode process it might work. And how well does the database roam if you are using a profile roaming solution such as fslogix? It should be OK but, again, I haven't tested it.

However you run the software, if you have applications with LocalDB as a dependency, you need to be aware of the risk to your environment and have someone in your organisation own that risk.