---
layout: post
title: 'BI Theory #4: Where to get Power Query and Power Pivot'
date: 2017-03-02 07:30:29.000000000 +01:00
summary: 'BI Theory #4: Where to get Power Query and Power Pivot'
mermaid: true
tag: PBI
image: '/attachments/asset-12.png'
---
Now that you know the basic idea behind Power Query and Power Pivot it's time to start downloading them! (if you have no idea what I'm talking about you may want to read my earlier posts here [[1]({% post_url 2016-11-12-bi-theory-1-an-introduction-to-the-changing-world-of-self-service-bi %})], [[2]({% post_url 2017-02-20-bi-theory-2-the-microsoft-self-service-bi-toolset %})], [[3]({% post_url 2017-02-28-bi-theory-3-m-vs-dax %})].

<div align='center'><img src = '/attachments/asset-12.png' width='400'></div>

The thing to remember is that **Power Query** and **Power Pivot** aren’t just Excel features – they pop up in a few places and if you know how to use them in one application you can easily use them in another.

See below for some housekeeping on where to get access to Power Query and Power Pivot.
<table>
<tr>
<td><strong>Application</strong></td>
<td><strong>Power Query</strong></td>
<td><strong>Power Pivot</strong></td>
</tr>
<tr>
<td>Excel 2010 (Windows)</td>
<td><a href="https://www.microsoft.com/en-au/download/details.aspx?id=39379">Download Add-In</a> (“Professional Plus” Excel Only)</td>
<td><a href="https://www.microsoft.com/en-us/download/details.aspx?id=43348">Download Add-In</a><p></p>
<p>(All Excel versions)</p></td>
</tr>
<tr>
<td>Excel 2013 (Windows)</td>
<td><a href="https://www.microsoft.com/en-au/download/details.aspx?id=39379">Download Add-In</a> (All Excel Versions)</td>
<td><a href="https://support.office.com/en-us/article/Start-the-Power-Pivot-in-Microsoft-Excel-add-in-a891a66d-36e3-43fc-81e8-fc4798f39ea8">Default Add-In</a><p></p>
<p>(Pro Excel only)</p></td>
</tr>
<tr>
<td>Excel 2016 (Windows)</td>
<td>Included (called “Get &amp; Transform”)</td>
<td><a href="https://support.office.com/en-us/article/Start-the-Power-Pivot-in-Microsoft-Excel-add-in-a891a66d-36e3-43fc-81e8-fc4798f39ea8">Default Add-In</a><p></p>
<p>(Pro Excel + Standalone Excel only)</p></td>
</tr>
<tr>
<td><a href="https://powerbi.microsoft.com/en-us/downloads/">Power BI Desktop</a></td>
<td colspan="2">Included</td>
</tr>
<tr>
<td><a href="https://www.microsoft.com/en-au/sql-server/sql-server-downloads">MS SQL Server</a><p></p>
<p><a href="https://www.visualstudio.com/downloads/">Visual Studio</a></p></td>
<td colspan="2">Add-In (<a href="https://docs.microsoft.com/en-us/sql/ssdt/download-sql-server-data-tools-ssdt">SQL Server Data Tools</a>&nbsp;or <a href="https://marketplace.visualstudio.com/items?itemName=ProBITools.MicrosoftAnalysisServicesModelingProjects">Analysis Services Project</a>)</td>
</tr>
</table>

The best experience for using Power Query and Power Pivot is within Power BI as it has regular monthly updates with the newest features.  
Standalone Excel means the [single-application purchase of Excel](https://www.microsoftstore.com/store/msusa/en_US/pdp/Excel-2016/productID.323021400) without the rest of the Office suite.**Note:** “Pro” Excel refers to Excel that comes via the Professional Plus package installation of the MS Office suite included in subscriptions of Office 365 ([ProPlus](https://products.office.com/en-au/business/office-365-proplus-business-software), [E3](https://products.office.com/en-au/business/office-365-enterprise-e3-business-software), [E4](https://products.office.com/en-au/business/office-365-enterprise-e4-business-software), [E5](https://products.office.com/en-au/business/office-365-enterprise-e5-business-software), [Education E5](https://products.office.com/en-au/academic/compare-office-365-education-plans)) as well as the [Professional Plus 2016](https://www.microsoftstore.com/store/msusa/en_US/pdp/Office-Professional-2016/productID.323023800) non-subscription installation.

For Excel the best option is via Office 365, with updates being delayed by 1-2 months from Power BI [depending on your Office 365 channel](https://technet.microsoft.com/en-us/library/mt455210.aspx). I’m currently on an Education E5 subscription through my university (current channel) and an E5 subscription through work (deferred channel).

In future it's expected that Power Query will also be available in SQL Server Reporting Services (to allow bypassing of a SSAS tabular model), as well as SSIS, and perhaps even other applications such as Access and Visio.

Now that you're all set to go see here (coming soon) to get started with building your first model, or read on [here]({% post_url 2017-11-04-bi-theory-5-what-are-databases %}) to learn about the basics of databases and tables.