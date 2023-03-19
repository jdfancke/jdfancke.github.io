---
layout: post
title: Creating a date table in Power Query
date: 2018-12-25 09:08
tags: ["Power Query"]
summary: 'How to make a date table in Power Query’s M language and what fields I think are important to include.'
mermaid: true
comments: true
---

_I use "columns" and "fields" interchangeably in this post._

There are many posts out there showing how to create a date table and I believe this is because it's a common problem that almost everyone encounters when first making a data model in Power BI, Excel, or Analysis Services.

By "common problem" I mean that it's the one dimension that almost every data model shares, and fitting it to your data model can sometimes be a bit tricky.

You can create date tables in <a href="https://stackoverflow.com/questions/5635594/how-to-create-a-calendar-table-for-100-years-in-sql">SQL</a>, <a href="https://www.sqlbi.com/articles/reference-date-table-in-dax-and-power-bi/">DAX</a>, <a href="https://exceleratorbi.com.au/build-reusable-calendar-table-power-query/">M</a>, and many other programming languages.

In this post I want to show you how you can make a date table in Power Query's M language, as well as explain what fields I think are important and how you can create them.

**Note**: There's info out there about whether you should join your fact table dates to your date table on date columns or date-integer columns, as well as whether you should "mark as date table". For now, however, these topics are not within the scope of this post. _I join my tables on a date column and I do not mark my custom date tables as date. A video by Patrick Leblanc __<a href="https://youtu.be/i8aKjGZd5kY">here</a>__ can start you down that topic._

### What makes a good date table?
This can almost be rephrased as "what columns should I include in a date table". I'll list what columns I think are important further below, but in the mean time, here are some other characteristics of a good date table:

**Good characteristic 1: Only include columns that you know you will use, or think you will use at least once.**

My philosophy when making a date table (and data modelling in general) is: _"I want what I want and nothing that I don't"_, meaning that I only include columns that I know I will use, albeit if infrequently. In this regard I generally start off with a minimalist attitude and add more columns as I go.

If you do find yourself with an overly cluttered model, you can always use <a href="https://github.com/otykier/TabularEditor/wiki/Best-Practice-Analyzer#finding-unused-objects">Tabular Editor and the Best Practice Analyzer feature</a> or <a href="https://github.com/otykier/TabularEditor/wiki/Roadmap#ui-for-showing-object-dependencies">view dependencies</a> to see where you can prune your model. **(Small update**: <a href="https://www.reddit.com/user/Data_cruncher">u/data_cruncher</a> has suggested I also include <a href="https://www.sqlbi.com/tools/vertipaq-analyzer/">Vertipaq Analyzer</a>, a handy tool for quickly analyzing data models in Excel - thanks for the tip! I might even do a future post dedicated to this tool and DMVs).

Kasper's post <a href="https://www.kasperonbi.com/determine-columns-you-dont-need-using-dmvs-in-power-bi">here</a> also explains how and why you should remove unused columns. Note that a few extra columns on a dimension table such as this date table will make barely any difference to the size of your model, so from an optimisation point of view having a few extra columns is not a big deal.

If you add too many columns to your model, or individual tables, it can quickly become cluttered, and before you know it you might have difficulty understanding your model.

This leads me to my next "good characteristic" of a date table.

**Good characteristic 2: Make sure that you understand your date table.**

If you don't understand how it was made or what the columns mean, it can be confusing to add new columns in the future that weren't originally included. This is part of the reason why I don't use the <a href="https://www.sqlbi.com/articles/reference-date-table-in-dax-and-power-bi/">fantastic DAX date table available on SQLBI</a> (however I do look to it for inspiration!).

While the "width" of your date table is very important and you should always try to avoid unnecessary columns, it is also somewhat important to consider the "length" of your date table, which leads to...

**Good characteristic 3: Make sure the range of dates is appropriate for your data model.**

By this I mean make sure that you include enough historical and future date rows in your date table so that you can be comfortable that each date in all of your fact tables can find a corresponding date in your date table.

It's often suggested to use the maximum date from a particular fact table to determine a dynamic maximum date on your date table.

