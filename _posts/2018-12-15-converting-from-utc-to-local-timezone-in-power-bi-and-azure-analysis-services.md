---
layout: post
title: 'Converting from UTC to Local Timezone in Power BI and Azure Analysis Services'
date: 2018-12-15 07:25:28.000000000 +01:00
summary: 'Converting from UTC to Local Timezone in Power BI and Azure Analysis Services is a common issue due to the servers using UTC time, this post explains how to work around this issue.'
mermaid: true
tag: PBI
---
This post is a quick response / supplement to Kasper's post "[Show the refresh date\\time in a Power BI report and dashboard](https://www.kasperonbi.com/show-the-refresh-datetime-in-a-power-bi-report-and-dash)".

Currently there is a small issue when using the DateTime.LocalNow() (and other similar functions) within the Power BI Service or Azure Analysis Services when processing a refresh of data.

The issue is that the server will record a refresh time using Universal Coordinated Time (UTC), rather than the specific time zone that the server is located in.

(Note that UTC is the same as Greenwich Mean Time, GMT).

There are some great resources out there for solving this problem (particularly [this post by Reza Rad](http://radacad.com/solving-dax-time-zone-issue-in-power-bi), but I thought I’d add in one more hard-coded solution for those tricky times where you can’t or don’t want to connect to a web source and are operating in a time zone that has daylight savings.

The “best” solutions often say that you should query a web service or grab the correct time for your location from a web site. Although I’d counter that by asking:

*   Do you really want to add yet another query to your model?
*   How about a query that’s dependent upon a potentially unreliable external web source?
*   What do you do if you’re using Analysis Services and there is no “from web” connector available?
*   Is it actually that hard to just code in the daylight savings time difference using an M formula in Power Query?

The answers to the above are: No, no, you’re sh\*t out of luck, and no.

**When do daylight savings occur?**

It’s not borne from a crystal ball, and neither are there specialists throwing darts at a calendar. When you look at [when daylight savings changes actually occur](https://en.wikipedia.org/wiki/Daylight_saving_time_by_country), they’re pretty much universally on a Sunday and also usually on a particular week of the month, with “the first Sunday of x month” popping up quite often.

Because I’m from Melbourne (Australia, not Florida) and we fit into a common category, I’ll try to explain the M code for calculating from the server’s local datetime (aka UTC time) to Melbourne time.

Firstly, you want to grab the current year.

```fsharp
CurrentYear = Date.Year(DateTime.LocalNow())
```

Secondly, you need to grab the daylight savings dates.

In Melbourne, daylight savings is defined by the first Sunday of April and the first Sunday of October. By working from the inside out of the formula outwards it is a bit easier to understand: you first check the starting-day-of-the-week (i.e. Sunday) of the first day in April. If the 1st of April is a Monday, the Date.StartOfWeek function will return Sunday 31st March. If the 1st of April is a Tuesday, the Date.StartOfWeek function will return Sunday 30th March. To get the first Sunday you then just use the Date.AddDays function to add 7 days IF the date previously returned was in March (i.e. the 3rd month).

```fsharp
FirstSundayInApril = if Date.Month(Date.StartOfWeek(#date(CurrentYear,4,1),Day.Sunday)) = 3 then Date.AddDays(Date.StartOfWeek(#date(CurrentYear,4,1),Day.Sunday),7) else Date.StartOfWeek(#date(CurrentYear,4,1),Day.Sunday)

FirstSundayInOctober = if Date.Month(Date.StartOfWeek(#date(CurrentYear,10,1),Day.Sunday)) = 9 then Date.AddDays(Date.StartOfWeek(#date(CurrentYear,10,1),Day.Sunday),7) else Date.StartOfWeek(#date(CurrentYear,10,1),Day.Sunday)
```

Finally, you want to check to see if the current date is between the April and October daylight savings dates.

If it is between the dates it should be 10 hours ahead of UTC, and if it’s not it should be 11 hours ahead of UTC.

```fsharp
CurrentDate = Date.From(DateTime.LocalNow()),

AustralianTime = if CurrentDate >= FirstSundayInApril and CurrentDate < FirstSundayInOctober then DateTime.LocalNow() + #duration(0,10,0,0) else DateTime.LocalNow() + #duration(0,11,0,0)
```

The full code is below:
```fsharp
CurrentYear = Date.Year(DateTime.LocalNow()),

FirstSundayInApril = if Date.Month(Date.StartOfWeek(#date(CurrentYear,4,1),Day.Sunday)) = 3 then Date.AddDays(Date.StartOfWeek(#date(CurrentYear,4,1),Day.Sunday),7) else Date.StartOfWeek(#date(CurrentYear,4,1),Day.Sunday),

FirstSundayInOctober = if Date.Month(Date.StartOfWeek(#date(CurrentYear,10,1),Day.Sunday)) = 9 then Date.AddDays(Date.StartOfWeek(#date(CurrentYear,10,1),Day.Sunday),7) else Date.StartOfWeek(#date(CurrentYear,10,1),Day.Sunday),

CurrentDate = Date.From(DateTime.LocalNow()),

AustralianTime = if CurrentDate >= FirstSundayInApril and CurrentDate < FirstSundayInOctober then DateTime.LocalNow() + #duration(0,10,0,0) else DateTime.LocalNow() + #duration(0,11,0,0)
```

You can also use a variant of this using DateTimeZone.FixedUtcNow and DateTime.AddZone, however in most cases it’s probably not necessary.

As well as this, most regions will switch to a different timezone at a specific time of the day, such as 1AM or 2AM. For now, I’ll leave this as an exercise for the reader 😊