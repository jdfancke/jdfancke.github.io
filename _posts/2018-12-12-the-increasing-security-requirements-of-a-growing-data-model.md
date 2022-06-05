---
layout: post
title: 'The increasing security requirements of a growing data model'
date: 2018-12-12 22:52:30.000000000 +01:00
summary: 'The increasing security requirements of a growing data model'
mermaid: true
tag: PBI
---
**Update**: *Power BI Premium has been updated to now include Object Level Security.*

In [my previous post about why I think AAS is a great value proposition]({% post_url 2018-12-11-why-azure-analysis-services-is-a-great-value-proposition %}), I mentioned that AAS currently has Object Level Security, whereas Power BI Premium does not. 

I'd like to now expand on this point by explaining why security is an essential part of a growing data model, and why Object Level Security is an important requirement of "larger" data models. (Note: large as in many dimensions and fact tables regardless of the row count).

I don't intend to get too technical with this post and only want to provide some basic understanding of why I think OLS is an important feature as a data model matures.

#### The data model development story

If we think of the typical "grow up" story of a data model, it may first start its life as an embedded model within Excel. At this youthful stage there is no security except for refreshing against the original data sources.

At first this isn't a problem because most likely the only person using the data model is you. But it's also likely that others may want to eventually use your model as well, meaning that they will want either a copy of the model or at least have access to it.

And why wouldn't they? The value provided by a data model is huge! Reusable calculations across different dimensions is virtually the ideal form of ad-hoc analysis.

However, the first gateway-drug into the Microsoft BI ecosystem now rears its head. For others to view your model without compatibility issues, they will need a similar version of Excel, and the best way to consistently achieve this is to use Office 365.

Many businesses now use Office 365, however, so it's not really the large hurdle that it once was. For now, let's continue this thought exercise assuming you've got your compatibility issues worked out.

If or when you later decide to send your Excel workbook to a colleague or external recipient, they will have full access to the tables loaded into the data model, but may not be able to refresh the data (with their ability to refresh determined by the data source security settings). All tables currently loaded into Power Pivot, however, can be extracted.

One of the first inherent problems of sharing your workbook in this manner (after making sure everyone can open it) is that you can't all access and edit it at the same time. For example, if it was kept on a network share you would be limited to only one user editing and saving the workbook at a time.

The interim solution is that you may decide to create copies of the model or to distribute it on a regular basis.

The problem with creating copies of the model is that whenever you update your local model you'll need to re-send it to each user, as well as now deal with version control issues.

While the above problems represent early growing pains, they aren't necessarily deal breakers.

The real deal breaker is introduced when you have to develop different variants of the same model for different users depending on their user and security requirements... Bob needs sales data, Sally needs purchasing data, and John the head honcho needs everything.

Maintaining different models for different users, plus version control, plus the soul destroying inefficiency of recreating or updating the same tables and measures between different models clearly isn't a sustainable solution. Oh, and did I mention that if your end user doesn't have access to the data source you'll need to re-send them an updated workbook with refreshed data on the regular?

#### A solution presents itself

Although it's not so much "present itself" as "parade around in plain sight".

Power BI solves the above problems (for a price of $10 USD per person per month) by providing scheduled refresh capabilities, a centralised data model that others can access, and last but not least, role based security.

#### A word on roles

Roles let you define a set of permissions for a group of users. A user may belong to one or more roles.

In Power BI, roles are limited to only include permissions for Row Level Security (RLS).

Some quick points on RLS:

1.  RLS works by defining which rows are available to that role through essentially a custom created filter statement.
2.  When you write your filter statement you are granting access to rows, so that if a user belongs to multiple roles they will receive all of the rows granted from all roles combined.
3.  New tables added to the model do not get filtered by any roles.
4.  RLS only secures row data, it does not secure against metadata such as table or column names.

That last point is important, because it sort of means that the main purpose of RLS is to secure a subset of all rows, not necessarily all of them and to secure against the entire table.

In larger models it might have been useful if when a table has all rows restricted to a user, the table and all associated measures and columns are hidden as well.

This is where Object Level Security (OLS) comes in, currently available in Analysis Services as a role permission alongside RLS.

Some quick notes on OLS:

1.  OLS works in the user interface by selecting which tables or columns should be hidden from users, however in reality you are granting access to all other tables/columns.
2.  If a user belongs to multiple roles, each with OLS settings that grant access to different tables/columns, the "grant" permissions are additive.
3.  New tables added to the model are visible to all roles unless the OLS for that role is updated.
4.  RLS secures the metadata so that the user cannot see any tables or columns that they don't have access to, as well as any measures that reference those columns. This is probably why the property name in the underlying model.bim file is called MetadataPermission.

There are some [complications with mixing roles and permission settings](https://blogs.msdn.microsoft.com/analysisservices/2017/04/19/whats-new-in-sql-server-2017-ctp-2-0-for-analysis-services/), however my philosophy on using roles currently isn't affected by this too much.

So what's my philosophy on roles? To keep it simple, it's 1 user per role.

Why? Well, because working out effective permissions is difficult:

*   The UI for OLS is unintuitive as it almost implies that you're denying access instead of granting access, which adds to the difficulty of figuring out effective permissions.
*   Comparing users permissions when the user only belongs to one role is much easier, and is generally how most people approach security access. (e.g. give the new guy the same security as the person they replaced / give everyone on this team the same access).
*   You can then then also decide to name a role after a group of users that it will include.
*   On top of that, I generally prefer to add single users rather than groups of users, just because it reduces the obfuscation even more and means that people aren't accidentally in more than one group.

#### The maturing data model

Coming back to our maturing data model, you may face growing pains due to the increasing licensing costs of sharing a PBI model to ever increasing numbers of users needing a Pro license, or it could be that you're starting to get so many tables in your model and don't like that users can see tables and measures that "for some reason" return blanks when they query their co-worker's salary information.

In that case, there's one more step to find your model a permanent home, and currently it's to move the model to Analysis Services where OLS can take care of those problems, as well as provide you with a few more fancy features like partitions and perspectives (if you splurge for Enterprise on prem or an "S" instance in Azure).

Until Power BI Premium receives OLS, Analysis Services provides a compelling reason why it should be a model's final destination.