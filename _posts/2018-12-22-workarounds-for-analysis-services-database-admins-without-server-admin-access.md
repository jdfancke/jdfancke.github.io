---
layout: post
title: 'Workarounds for Analysis Services database admins without server admin access'
date: 2018-12-22 09:03:19.000000000 +01:00
summary: 'Workarounds for Analysis Services database admins without server admin access'
mermaid: true
tag: PBI
---
_This is a somewhat obscure topic, but I hope that others (if they ever find themselves in this position) find it useful!_

In Analysis Services you can have [administrator rights over an entire server instance](https://docs.microsoft.com/en-us/sql/analysis-services/instances/grant-server-admin-rights-to-an-analysis-services-instance?view=sql-server-2017), as well as [administrator rights over particular databases](https://docs.microsoft.com/en-us/sql/analysis-services/multidimensional-models/grant-database-permissions-analysis-services?view=sql-server-2017) on the server instance.

To give an example, let’s say that an organisation has a structure where there is one server administrator responsible for managing the Analysis Services server, and many database administrators responsible for developing, deploying, and updating particular database models hosted on that server.

This situation may become more common as organizations move their infrastructure to the cloud and consolidate server deployments to Azure Analysis Services. When this happens there may be multiple developers working on different models all hosted by a single server that supports a specific country/region.

In this example there are limitations that these database admins may face because they are not also server admins.

To get straight to the point, listed below are the main limitations that I have found so far, as well as any workarounds.

## Limitation 1: Database admins cannot create new databases

I wasn’t sure whether to include this limitation or not because it’s obvious.

Without the database existing, it follows that there is no database for you to have database administrator access over.

There is no workaround.

In this scenario you just need a server administrator to create a new database and create a role within it that has administrator access, and then add you to that role.

## Limitation 2: Database admins cannot deploy a model to the server using restricted data source settings such as impersonation mode (_except when using BISM-Normalizer_)

I’m not sure if this is a bug within Analysis Services or working as intended. But the result seems to be that for the vast majority of the time, you can't deploy your model using SSDT (or [Tabular Editor](https://tabulareditor.github.io/)!).

The problem you will encounter is that you are unable to deploy updated model metadata via SSDT to a database that you are the administrator over if the metadata contains an ImpersonationMode (e.g. the data source settings include impersonation of another account).

_Information on impersonation from [Microsoft docs here](https://docs.microsoft.com/en-us/sql/analysis-services/tabular-models/impersonation-ssas-tabular?view=sql-server-2017), a forum post [where the "workaround" is to just make them a server admin here](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/b54944ad-bf8a-4a45-9264-e52a3f501071/the-impersonationinfo-for-datasource-contains-an-impersonationmode-that-can-only-be-used-by-a-server?forum=sqlanalysisservices), and [another sort of workaround here, but may not necessarily apply to Tabular](http://www.biadmin.com/2012/08/ssas-2012-gotcha-impersonation-info-in.html)._

The error message looks like this:
```
------ Deploy started: Project: VSProject, Configuration: Development x86 ------
Cannot deploy metadata. Reason: Failed to save modifications to the server. 
Error returned: 'The ImpersonationInfo for datasource contains an ImpersonationMode that can only be used by a server administrator.
```
<div align='center'><img src = '/attachments/SSDT-Cannot-Deploy-to-AAS.png' width='400'></div>

Unfortunately, you also receive the same error when deploying from Tabular Editor, which hints at the fact that these two tools may have similar deployment methods.

<div align='center'><img src = '/attachments/Tabular-Editor-Cannot-Deploy-to-AAS.png' width='400'></div>
Tabular Editor also cannot deploy without server admin rights when using ImpersonationMode settings\[/caption\]

Note that there also appears to be a bug with Tabular Editor in that AAS role members cannot be deployed to an AAS model. If you test this out for yourself make sure you don’t include role members, otherwise you may receive a different error before you get to the ImpersonationMode error.

**The workaround:** here’s where it gets interesting, you can actually avoid this error message by deploying via [BISM-Normalizer](http://bism-normalizer.com/). I’m not sure yet what black magic is used to make this work, but it’s obviously related to the deployment methods. Paging [@\_christianWade](https://twitter.com/_christianWade) if he’s able to shed any light on this!

<div align='center'><img src = '/attachments/BISM-Normalizer-Deploy-to-AAS.png' width='400'></div> 
Deploying metadata without server admin rights via BISM-Normalizer is successful! However server profile traces are not allowed.

I’ve had a look at the deployment trace files and can’t seem to figure out how BISM-Normalizer is doing it, but I’ll keep exploring and will update this post if I find anything interesting. I have a sinking feeling it might require [looking through the SSAS Tabular protocol](https://msdn.microsoft.com/library/mt719260)...

A quick tip: make sure that if you want a full deployment via BISM-Normalizer that you include everything you mean to. (e.g. including partitions / perspectives / roles / members / etc.)

**Limitation 3: Database admins cannot set up server-side automated refreshing of data (aka “processing”).**

There are many different ways to automate the “processing” of your model. Processing is the term used to refresh the data and perform static calculations such as re-calculating DAX columns or tables.

In Azure, from what I’ve seen so far, most methods rely on [adding a service credential to the list of server administrators](https://docs.microsoft.com/en-us/azure/analysis-services/analysis-services-addservprinc-admins) or using a server administrator’s credentials. (while on this topic, [running your automation through a runbook](https://blogs.technet.microsoft.com/uktechnet/2018/06/22/how-to-automate-processing-your-azure-analysis-services-models/) means you can only refresh as often as once per hour, although there are [other methods](https://azure.microsoft.com/en-au/blog/automating-azure-analysis-services-processing-with-azure-functions/) in AAS to have quicker processing).

In on-premises I believe it relies on the SQL Server Agent using proxy credentials of a server administrator.

**_Workaround_**_:_ Get around this by using remote process execution via PS + Task Scheduler. [I made a post last week on how to do this]({% post_url 2018-12-14-how-to-automate-azure-analysis-services-processing-from-a-remote-pc %}).

**Limitation 4: Database admins have limited query tracing capability**

Note that when you use BISM-Normalizer to deploy your model, you will receive the error message (above) after it has been deployed saying that a server trace was unable to be captured.

The reason for this error message is because only server admins can capture trace events..._sort of_.

If you connect SQL Server Profiler to a server you might have to wait for a minute or two but it should eventually connect in a somewhat limited fashion, and you actually can capture what appear to be partial traces of queries sent to the database on the server. (Similar to how you can see most "DISCOVER" [Dynamic Management Views](https://docs.microsoft.com/en-us/sql/analysis-services/instances/use-dynamic-management-views-dmvs-to-monitor-analysis-services?view=sql-server-2017)).

This makes sense because if you’re querying a server, you should only be able to see the queries that relate to the databases that you administer, however there are also many events that relate to the server in general, such as connection attempts with the server itself, as well as queries to other databases.

Note that when using SQL Server Profiler to connect to Analysis Services you only specify the server, not the actual database, although the database _can_ be filtered using the row filter settings once you’re connected.

Sadly, part of not being able to perform traces means that [DAX Studio](http://daxstudio.org/) runs into fatal errors or displays the same error message as when you deployed the model using BISM-Normalizer.

<div align='center'><img src = '/attachments/DAX-Studio-Cannot-Do-Trace.png' width='400'></div>

This means that testing/optimizing your queries on the production instance will not work, and instead you'll just have to deploy it to a local developer instance or another server in which you have administrator rights.

Note that you can still get most Dynamic Management Views, with the exception of some DISCOVER views that require server administrator permissions.

**Closing remarks and a guess at the future**

Hopefully in the future SSDT and Tabular editor can overcome this deployment issue, however the workaround isn't that painful and using BISM-Normalizer for deployments (especially to a production database) is probably a best practice anyway!

In regards to how this relates to Power BI, hopefully the “publishing”(deployment) of Power BI "datasets"(models) to PBI “workspaces”(server instances) is done in a similar way to BISM-Normalizer instead of the current PBI publishing method or the method that SSDT and Tabular Editor use.

To get a glimpse at this, see the [MAQ software ALM toolkit for Power BI](https://maqsoftware.com/case-studies/alm-toolkit), and this display by [Amir Netz at the Business Applications Summit 2018](https://www.microsoft.com/en-us/businessapplicationssummit/video/BAS2018-204), starts at 26:30.