While this dynamic maximum can be done relatively easily in DAX, and perhaps slightly less easily in M (and perhaps even less easily when using M in Analysis Services unless you're using expressions), I, however, think this is unnecessary and can create complications if you decide to later add new fact tables that have dates in the future (such as budget tables or 5-year plans).

_If the reason you're linking your date table's maximum date to a fact table is to avoid your "Sales YTD" or other measures running into future periods, you can just filter the measure so that it must be before the current date._

But enough about "good characteristics" of a date table..

### Which fields should you include?

I've organized the fields into the 3 groups below:

- Standard descriptors
- Combined descriptors
- Indexes

**Standard descriptors** provide common ways of describing your data. Generally included in this category are:

- Date
- Day Name
- Year
- Half-year
- Trimester
- Quarter
- Month
- Week
- IsWeekend
- IsHoliday

**Combined descriptors** provide a way to uniquely filter rows in your report. For example, to exclude December 2018 from your report without excluding December 2017 it is handy to have a "year-month" field to use as your filter (as filtering in just December will remove all occurances of December, regardless of year).

- Year - Quarter
- Year - Month
- Year - Week
- Year - Month - Week
- Date - Day Name

**Indexes** provide a way to make comparisons of your data for YTD and other period based measures.

- Week index
- Month index
- Quarter index

### Building the date table
**Step 1: Parameters**
Begin your table by declaring a start date, end date, and optionally the culture to use for month names later on.
{% raw %}
```fsharp
let
// Enter start and end dates (format YYYY,MM,DD)
StartDate = #date(2010, 1, 1),
EndDate = #date(2029, 12, 31),
Culture = "en-US", // Examples: en-US, nl-NL, zh-ZH, etc.
```
{% endraw %}
Next you want to generate the date list based on your start and end dates.
{% raw %}
```fsharp
// Generate date list from variables above (start and end years), convert list to table, rename, convert to dates
DayCount = Duration.Days(Duration.From(EndDate - StartDate)) + 1,
GenerateDates = List.Dates(StartDate, DayCount, #duration(1, 0, 0, 0)),
TableConversion = Table.FromList(GenerateDates, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
RenamedColumns = Table.RenameColumns(TableConversion,{{"Column1", "Date"}}),
DateTypeChange = Table.TransformColumnTypes(RenamedColumns,{{"Date", type date}}),
```
{% endraw %}
**Step 2: Inserting the year and week**
Once you have an ordered list of dates, you will want to insert the year and week.

Getting the week number can be tricky if you want to use the ISO standard week number or similar variants. If you don't need an ISO week then you're in luck and can just use the Date.WeekOfYear function, but for this post I'll include how to get an ISO week.

For reference, here's an old <a href="https://social.technet.microsoft.com/Forums/en-US/2a85c825-c75b-4622-98a1-25547dec49c8/enhancement-request-iso-week-number?forum=powerquery">feature request</a> that explains the issue and hopes for an update to the Date.WeekOfYear function to accept arguments similar to the Excel WEEKNUM function (which can correctly calculate the ISO week number). Interestingly, this has been a feature request for quite a while and is not yet implemented. Calling on <a href="https://twitter.com/mllopis">Miguel Llopsis</a> if he may know of any plans on the drawing board! FYI you can vote for an update to the Date.WeekOfYear function on <a href="https://ideas.powerbi.com/forums/265200-power-bi-ideas/suggestions/13507155-iso-week-number-option-on-date-weekofyear-function">the Power BI Ideas forum here</a>.
To get your ISO week, use the following code:
{% raw %}
```fsharp
// Insert a standard year, insert a reference date, insert ISO8601 week number using ref date
InsertYear = Table.AddColumn(DateTypeChange, "Year", each Date.Year([Date]), type number),
InsertRefDate = Table.AddColumn(InsertYear , "RefDate", each #date([Year],1,3)),
InsertISOWeek = Table.AddColumn(InsertRefDate , "ISOWeek", each
    if Number.IntegerDivide(Duration.Days([Date]-[RefDate])+Date.DayOfWeek([RefDate],0)+7,7) < 1 then 1
    else Number.IntegerDivide(Duration.Days([Date]-[RefDate])+Date.DayOfWeek([RefDate],0)+7,7), type number),
```
{% endraw %}
_Note if you wanted an exact ISO year you can use the below (although I don't use this because financial year-ends are the same date each year):_

```fsharp
InsertYear = Table.AddColumn(DateTypeChange, "ISOYear", each Date.Year(Date.AddDays([Date],3-Date.DayOfWeek([Date],1))), type number),
```

The way that getting the ISO week works is that it adds up the difference between the current date and the reference date, adds on the day of the week of the reference date, and then adds another 7. (For a correct ISO week beginning on Monday you add 6 and use 0 for your Date.DayOfWeek argument), for a week beginning Sunday you add 7 and use 1 for your Date.DayOfWeek argument).

You then take this number and divide it by 7, ignoring any remainder.

Worked examples are always very useful, so let's use the 9th and 10th of January 2016. In these examples we're saying that the week starts on a Sunday (not strictly ISO).

<pre>
<FONT COLOR="BLUE">Example 1: ISO Week of 9 January 2016</FONT>
<FONT COLOR="GREY">1:</FONT> 9/1/2016 - 3/1/2016 = 6 <FONT COLOR="GREY">//Get difference between current and reference date</FONT>
<FONT COLOR="GREY">2:</FONT> 3/1/2016 is a Sunday = 0 <FONT COLOR="GREY">//Reference date is a Sunday (0 to 6, Sunday to Saturday)</FONT>
<FONT COLOR="GREY">3:</FONT> Add 7 = 7
<FONT COLOR="GREY">4:</FONT> 6+0+7=13 <FONT COLOR="GREY">//Add results of 3 previous calculations</FONT>
<FONT COLOR="GREY">5:</FONT> 13/7 = <FONT COLOR="RED">1</FONT> (ignore the remainder), Saturday 9 January 2016 is in week <FONT COLOR="RED">1</FONT>
 
<FONT COLOR="BLUE">Example 2: ISO Week of 10 January 2016</FONT>
<FONT COLOR="GREY">1:</FONT> 10/1/2016 - 3/1/2016 = 7
<FONT COLOR="GREY">2:</FONT> 3/1/2016 is a Sunday = 0
<FONT COLOR="GREY">3:</FONT> Add 7 = 7
<FONT COLOR="GREY">4:</FONT> 7+0+7=14
<FONT COLOR="GREY">5:</FONT> 14/7 = <FONT COLOR="RED">2</FONT> (ignore the remainder), Sunday 10 January 2016 is in week <FONT COLOR="RED">2</FONT>
</pre>

**Step 3: Inserting the months and quarters**
After getting your weeks sorted, you can then insert the months. If you want calendar months then you can just use Date.Month, otherwise you can use the below series of if statements for how to code for 4-4-5 months.
{% raw %}
```fsharp
// Insert the Month Number based on the Week numbers, change it to the number type
InsertISOMonth =Table.AddColumn(InsertISOWeek, "ISOMonth", each
    if [ISOWeek]<= 4 then "1" else
    if [ISOWeek]<= 8 then "2" else
    if [ISOWeek]<= 13 then "3" else
    if [ISOWeek]<= 17 then "4" else
    if [ISOWeek]<= 21 then "5" else
    if [ISOWeek]<= 26 then "6" else
    if [ISOWeek]<= 30 then "7" else
    if [ISOWeek]<= 34 then "8" else
    if [ISOWeek]<= 39 then "9" else
    if [ISOWeek]<= 43 then "10" else
    if [ISOWeek]<= 47 then "11" else "12"),
ISOMonthTypeChange = Table.TransformColumnTypes(InsertISOMonth,{{"ISOMonth", type number}}),
```
{% endraw %}
After you've got your month numbers you can insert the corresponding month name. Note that if you add the "culture" parameter defined earlier it will change the language of the month name.
{% raw %}
```fsharp
// Insert the month name that corresponds with the ISOMonth, then reorder the columns
InsertISOMonthName = Table.AddColumn(ISOMonthTypeChange, "ISOMonthName", each Date.ToText(#date([Year],[ISOMonth],1), "MMMM", Culture)),
InsertISOQuarter = Table.AddColumn(InsertISOMonthName, "ISOQuarter", each
    if [ISOMonth] <= 3 then "Q1" else
    if [ISOMonth] <= 6 then "Q2" else
    if [ISOMonth] <= 9 then "Q3" else "Q4"),
ISOQuarterTypeChange = Table.TransformColumnTypes(InsertISOQuarter,{{"ISOQuarter", type text}}),
```
{% endraw %}
**Step 4: Insert the combined descriptors**
For now I'll just be adding in some simple combined descriptors, but you may want to add others or different variants.
{% raw %}
```fsharp
// Insert "YYYY-WWW" column, "YYY-MMM" column, remove the reference date
InsertYrWk = Table.AddColumn(ISOQuarterTypeChange, "YrWk", each Number.ToText([Year]) &amp; Number.ToText([ISOWeek],"-W00")),
InsertYrMn = Table.AddColumn(InsertYrWk,"YrMn", each Number.ToText([Year]) &amp; Date.ToText(#date([Year],[ISOMonth],1), "-MMM", Culture)),
RemoveRefDate = Table.RemoveColumns(InsertYrMn ,{"RefDate"}),
```
{% endraw %}
**Step 5: Insert the indexes**
One of the final steps is to add in the indexes for use in time intelligence measures.
{% raw %}
```fsharp
// Create ISOWeek Index Table
ISOWeekIndexTableSelectColumns = Table.SelectColumns(RemoveRefDate,{"Year", "ISOWeek"}),
ISOWeekIndexTableSelectDistinct = Table.Distinct(ISOWeekIndexTableSelectColumns),
ISOWeekIndexTable = Table.AddIndexColumn(ISOWeekIndexTableSelectDistinct, "ISOWeekIndex", 1, 1),
// Create ISOMonth Index Table
ISOMonthIndexTableSelectColumns = Table.SelectColumns(RemoveRefDate,{"Year", "ISOMonth"}),
ISOMonthIndexTableSelectDistinct = Table.Distinct(ISOMonthIndexTableSelectColumns),
ISOMonthIndexTable = Table.AddIndexColumn(ISOMonthIndexTableSelectDistinct, "ISOMonthIndex", 1, 1),
// Create ISOQuarter Index Table
ISOQuarterIndexTableSelectColumns = Table.SelectColumns(RemoveRefDate,{"Year", "ISOQuarter"}),
ISOQuarterIndexTableSelectDistinct = Table.Distinct(ISOQuarterIndexTableSelectColumns),
ISOQuarterIndexTable = Table.AddIndexColumn(ISOQuarterIndexTableSelectDistinct, "ISOQuarterIndex", 1, 1),
```
{% endraw %}
The way that this works is that you're grabbing your most recent step, selecting the year and other column that you want to make an index on, getting their distinct values, and then creating an index column. _Remember, just because the query editor puts each step after the other doesn't mean you can't re-use steps for other purposes._

After creating your index tables you can then join them into your main table:
{% raw %}
```fsharp
// Join Index Tables
MergedISOWeekIndexTable = Table.NestedJoin(RemoveRefDate,{"Year", "ISOWeek"},ISOWeekIndexTable,{"Year", "ISOWeek"},"NewColumn",JoinKind.LeftOuter),
ExpandMergedISOWeekIndexTable = Table.ExpandTableColumn(MergedISOWeekIndexTable, "NewColumn", {"ISOWeekIndex"}, {"ISOWeekIndex"}),
MergedISOMonthIndexTable = Table.NestedJoin(ExpandMergedISOWeekIndexTable,{"Year", "ISOMonth"},ISOMonthIndexTable,{"Year", "ISOMonth"},"NewColumn",JoinKind.LeftOuter),
ExpandMergedISOMonthIndexTable = Table.ExpandTableColumn(MergedISOMonthIndexTable, "NewColumn", {"ISOMonthIndex"}, {"ISOMonthIndex"}),
MergedISOQuarterIndexTable = Table.NestedJoin(ExpandMergedISOMonthIndexTable,{"Year", "ISOQuarter"},ISOQuarterIndexTable,{"Year", "ISOQuarter"},"NewColumn",JoinKind.LeftOuter),
ExpandMergedISOQuarterIndexTable = Table.ExpandTableColumn(MergedISOQuarterIndexTable, "NewColumn", {"ISOQuarterIndex"}, {"ISOQuarterIndex"}),
```
{% endraw %}
Finally, you may want to do a reorder of your columns and a type change.
{% raw %}
```fsharp
ReorderColumns = Table.ReorderColumns(ExpandMergedISOQuarterIndexTable,{"Date", "Year", "ISOQuarter", "ISOQuarterIndex", "ISOMonth", "ISOMonthName", "ISOMonthIndex", "ISOWeek", "ISOWeekIndex", "YrMn", "YrWk"}),
TypeChange = Table.TransformColumnTypes(ReorderColumns,{{"Year", Int64.Type}, {"ISOQuarterIndex", Int64.Type}, {"ISOMonth", Int64.Type}, {"ISOMonthIndex", Int64.Type}, {"ISOWeek", Int64.Type}, {"ISOWeekIndex", Int64.Type}})
in TypeChange
```
{% endraw %}
**All the code in one block can be found below:**
{% raw %}
```fsharp
let
// Enter start and end dates (format YYYY,MM,DD)
StartDate = #date(2010, 1, 1),
EndDate = #date(2029, 12, 31),
Culture = "en-US", // Examples: en-US, nl-NL, zh-ZH, etc.

// Generate date list from variables above (start and end years), convert list to table, rename, convert to dates
DayCount = Duration.Days(Duration.From(EndDate - StartDate)) + 1,
GenerateDates = List.Dates(StartDate, DayCount, #duration(1, 0, 0, 0)),
TableConversion = Table.FromList(GenerateDates, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
RenamedColumns = Table.RenameColumns(TableConversion,{{"Column1", "Date"}}),
DateTypeChange = Table.TransformColumnTypes(RenamedColumns,{{"Date", type date}}),

// Insert a standard year, insert a reference date, insert ISO8601 week number using ref date
InsertYear = Table.AddColumn(DateTypeChange, "Year", each Date.Year([Date]), type number),
InsertRefDate = Table.AddColumn(InsertYear , "RefDate", each #date([Year],1,3)),
InsertISOWeek = Table.AddColumn(InsertRefDate , "ISOWeek", each
    if Number.IntegerDivide(Duration.Days([Date]-[RefDate])+Date.DayOfWeek([RefDate],0)+7,7) < 1 then 1
    else Number.IntegerDivide(Duration.Days([Date]-[RefDate])+Date.DayOfWeek([RefDate],0)+7,7), type number),

// Insert the Month Number based on the Week numbers, change it to the number type
InsertISOMonth =Table.AddColumn(InsertISOWeek, "ISOMonth", each
    if [ISOWeek]<= 4 then "1" else
    if [ISOWeek]<= 8 then "2" else
    if [ISOWeek]<= 13 then "3" else
    if [ISOWeek]<= 17 then "4" else
    if [ISOWeek]<= 21 then "5" else
    if [ISOWeek]<= 26 then "6" else
    if [ISOWeek]<= 30 then "7" else
    if [ISOWeek]<= 34 then "8" else
    if [ISOWeek]<= 39 then "9" else
    if [ISOWeek]<= 43 then "10" else
    if [ISOWeek]<= 47 then "11" else "12"),
ISOMonthTypeChange = Table.TransformColumnTypes(InsertISOMonth,{{"ISOMonth", type number}}),

// Insert the month name that corresponds with the ISOMonth, then reorder the columns
InsertISOMonthName = Table.AddColumn(ISOMonthTypeChange, "ISOMonthName", each Date.ToText(#date([Year],[ISOMonth],1), "MMMM", Culture)),
InsertISOQuarter = Table.AddColumn(InsertISOMonthName, "ISOQuarter", each
    if [ISOMonth] <= 3 then "Q1" else
    if [ISOMonth] <= 6 then "Q2" else
    if [ISOMonth] <= 9 then "Q3" else "Q4"),
ISOQuarterTypeChange = Table.TransformColumnTypes(InsertISOQuarter,{{"ISOQuarter", type text}}),

// Insert "YYYY-WWW" column, "YYY-MMM" column, remove the reference date
InsertYrWk = Table.AddColumn(ISOQuarterTypeChange, "YrWk", each Number.ToText([Year]) &amp; Number.ToText([ISOWeek],"-W00")),
InsertYrMn = Table.AddColumn(InsertYrWk,"YrMn", each Number.ToText([Year]) &amp; Date.ToText(#date([Year],[ISOMonth],1), "-MMM", Culture)),
RemoveRefDate = Table.RemoveColumns(InsertYrMn ,{"RefDate"}),

// Create ISOWeek Index Table
ISOWeekIndexTableSelectColumns = Table.SelectColumns(RemoveRefDate,{"Year", "ISOWeek"}),
ISOWeekIndexTableSelectDistinct = Table.Distinct(ISOWeekIndexTableSelectColumns),
ISOWeekIndexTable = Table.AddIndexColumn(ISOWeekIndexTableSelectDistinct, "ISOWeekIndex", 1, 1),

// Create ISOMonth Index Table
ISOMonthIndexTableSelectColumns = Table.SelectColumns(RemoveRefDate,{"Year", "ISOMonth"}),
ISOMonthIndexTableSelectDistinct = Table.Distinct(ISOMonthIndexTableSelectColumns),
ISOMonthIndexTable = Table.AddIndexColumn(ISOMonthIndexTableSelectDistinct, "ISOMonthIndex", 1, 1),

// Create ISOQuarter Index Table
ISOQuarterIndexTableSelectColumns = Table.SelectColumns(RemoveRefDate,{"Year", "ISOQuarter"}),
ISOQuarterIndexTableSelectDistinct = Table.Distinct(ISOQuarterIndexTableSelectColumns),
ISOQuarterIndexTable = Table.AddIndexColumn(ISOQuarterIndexTableSelectDistinct, "ISOQuarterIndex", 1, 1),

// Join Index Tables
MergedISOWeekIndexTable = Table.NestedJoin(RemoveRefDate,{"Year", "ISOWeek"},ISOWeekIndexTable,{"Year", "ISOWeek"},"NewColumn",JoinKind.LeftOuter),
ExpandMergedISOWeekIndexTable = Table.ExpandTableColumn(MergedISOWeekIndexTable, "NewColumn", {"ISOWeekIndex"}, {"ISOWeekIndex"}),
MergedISOMonthIndexTable = Table.NestedJoin(ExpandMergedISOWeekIndexTable,{"Year", "ISOMonth"},ISOMonthIndexTable,{"Year", "ISOMonth"},"NewColumn",JoinKind.LeftOuter),
ExpandMergedISOMonthIndexTable = Table.ExpandTableColumn(MergedISOMonthIndexTable, "NewColumn", {"ISOMonthIndex"}, {"ISOMonthIndex"}),
MergedISOQuarterIndexTable = Table.NestedJoin(ExpandMergedISOMonthIndexTable,{"Year", "ISOQuarter"},ISOQuarterIndexTable,{"Year", "ISOQuarter"},"NewColumn",JoinKind.LeftOuter),
ExpandMergedISOQuarterIndexTable = Table.ExpandTableColumn(MergedISOQuarterIndexTable, "NewColumn", {"ISOQuarterIndex"}, {"ISOQuarterIndex"}),
ReorderColumns = Table.ReorderColumns(ExpandMergedISOQuarterIndexTable,{"Date", "Year", "ISOQuarter", "ISOQuarterIndex", "ISOMonth", "ISOMonthName", "ISOMonthIndex", "ISOWeek", "ISOWeekIndex", "YrMn", "YrWk"}),
TypeChange = Table.TransformColumnTypes(ReorderColumns,{{"Year", Int64.Type}, {"ISOQuarterIndex", Int64.Type}, {"ISOMonth", Int64.Type}, {"ISOMonthIndex", Int64.Type}, {"ISOWeek", Int64.Type}, {"ISOWeekIndex", Int64.Type}})

in 
    TypeChange
```
{% endraw %}
I hope you've found this waffle-y post useful. If you find any errors or think I've missed something, please let me know and I'll add it to this post!
