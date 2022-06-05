---
layout: post
title: 'Introduction to hierarchies in Analysis Services and Power BI'
date: 2019-01-08 08:16:53.000000000 +01:00
type: post
summary: 'Introduction to hierarchies in Analysis Services and Power BI'
mermaid: true
tag: PBI
---

<p>A <a href="https://en.m.wikipedia.org/wiki/Hierarchy">hierarchy</a> is a concept where related items are represented as being above, below, or at the same level as each other.</p>
<p>Some examples of hierarchies include:</p>
<ul>
<li><strong>Organisation charts</strong> with different levels of managers and employees;</li>
<li><strong>Calendars</strong> with dates organized in years, months, and days;</li>
<li><strong>Products</strong> arranged into different classes, groups, or brands; and</li>
<li><strong>Bills of material</strong> grouped into sub assemblies, kits, and components.</li>
</ul>
<p>Each of these hierarchies include "parent" members and "child" members beneath them, and serve as a way of organizing information into different levels.<!--more--></p>
<h3>How hierarchies are implemented in Analysis Services</h3>
<p>In the context of data modelling, each item in the above examples is an attribute (i.e. column) within a table.</p>
<p>It follows that <a href="https://docs.microsoft.com/en-us/sql/analysis-services/tabular-models/hierarchies-ssas-tabular?view=sql-server-2017">Analysis Services and Power BI use hierarchies</a> as a way to relate different <strong>columns</strong> to each other, with each column on a different level to the others. <i>Note that this is different to relating tables, which is handled separately by the Relationship object.</i></p>
<p>The specific definition used in the Analysis Services Tabular Protocol is:</p>
<blockquote><p><strong>Hierarchy</strong>: A logical tree structure that organizes a record such that each member has one parent member and zero or more child members.</p>
<p><i>— </i><i><a href="https://msdn.microsoft.com/en-us/library/mt719260(v=sql.105).aspx">Analysis Services Tabular Protocol</a></i></p></blockquote>
<p>To get a better understanding of the components that make up a hierarchy, I think it's worth exploring the Hierarchy object's location within the Tabular Object Model of Analysis Services and the properties it has.</p>
<h4>Hierarchy Object Model Location</h4>
<p>Below is the current metadata hierarchy of the Hierarchy object within the Tabular Object Model (TOM).</p>
<p>[caption id="" align="alignnone" width="771"]<img class="wp-image-868 size-full" src="{{ site.baseurl }}/assets/2019/01/ssas-hierarchy-metadata-location.png" width="771" height="304" /> Location of the Hierarchy object within the Tabular Object Model, from the Analysis Services Tabular Protocol[/caption]</p>
<p><i>I'll have to be careful not to write myself into knots when referencing the hierarchy of the Hierarchy object.</i></p>
<p>Note that the Hierarchy object is a child object to the Table object, meaning it will inherit properties such as the table location.</p>
<p>You can also see that the Hierarchy object contains the Level object for each of its hierarchy levels.</p>
<h4>Hierarchy object properties</h4>
<p>Within the Hierarchy object there are the following properties:</p>
<p>[caption id="" align="alignnone" width="848"]<img class="wp-image-869 size-full" src="{{ site.baseurl }}/assets/2019/01/ssas-hierarchy-metadata-properties.png" width="848" height="587" /> Hierarchy object properties[/caption]</p>
<p>I'd like to call out a few of the above properties:</p>
<ol>
<li><strong>TableID</strong> - the hierarchy will belong to a specific table.</li>
<li><strong>Name/Description</strong> - the hierarchy can have its own name and description separate to that of the table.</li>
<li><strong>IsHidden</strong> - it can be hidden if needed.</li>
<li><strong>DisplayFolder</strong> - it can be organized into its own display folder like any other column or measure.</li>
<li><strong>HideMembers</strong> - it can have a property that gets read by Excel (as of January 2019 Power BI does not respect this property) in which blank members are hidden. <i>More on this later.</i></li>
</ol>
<h4>Hierarchy Level Object Properties</h4>
<p>Underneath the Hierarchy object is the Level object. Below are the properties of each of the hierarchy levels:</p>
<p>[caption id="" align="alignnone" width="848"]<img class="wp-image-867 size-full" src="{{ site.baseurl }}/assets/2019/01/ssas-hierarchy-level-metadata-properties.png" width="848" height="317" /> Hierarchy Level object properties[/caption]</p>
<p>To highlight some of the properties:</p>
<ol>
<li><strong>ColumnID</strong> - this is the column that the hierarchy level is based on. It is a pointer to the column and does <strong>not</strong> create a copy of the column's data. <strong>Important</strong>: There is no concept of a ParentColumn-ChildColumn structure. Instead there must be one column per level.</li>
<li><strong>Name</strong> - despite referencing another column, the level can have its own name separate to that of the column it references.</li>
<li><strong>HierarchyID</strong> - this links the level back to the hierarchy it belongs to.</li>
<li><strong>Ordinal</strong> - this is a 0-based index that shows where to place the level relative to other levels within the hierarchy, with 0 being the top level.</li>
</ol>
<h3>The purpose of hierarchies</h3>
<p>Now that we know the basic structure of how hierarchies are implemented, let's explore why you'd want to use them.</p>
<p>The best explanation I've read is from Marco Russo and Alberto Ferrari's book:</p>
<blockquote><p>You can think of them [hierarchies] as predefined pathways through your data that help your users explore down from one level of granularity to another in a meaningful way.</p>
<p><i>— p. 190, </i><i><a href="https://www.microsoftpressstore.com/store/tabular-modeling-in-microsoft-sql-server-analysis-services-9781509302772">Tabular Modelling in Microsoft SQL Server Analysis Services, 2nd Edition</a></i></p></blockquote>
<p>In other words, hierarchies provide intuition on how different columns relate to each other and provide a guide for how you may want to structure your Pivot Tables and other visualizations.</p>
<p>Other advantages to using hierarchies include only needing to add one item (the hierarchy) to your report instead of many, meaning that report building could be quicker for new users. Note that in Excel the entire hierarchy must be used within a field, whereas Power BI allows you to use individual levels from a hierarchy.</p>
<p>Because each level can be renamed, you're also able to have different names for the same columns if they mean different things to different departments. However, I'd caution against this as it can easily cause confusion. A much better solution is to get everyone using the same terms for the same data.</p>
<p>Finally, hierarchies (beginning in SQL Server 2017 / 1400 level compatibility) allow you to implement ragged hierarchies.</p>
<h3>Different kinds of hierarchies</h3>
<p>Now that I've mentioned "ragged hierarchies", it's probably best to start explaining the different kinds of hierarchies that you might encounter.</p>
<p><strong>Ragged hierarchies</strong></p>
<p>A ragged hierarchy is a hierarchy where not all items have children objects down to the lowest level.</p>
<p>Patrick Leblanc has a great video explaining this topic (below), but essentially a ragged hierarchy is one where there are blanks in your hierarchy, and that your blank members only have blank child members.</p>
<p>[youtube https://www.youtube.com/watch?v=4Km6HmydY7A&amp;h=190]</p>
<p>Ragged hierarchies are supported in 1400 compatibility level models, and possible in Power BI Analyze in Excel, but <a href="https://community.powerbi.com/t5/Issues/Hierarchy-blanks-issue/idi-p/221123">not yet officially supported (as of January 2019) by Power BI visuals</a> (they ignore the HideMembers property). See the <a href="https://ideas.powerbi.com/forums/265200-power-bi-ideas/suggestions/14511612-ragged-or-parent-child-hierarchy-visuals">feature request here</a>.</p>
<p>https://twitter.com/jdfancke/status/1081195563247235074</p>
<p>Christian Wade has commented that they are currently working on implementing ragged hierarchies into Power BI.</p>
<p>https://twitter.com/_christianWade/status/1081407315624906752</p>
<p><strong>Natural vs Unnatural hierarchies</strong></p>
<p>Natural hierarchies are where all identical values within a hierarchy column all have the same parent value. An example of this is where each product SKU belongs to a brand. There are no cases where the same product SKU value belongs to different brands.</p>
<p>An unnatural hierarchy is one where the same value in a hierarchy level has different parent levels. An example of this is where the value "January" can belong to multiple years.</p>
<p><strong>Attribute hierarchies vs user defined hierarchies</strong></p>
<p>The Analysis Services Tabular Protocol defines an <strong>attribute hierarchy</strong> as per below:</p>
<blockquote><p><strong>Attribute hierarchy:</strong> An implied single-level hierarchy, based on a single attribute, that consists of all the members of the attribute. An all-level member can optionally be enabled for an attribute hierarchy.</p>
<p><i>— </i><i><a href="https://msdn.microsoft.com/en-us/library/mt719260(v=sql.105).aspx">Analysis Services Tabular Protocol</a></i></p></blockquote>
<p>What this effectively means is that in Analysis Services Tabular, each attribute (i.e. column) has an attribute hierarchy automatically generated for itself. These attribute hierarchies are what you descend when you use MDX formulas (and when you're using Excel cube formulas). <i>More information on attribute hierarchies can be found </i><i><a href="https://docs.microsoft.com/en-us/sql/analysis-services/multidimensional-models-olap-logical-dimension-objects/attributes-and-attribute-hierarchies?view=sql-server-2017">here</a></i><i> and </i><i><a href="https://docs.microsoft.com/en-us/sql/analysis-services/lesson-4-4-hiding-and-disabling-attribute-hierarchies?view=sql-server-2017">here</a></i><i> (applies to Analysis Services Multidimensional). </i></p>
<p>Because there is virtually no control over attribute hierarchies in Analysis Services Tabular, this post is focused solely on user defined hierarchies.</p>
<p>A <strong>user defined hierarchy</strong> is what you, the data modeller, are able to develop in Analysis Services Tabular and Power BI, and is exposed as a hierarchy that can be dropped into Excel PivotTables and Power BI visuals.</p>
<h3>Closing remarks</h3>
<p>The aim of this post was to provide an introduction to the theory of hierarchies.</p>
<p>In my next post I'll be putting this theory to practice with how you'll want to prepare your data for a hierarchy, how you can create hierarchies objects using SSDT and Tabular Editor, and some practical hierarchy use cases.</p>
<p><i>Feel free to leave any comments or corrections below. </i></p>
