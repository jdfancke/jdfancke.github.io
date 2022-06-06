---
layout: post
title: 'KPIs #1: KPI usage in Analysis Services, Excel Power Pivot, and Power BI 🚦'
date: 2018-12-28 12:29:54.000000000 +01:00
summary: 'An introduction to the KPIs feature in Analysis Services and how you''re able to use them in Power BI.'
mermaid: true
tag: PBI
image: /attachments/kpi-header-image.svg
---
KPIs, or Key Performance Indicators, are a common way to gauge performance against a set of objectives, and are an integral part of performance management.

Without getting too deep in jargon, the fundamental idea behind a KPI is that you are comparing some known measure against a target, and then grading the comparison.

For example, you may decide to compare your actual sales against your budget sales, and then set different percentage grades based on the performance: if sales are above 98% of budget you are "meeting budget", above 92% you are "near budget", and below 92% you are "below budget".

How many grades you use and the scores you ascribe to them should be based on business needs. Furthermore, which measures get chosen as KPIs by a business, and the process for choosing them, is outside the scope of this article. _If you're interested in this topic you can look up [Balanced Scorecards](https://en.m.wikipedia.org/wiki/Balanced_scorecard) and [Results Based Management](https://en.m.wikipedia.org/wiki/Results-based_management)._

With the above ideas in mind, let's take a look at how KPIs are implemented within Power Pivot, Analysis Services, and Power BI. However, because all three services use the same underlying Vertipaq engine and share a similar framework, let's focus on Analysis Services.

### How are KPIs implemented in Analysis Services?

To start us off, it's useful to reiterate that a KPI uses a base measure, and then compares it against a target. Because it starts off with a base measure, you can think of a KPI as an additional layer sitting on top of the existing measure.

It's no coincidence that Analysis Services treat KPIs in the same way.

#### KPI object model hierarchy

