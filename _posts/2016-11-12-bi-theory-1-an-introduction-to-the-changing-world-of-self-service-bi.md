---
layout: post
title: 'BI Theory #1: An introduction to the changing world of self-service BI'
date: 2016-11-12 20:36:01.000000000 +01:00
summary: 'BI Theory #1: An introduction to the changing world of self-service BI'
mermaid: true
tag: PBI
image: /attachments/asset-18.png
---
**New beginnings**

Hi everyone.

It seems like the best way to start a blog is with an introduction, but instead of boring you with my life’s details let’s just jump straight into it.

![Asset 18.png](/attachments/asset-18.png)

The gist is that I’ve decided to start this blog about business intelligence and analytics in the hope that I can share some of the tools, tips, and tricks that I’ve picked up on my journey to becoming a better analyst.

Starting a blog always comes with some trepidation – the fear that not only will you lack the stamina to continue with what you’ve started, but also that it’ll be public for the world to see! Hopefully this blog marks the beginning of a great journey… but for now you’ll just have to settle with some thoughts on why I think the analytics industry is in for a shake-up.

**What _is_ business intelligence?**

In short it’s using your business’ data to help make decisions.

It’s a rather loose definition but I think it mostly fits. To narrow it down you can think of it as making **comparisons**, performing **calculations**, and looking at **trends**.

To do those things you usually follow the same steps:

1.  Collect your data
2.  Clean and/or Transform your data
3.  Analyse the data and look for insights
4.  Present your findings and allow others to explore your analysis

Those steps are all pretty straight forward so you might be asking yourself: **what’s changed, why is the industry "in for a shake-up"**?

Well, I’m glad you asked. In my opinion there are two things that have changed:

1.  The workforce has become more tech savvy and the average worker has become more confident using basic analytics tools (including the ubiquitous Excel)
2.  The tools for performing business intelligence have become both simpler to use and more effective at providing insights.

These simple changes have opened the door for what you might call "everyday analysts".

**Who are these "everyday analysts?"**

These are people that are familiar with the business and more often than not are also the same people that do the data entry. They have deep knowledge of business processes and usually have an idea of which trends would be meaningful.

Now that robust tools to perform analysis are highly accessible (i.e. built right into Excel), it’s likely that at least one of your colleagues, if not yourself, can perform the analysis directly, rather than rely on assistance external to the department.

**So what are the "new" tools?**

In short they are **Power Query** and **Power Pivot**, and as of Excel 2016 (Pro edition) they are built right into Excel. You may already know them as the new "Get & Transform" interface (Power Query) and the "Data Model" (Power Pivot). They have many names but I’ll always refer to them by their initial add-in names.

**_A quick caveat_**: I’m not saying that **_everyone_** is going to love these tools or that they’re child’s play – the learning curve is still there, although I’d say it’s much easier than learning VBA and generally provides a more robust analysis.

With that said, each department will usually have at least one or two people that love fiddling with Excel, or as Rob Collie (a popular self-service-BI evangelist and blogger) has said, they have the ["data gene"](https://www.powerpivotpro.com/2015/12/bi-everyone-tear-wall-business/). These "data gene" people will quickly come to realize that there is much more power at their fingertips than previously thought. But before I get ahead of myself let’s check out what the "old style" of reporting workflow looked like.

**The "old style" reporting workflow:**

To give you an example of the "old style" reporting workflow: if one of these sales people wanted to analyse their data to provide a report to their manager, they’d likely have to resort to doing one of the following things:

*   Use pre-built reports
*   Export their data to Excel
*   Engage a developer to create a new report

If your pre-built reports don’t suit your current needs, or you’re sick of spending time exporting data to Excel to provide a routine report, you may decide to get in touch with a developer to create a new report. There are a few reasons why this option isn’t very appealing:

*   It **requires specialised skills**
*   It’s **expensive** (especially if you have to go external to your org)
*   It’s **time consuming**
*   It’s **often not what you wanted** (especially not the first time)

**The revolution:**

Now imagine if someone on your team (if not you personally) was able to create a quick framework for new reports that, frankly, is probably better than your existing cumbersome infrastructure. No more visits to central IT, no more back and forth between your team and a developer who doesn’t "get it", and best of all, central IT at your company can instead focus on what they do best – managing security, ensuring uptime, and empowering users with software to achieve their own goals.

How many times have you been sold on the idea of new tools only to find out that they’re proprietary, expensive, and in the end everyone just presses the infamous "Export to Excel" button? Hopefully never, but the point is that the vast majority of analysis, regardless of which enterprise-style solution has been adopted by the organization, still manages to just find its way into Excel.

So why not just perform the analysis within Excel while maintaining a direct link to your database? If you think it sounds like a good idea, so does Microsoft, which is why they’ve spent the last 10+ years building the framework to include this in Excel in the form of **Power Query** and **Power Pivot**. [Read on for an introduction to these tools.]({% post_url 2017-02-20-bi-theory-2-the-microsoft-self-service-bi-toolset %})