---
layout: post
title: 'Why Azure Analysis Services is a great value proposition'
date: 2018-12-11 08:16:53.000000000 +01:00
summary: 'Azure Analysis Services continues to provide excellent value for money while maintaining strong administration controls and source control capability.'
mermaid: true
tag: PBI
image: '/attachments/Analysis-Service.png'
---

**Update**: after waking up this morning I realized there was just one more item I wanted to add, the suitability of AAS for multi-geo environments, which for many can be a deal breaker.

**Update 2 (18/12/2018)**: [Daniel Otykier](https://twitter.com/DOtykier?s=09) has reminded me of a further important distinction between Power BI Premium and Analysis Services, and that is the ability to use third party tools such as [Tabular Editor](https://tabulareditor.github.io/blog.html).

After reading Matthew Roche’s blog post on [Choosing Between Power BI Premium and Azure Analysis Services](https://ssbipolar.com/2018/12/09/choosing-between-power-bi-premium-and-azure-analysis-services/) I couldn’t help but feel that Azure Analysis Services (AAS) was coming off second best in a two-person race.

I know that doesn’t really make sense because whenever these kinds of discussions take place there are lots of disclaimers thrown out saying that choosing the tool should always be based on your specific requirements or what’s right for your environment, and that no single solution fits all scenarios.

But despite all of that, I still felt the urge to defend AAS as a great no-frills choice (who knew I was so loyal to it?) even though it probably wasn’t even criticized in the first place.

In fact, I'm almost a little bit nervous posting this because I think if more people "found out" about how great a value proposition AAS is, Microsoft might even have to find a way to make it a less attractive offering.

**My argument for why AAS is a great value proposition boils down to 3 different factors:**

1.  The licensing and costing is simple, flexible, and **inexpensive**.
2.  It includes all **core functionality** you would want from a data model.
3.  It is highly **accessible** for data model consumers.

Price is a fundamental aspect of assessing value and it follows that one of the main considerations when comparing AAS with Power BI Premium is cost. If we were comparing holiday cruise packages, you can think of AAS as the “standard” package and Power BI Premium as the “all-inclusive” package.

While both options will get you to where you want to go, Power BI Premium will get you there in a more comfortable manner and include a few more features that you might enjoy along the way.

**So… let’s talk pricing**

An [S1 instance of AAS](https://azure.microsoft.com/en-au/pricing/details/analysis-services/) with 100 QPUs and 25GB of RAM will set you back almost $1500 USD per month.

A [P1 instance of PBI Premium](https://powerbi.microsoft.com/en-us/calculator/) with 8 v-cores and 25GB of RAM will set you back $4,995 USD per month.

I know it’s not an apples-to-apples comparison because PBI Premium includes the ability for end-users to view PBI content within the PBI service without a license (as well as a range of other features), but it’s still interesting to see what you DO get for your S1 instance.

\[And for comparison’s sake, you can also get a B1 instance that costs $313.90 per month, and even a D1 instance that only costs $96.36 (!) per month. That’s an easy point to AAS for pricing flexibility here.\]

**Core functionality**

What do you get in the “standard” package?

It turns out you get the important stuff.

You get a _nearly_ identical, highly accessible data model that can be published to the cloud and accessed by end users or reports to provide structured or ad-hoc insights.

In fact, you even get a few things that Power BI Premium doesn’t have yet (although I’m sure it will soon), including but not limited to:

1.  [Connectivity from Excel in Office 365](https://azure.microsoft.com/en-au/blog/connect-excel-to-an-azure-analysis-services-server/) without needing to download any extra drivers. No muss, no fuss, and certainly no administrator rights needed. [The same can’t be said for Power BI](https://docs.microsoft.com/en-us/power-bi/service-analyze-in-excel).
2.  [Ragged Hierarchies](https://docs.microsoft.com/en-us/azure/analysis-services/tutorials/aas-supplemental-lesson-ragged-hierarchies). With this relatively simple feature you can create things like hierarchical Active Directory OU container explorers, organization charts, hierarchical profit and loss statements, and more!
3.  [Object Level Security](https://docs.microsoft.com/en-us/sql/analysis-services/tabular-models/object-level-security?view=sql-server-2017). With object level security you can secure both entire tables as well as individual columns within your tables. Note, however, that similar-ish security can also be achieved in Power BI through row level security, however you’ll still be showing the end user the name of the table and any columns/measures that should be secured with it. (Also note that this feature and its interaction with row level security can easily cause unintended security access violations/errors if not done carefully. Also, column level security can cause unintended security access violations even if done correctly through the use of trickily written DAX queries, which is probably why it’s not yet in Power BI).
4.  [Very simple external sharing](https://azure.microsoft.com/en-au/blog/invite-guest-users-to-your-azure-analysis-services-by-using-b2b/). Just invite your end user using their email address. If they have an Office 365 login or a Microsoft account they’ll be authorized, alternatively they’ll just create a guest account and away you go. No additional licensing needed.
5.  SQL Server Management Studio, Visual Studio source control, and [BISM Normalizer](http://bism-normalizer.com/). These are the application lifecycle management tools that help you provide a very high quality data model and enable it’s continued improvement and reliability.
6.  The ability to deploy your model to one of the many Azure regions around the world, enabling low latency and the highly performant click-clicky-draggy-droppy analysis that Christian Wade so often speaks about. For comparison, Power BI Pro and (I _believe)_ Premium, while having multi-geo for data compliance reasons, still sends queries through your default tenant region (although I'm sure this will change).
7.  Analysis Services also supports third party development tools (I'm thinking of the fantastic [Tabular Editor](https://github.com/otykier/TabularEditor/releases/tag/2.7.4)) that can connect directly to the Analysis Services instance (as well as to individual model metadata files or open projects). However, as Daniel mentions in his comment to this post, this will likely go away as XMLA endpoints become available in the coming months.

**What are you missing?**

Of all the features missing, and there are many, the biggest is that your “regular” users can’t view Power BI reports published to the PBI service. The second biggest is the lack of many data source connectors when developing your model, which _might_ be a deal breaker.

So how do users interact with this data model if they can’t access the PBI service? How do you get value out of it?

You might be thinking “you could share Power BI desktop files”, but I’ve always found that very clunky. Plus, it’s sort of the whole reason why the Power BI service exists.

So no, sharing PBI desktop files isn’t what I was thinking.

Instead, and perhaps I am naive, but I still believe that even now at the end of 2018, all data eventually makes its way into Excel. (While I love the visualizations within Power BI, the urge to get knee-deep into the numbers using Excel has never gone away, and it’s a credit to the Excel team that their tool is so immensely powerful that there are [estimates of over 100 million people using it](https://www.quora.com/How-many-people-use-Microsoft-Excel)).

Simply share a workbook that you’ve prepared containing PivotTables, PivotCharts, and cube formulae with someone who has access to the model and you’re done.

If you have Windows authentication enabled on your AAS server I don’t think the user even gets prompted for credentials when interacting with the data.

The whole experience is just so easy, so seamless, and there’s so little time spent on licensing scenarios or access control. I find that this solution works very well for small to medium sized teams. Even for sharing with external auditors or consultants this kind of solution can be highly (cost-) effective.

**Closing remarks**

...and we’ve come full circle, I’ve just realized I’m making a recommendation assuming that you, the reader, might be in a specific scenario where the value you’ll get from AAS is particularly high. I guess it really does all depend on your requirements and current environment.

I hope you got something out of this article. It’s at least made me realize that I really do think Power BI Premium is (soon-to-be) fantastic, and if you can afford it, all the more power to you! But even without Premium, don’t underestimate how effective an Analysis Services tabular model in Azure can be (when paired with Excel).

_If you notice any errors or inaccuracies please let me know and I'll make any amendments as necessary._

**Added reference:** For a more detailed comparison between the different variants of Analysis Services and Power BI, Paul Turley has [a great blog post](https://sqlserverbi.blog/2018/12/13/data-model-options-for-power-bi-solutions/) where he makes direct feature comparisons in a table format.