---
layout: post
title: 'Power BI Datamarts First Impressions'
date: 2022-05-30 07:32:39.000000000 +02:00
summary: 'My first impressions of the Power BI Datamarts feature announced at Microsoft Build 2022.'
image: /attachments/power-bi-datamarts4.png
---
Last week at [Microsoft Build](https://mybuild.microsoft.com/en-US/sessions/880c2556-1f8e-4e4c-89ac-77c4149cdf93?source=/schedule) the Power BI team announced Datamarts for Power BI. ([Main blog post](https://powerbi.microsoft.com/en-us/blog/announcing-public-preview-of-datamart-in-power-bi/), [Mechanics demo video](https://youtu.be/QxrUJCbcGy4))

After spending some of my weekend playing with these features I thought I would share my first impressions.

### What has been released?

In summary:

1.  A special [dataflow](https://docs.microsoft.com/en-us/power-bi/transform-model/dataflows/dataflows-introduction-self-service) that deposits data into an...
2.  automatically provisioned [Azure SQL database](https://azure.microsoft.com/en-us/products/azure-sql/database/)...
3.  which is then used as the data source for an automatically created [DirectQuery](https://docs.microsoft.com/en-us/power-bi/connect-data/desktop-use-directquery) dataset.
4.  All the above is editable via a single web UI and treated as one package.

### How does it work?

When you log onto [app.powerbi.com](http://app.powerbi.com) and go into a premium workspace (premium capacity or per user), you'll see the ability to create a new Datamart.

<figure>
<img src='/attachments/screenshot_20220529-220243_edge.jpg' width="250">
<figcaption><em>Screenshot of creating a new Datamart in a premium capacity workspace.</em></figcaption>
</figure>

This will open a new web experience where you then need to create a special kind of Power Query dataflow, except this time your dataflow will deposit the data into an automatically provisioned [Azure SQL database](https://azure.microsoft.com/en-us/products/azure-sql/database/) (an [elastic](https://docs.microsoft.com/en-us/azure/azure-sql/database/elastic-pool-overview?view=azuresql) [general purpose](https://docs.microsoft.com/en-us/azure/azure-sql/database/service-tier-general-purpose?view=azuresql) database). Also, your dataflow will only be editable within the new Datamart UI and will not be treated as a regular dataflow... Except that it very much _looks_ like a regular dataflow but now it deposits the data into an Azure SQL database instead of a datalake ([CDM format](https://docs.microsoft.com/en-us/common-data-model/model-json)).

On top of this, a read-only (and "locked down") [DirectQuery](https://docs.microsoft.com/en-us/power-bi/connect-data/desktop-use-directquery) dataset will be automatically provisioned that is based on the data inside the Azure SQL database. It is locked into sharing the same name as the Datamart. The dataset can also only be edited via the Datamart web UI (you cannot connect to the Datamart's auto-DQ dataset via external tools such as [Tabular Editor](https://tabulareditor.com/) or [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16) for editing).

The reason for the locked down approach is likely because from the Datamart web UI you can create roles that in turn will create roles for both the auto-DQ dataset as well as database roles on the Azure SQL DB.

Measures and relationships are also able to be created in the web UI, which reminds me almost of the [Azure Analysis Services Web designer](https://azure.microsoft.com/en-us/blog/introducing-the-azure-analysis-services-web-designer/?cdn=disable) UI before it got [shut down](https://azure.microsoft.com/en-us/updates/azure-analysis-services-web-designer-to-be-discontinued/?cdn=disable).  
On top of that, there's also a simple interface for running SQL queries and "visual" queries against the database.

<figure>
<img src='/attachments/screenshot_20220529-221146_edge.jpg' width="500">
<figcaption><em>Example row level security dialog.</em></figcaption>
</figure>

### So why are Power BI Datamarts such a big deal?

The big draw card in all of this is really the creation and automatic provision of Azure SQL databases, as well as being able to easily load them from Power Query.

The other items (in my opinion) are nice to have but are also use-case specific:

*   The web UI for data modelling is nice for lowering the barrier for newcomers but I'm unlikely to use it (creating measures and managing the data model will always be more efficient in Tabular Editor).
*   The auto-DQ dataset is interesting, but it steers people away from import mode and taking full advantage of the incredible [Vertipaq engine](https://www.microsoftpressstore.com/articles/article.aspx?p=2449192), probably Power BI's strongest selling point.

### Why is being able to create and load data into an Azure SQL DB so hyped?

1.  It let's you seperate the query logic layer from the data model layer.
2.  It allows for other tools to analyze the data besides Power BI (e.g. any tool that uses SQL)
3.  It gives you a good place to store additional / infrequent information, such as budgets, forecasts, and other supplemental data.

But above all, it cuts through several barriers that users face when they want to set up a reporting solution. Instead of having to get access to an [Azure subscription](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory) to set up an [Azure Data Factory](https://docs.microsoft.com/en-us/azure/data-factory/) resource and Azure SQL DB resource, you can now just do it with the click of a button. **For business users trying to come up with robust, supportable solutions, this is an absolute godsend**.

For internal IT departments it may seem daunting that users are creating their own Azure SQL databases, but in most cases this may be being done anyway by more advanced users, either through local SQL installs at the Business unit level, Access databases, inefficient SharePoint lists, or a network of Excel files as "databases".

IT departments should keep an eye on overall deployments, resource consumption, and data sensitivity, but for the most part this change is just another in a long process of IT departments transitioning away from being **_data gatekeepers_** towards being **_data gardeners_** (by helping the business "grow" their analytics capabilities).

### Why is loading from a dataflow into an Azure SQL DB better than into a datalake?

Both contain type information, both load quickly, both can be connected to via Power BI Desktop, however there are some differences:

Loading a dataflow into their standard datalake is much "cheaper" storage as it's just blob storage (CSVs + some json metadata files).

Loading a dataflow into an Azure SQL DB allows you to assign roles and row level security, as well as allows for simple connections from a variety of tools, (Tabular Editor provider datasource, SSMS, Python, and anything else that can connect to a SQL database).

### Rough Edges and improvements to be made

With all the benefits so far about this new package of features, there are still some improvements that could be made:

*   It is very much a **package** of features. If you didn't want your DirectQuery dataset, too bad you've now got one. Don't want to use the web UI and would rather use Tabular Editor? Not possible.
*   When first creating the Datamart you don't input a name, instead it gets given an autogenerated name that then needs to be renamed afterwards.
*   When you go to create a dataflow it asks if instead you would like to create a Datamart, when they're not really synonymous. Instead it would make more sense that when you make a dataflow you can choose whether to load the data into a datalake or into a database. [Support this idea here](https://ideas.powerbi.com/ideas/idea/?ideaid=af9c4467-22df-ec11-b5cf-281878a5ed2b).
*   People have reported different experiences, but trying to build an "import" dataset off of the Azure SQL DB was not workable for me. The main issue I ran into no matter what method I used was that the Azure SQL DB appeared to [throttle my queries](https://www.google.com/amp/s/blog.sqlauthority.com/2016/03/18/sql-server-introduction-sql-azure-database-throttling/%3famp) **hard**. Very occasionally I could load a 200k row table, but most times it would stall at 50k and then time out after several hours. To me this is an absolute deal breaker and removes an incentive to use the feature, so hopefully as Datamarts goes through preview these kinds of issues get ironed out.

# Final thoughts

I look forward to when the Azure SQL DB can handle me querying all of its data into my import model. Once this is possible it will be an absolute game changer.

I'm not convinced about the forcing of DirectQuery onto us or the requirements to use the web UI. Hopefully this will get removed and split out.

The story of dataflows, Datamarts, and people comparing them is confusing. To simplify this story I really think the Power BI team should position dataflows as a tool to transfer data from source systems into a storage destination (whether that be datalake storage or relational database storage).

<div align="center">
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Allowing dataflows to choose their destination simplifies the story for users:<br>- a dataflow is a way to get data from source systems into a storage destination<br>- you can pick either basic datalake storage (CSVs+manifest/typing) or richer relational database storage (Azure SQL DB)</p>&mdash; James Fancke (@jdfancke) <a href="https://twitter.com/jdfancke/status/1530825708607619073?ref_src=twsrc%5Etfw">May 29, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

Finally, I read a decent [blog post by Mim](https://datamonkeysite.com/2022/05/25/first-look-at-datamart/) that also has more specific detail behind Datamarts and recommend giving it a read.