To illustrate how KPIs are treated, below is the current metadata hierarchy of the KPI object within the Analysis Services [Tabular Object Model (TOM)](https://docs.microsoft.com/en-us/bi-reference/tom/introduction-to-the-tabular-object-model-tom-in-analysis-services-amo), showing the KPI object as a child object to the Measure object.

<figure><div align='center'><img src = '/attachments/KPI-TOM-Metadata.png' width='600'></div>
<figurecaption><em>Location of the KPI object within the Tabular Object Model, from the Analysis Services Tabular Protocol.</em></figurecaption></figure>

Because the KPI object is a sub-component of the Measure object, it is worth looking at the properties of the Measure object that will be inherited.

#### Measure object properties

<figure><div align='center'><img src = '/attachments/Measure-Object-Properties.png' width='600'></div>
<figurecaption><em>Measure object properties that will be inherited by the KPI object, from the Analysis Services Tabular Protocol.</em></figurecaption></figure>
I'd like to emphasise the below measure properties:

*   Table ID
*   Name
*   FormatString
*   IsHidden
*   DisplayFolder
*   DetailRowsDefinitionID

This means that your KPI will :

1.  Share the same Table location (unless separated directly into it's own KPIs location, as in Excel).
2.  Have the same name as your underlying measure.
3.  Have the same format as your underlying measure.
4.  Be hidden if your underlying measure is hidden.
5.  Share the same display folder structure as your underlying measure (which can be weird if it is in its own special KPI "table" location, as in Excel).
6.  Have the same Detail Rows Definition.

If you would not like your KPI to share those properties, the simple solution is to create a new base measure to be used just for your KPI.

#### KPI object properties

In contrast, the below image shows the properties specified directly in the KPI object.

<figure><div align='center'><img src = '/attachments/KPI-Object-Properties.png' width='600'></div>
<figurecaption><em>KPI object properties, from the Analysis Services Tabular Protocol.</em></figurecaption></figure>

To aid those descriptions, below are two KPIs as examples: a static KPI with five grades, and a dynamic KPI with 3 grades. By "static" I mean that the goal is a fixed number, whereas a "dynamic" KPI has a goal that is based on another measure.

<figure><div align='center'><img src = '/attachments/KPI-metadata-example2.png' width='600'></div>
<figurecaption><em>KPI object property examples.</em></figurecaption></figure>

Now that you've had a quick look at the above, I'd like to a call out a few properties:

*   **TargetExpression** indicates the goal that needs to be reached. It evaluates to a number. When creating the measure you can choose a static number, or another measure to set as your target.
*   **StatusExpression** represents an automatically generated DAX expression that will evaluate to either -2, -1, 0, 1, or 2 for five levels, or -1, 0, or 1 for three levels. The number of levels used is based on the StatusGraphic chosen.
*   Within the **StatusExpression** DAX formula, if the KPI is based on a static number, these are inputted directly into the expression. If it is instead based on another measure, the expression will divide base measure by the target measure, and use the different percentage amounts as the levels. **Note**: it is not immediately obvious, but this type of division can cause an issue if the target amount has non-null amounts along one dimension, and the base amount is null along some of the same members of the same dimension. For example, if you input budget sales for next year, but your sales for next year have not yet actually occurred, you will receive an error when pivoting your KPI sales and sales budget along the Year dimension.
*   The **Description**, **TargetDescription**, and **StatusDescription** properties are editable in SSDT through the KPI editor. Curiously, a **ValueDescription** field appears to be a property editable within the KPI editor, yet is not a property that appears within the KPI object nor the KPI notations object. _Weird_.
*   **TrendGraphic**, **TrendDescription**, and **TrendExpression** are [all currently unused in Analysis Services Tabular](https://social.msdn.microsoft.com/Forums/officeapps/en-US/ef7cb3db-1132-4223-9f62-31ed0374a61b/can-you-do-kpi-trends-in-tabular) and the Power BI and Excel client tools, despite showing up in the Analysis Services Tabular Protocol (I suspect this is a hangon from Multidimensional model development, yet the feature was not fully executed in Tabular.)

### Creating a KPI

The way it works is that you select a base measure, which could be something like "Sales".

You then need to select another measure as your "target" measure, or just type in a static number. Your target measure may be something like "budget sales", whereas a static number might be "target number of workplace injuries" (which should always be 0!). In the image below I've just made a non-sensical selection of "Sales" vs a series of static targets. The icon I've selected allows for 5 status levels.

<figure><div align='center'><img src = '/attachments/KPI-SSDT-Settings-1.png' width='600'></div>
<figurecaption><em>SSDT KPI Editor.</em></figurecaption></figure>

After having filled in the basic KPI information, you can also insert some descriptions. These don't seem to appear within the Excel or Power BI client tools, but they're there if you decide to view the model metadata via DMVs. Note, however, that I couldn't find the Value Description field in the metadata (although I didn't look that hard!).

<figure><div align='center'><img src = '/attachments/KPI-SSDT-Settings-2.png' width='600'></div>
<figurecaption><em>SSDT KPI Editor (descriptions submenu)</em></figurecaption></figure>

To give you an idea of the annotations associated with KPIs, you can see the below table where two different KPIs are listed, one static and another dynamic. (Note that their IDs correspond with the KPI IDs above).

<figure><div align='center'><img src = '/attachments/KPI-Annotations-1.png' width='600'></div>
<figurecaption><em>Annotations object for KPIs.</em></figurecaption></figure>

So now that you've hopefully got an understand of how KPIs are developed and their structure within the Tabular Object Model, let's see how they're used in the Excel and Power BI client tools.

### Using KPIs in Power BI and Excel

Unfortunately, KPIs cannot currently be developed within a Power BI model ("dataset"), however, they are able to connect to a Tabular Analysis Services instance and interact with the KPIs there. You can _sort of_ develop KPIs in Power BI, but not _really_ (see [Paul Turley's rant here](https://sqlserverbi.blog/2018/03/28/how-to-add-kpi-indicators-to-a-table-in-power-bi/)). This does not apply to models developed in Excel - [Excel](https://support.office.com/en-us/article/key-performance-indicators-kpis-in-power-pivot-e653edef-8a21-40e4-9ece-83a6c8c306aa) **[can](https://support.office.com/en-us/article/key-performance-indicators-kpis-in-power-pivot-e653edef-8a21-40e4-9ece-83a6c8c306aa)** [create KPIs](https://support.office.com/en-us/article/key-performance-indicators-kpis-in-power-pivot-e653edef-8a21-40e4-9ece-83a6c8c306aa).

When you connect to a Tabular Analysis Services model in Power BI you will see that the KPIs are located within the table that they were created in (inherited the TableID property from the Measure object).

Note that if a table has all columns hidden then it will show as a little calculator in Power BI. Also note that I have a third KPI but it is located in another table based on the measure that it belongs to.

<figure><div align='center'><img src = '/attachments/KPI-in-PBI.png' width='300'></div>
<figurecaption><em>KPIs in Power BI.</em></figurecaption></figure>

In contrast to Power BI, if you connect to an Tabular Analysis Services instance via Excel you will note that the TableID of the KPI base measure is ignored, and all KPIs are put in their own virtual "table" location called "KPIs".

<figure><div align='center'><img src = '/attachments/KPI-in-Excel.png' width='300'></div>
<figurecaption><em>KPIs in Excel.</em></figurecaption></figure>

An interesting thing to note here is that you can see how the "Sales (KPI)", which is based on a "Sales" measure inherits the folder structure from the measure object. This can be annoying if you'd like your KPI to have a different folder structure. To change this just create a new measure to base your KPI on with a unique structure.

In both Power BI and Excel, when you expand the little traffic light icon you'll find a base, target, and status field.

### A few last issues

**Issue 1: Null errors when blank base measure vs non-blank target measure**

I alluded to this earlier, but there's a slightly annoying issue when using KPIs in that if you have a base measure value that returns null while your target measure returns a non-null along the same dimension members, you'll then either get an error saying that you can't divide by 0, or you'll have the much-less-helpful interaction where dropping a field onto the rows or columns area will just. do. nothing...

Don't worry, there's a relatively simple workaround for this, just create a new base measure that references your previous base measure and returns blank if your target measure is 0, else returns your base measure if the target measure is not 0.

**Issue 2: Organizing your KPIs**

Another issue you may face is that you may end up with two measures that do the same thing, one with a KPI and another without. Normally you'd think, no problem, just hide the new measure but not the KPI. However, you'll remember from earlier that the KPI inherits the IsHidden property from its parent measure object. So what can you do?

Well, one thing you can do, and this isn't a very elegant solution, but you can create a new table, perhaps using a very simple DATATABLE DAX function. Then within this table you just want to stick all of your new base measures with their KPIs. Then when you're done you should hide your table _and_ the column(s).

In Power BI you'll only see your KPI table surfacing as a little calculator with all of your KPIs underneath it.

In Excel all of your KPIs will be within the KPI virtual table object, but the table itself and the measures should be hidden (because the table is hidden).

The other benefit is that you can then have a simplified folder structure for your KPIs.

This also lets you re-organize how the KPIs will be arranged into different folders within the KPI section rather than how the regular base measures would have been (because KPIs use the same folder structure as their base measures, which can be very annoying if you have many many measures and want a different structure for the KPIs).

**Closing remarks**

Power BI (Premium) is currently on a mission to envelop all of the features included within Analysis Services Tabular, so it will be interesting to see how and if KPI development ever makes its way into Power BI! Here's to hoping that it makes its way in!

**Update (3/1/2019)**: If you'd like to continue exploring KPIs and their development using Tabular Editor, please read on to the [follow up post here]({% post_url 2019-01-03-kpi-follow-up-developing-kpis-in-power-bi-and-analysis-services-using-tabular-editor %}).

_And that's it. If you found this brief discussion on KPIs in Analysis Services Tabular useful, let me know._