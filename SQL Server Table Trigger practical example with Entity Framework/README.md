# SQL Server Table Trigger practical example with Entity Framework
## Requires
- Visual Studio 2015
## License
- MIT
## Technologies
- SQL Server
- Entity Framework 6
- SQL Triggers
## Topics
- Entity Framework
- SQL Server Triggers
## Updated
- 01/15/2017
## Description

<h1>Description</h1>
<p><span style="font-size:small">The purpose for this code sample is to show a simple example for using a Trigger to do logic that could also be done in code but in this case we want to make sure that if someone makes changes to the database table outside of
 the application we get the proper action performed.</span></p>
<p><strong><span style="font-size:small">Definition of a Trigger (from Microsoft)</span></strong></p>
<p><span style="font-size:small">A trigger is a special kind of stored procedure that automatically executes when an event occurs in the database server. DML triggers execute when a user tries to modify data through a data manipulation language (DML) event.
 DML events are INSERT, UPDATE, or DELETE statements on a table or view. These triggers fire when any valid event is fired, regardless of whether or not any table rows are affected. For more information</span></p>
<p><span style="font-size:small">Scenario, we have a ordering system that when an order is placed there are order statuses, placed, pending, approved, denied, approved. When the order is placed it's status is set to placed, jumping to approved e.g. the person
 ordering has proper credit, credit card company approved the purchase etc.</span></p>
<p><span style="font-size:small">To have the order status change we use a trigger for after update. Let's look at the table and trigger.</span></p>
<p><img id="168109" src="168109-figure1.jpg" alt="" width="481" height="352"></p>
<p><img id="168112" src="168112-figure2.png" alt="" width="321" height="194"></p>
<p><span style="font-size:small">When a row is updated in TestTable if OrderStatus is set to Approved then OrderApprovalDate is set to the current date/time while any other status OrderApprovedDate is set to null (so this means that we need to have this field
 nullable).&nbsp;</span></p>
<p><span style="font-size:small">Note, my first step to ensure this works is to script out the table, add mocked data, change the mocked data to trigger our trigger to work. With that said, here is the script.</span></p>
<p><span style="font-size:small">&nbsp;</span></p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>SQL</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">mysql</span>

<div class="preview">
<pre class="js"><span class="js__ml_comment">/*&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Script&nbsp;for&nbsp;creating&nbsp;two&nbsp;tables&nbsp;in&nbsp;a&nbsp;database&nbsp;for&nbsp;a&nbsp;code&nbsp;sample&nbsp;which&nbsp;shows&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;how&nbsp;to&nbsp;do&nbsp;conditionals&nbsp;in&nbsp;a&nbsp;trigger.&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;In&nbsp;this&nbsp;case&nbsp;I&nbsp;created&nbsp;TriggerDemoOrderApproval&nbsp;database&nbsp;first&nbsp;then&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;execute&nbsp;this&nbsp;script.&nbsp;You&nbsp;can&nbsp;name&nbsp;the&nbsp;database&nbsp;as&nbsp;you&nbsp;see&nbsp;fit.&nbsp;
*/</span>&nbsp;
&nbsp;
--&nbsp;CHANGE&nbsp;THIS&nbsp;TO&nbsp;YOUR&nbsp;DATABASE&nbsp;
USE&nbsp;TriggerDemoOrderApproval&nbsp;
GO&nbsp;
&nbsp;
/*&nbsp;Drop&nbsp;and&nbsp;Recreate&nbsp;Table&nbsp;*/&nbsp;
IF&nbsp;EXISTS(SELECT&nbsp;*&nbsp;FROM&nbsp;sys.sysobjects&nbsp;WHERE&nbsp;type&nbsp;=&nbsp;<span class="js__string">'U'</span>&nbsp;AND&nbsp;name&nbsp;=&nbsp;<span class="js__string">'TestTable'</span>)&nbsp;
BEGIN&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;DROP&nbsp;TABLE&nbsp;dbo.TestTable;&nbsp;
END&nbsp;
GO&nbsp;
&nbsp;
CREATE&nbsp;TABLE&nbsp;dbo.TestTable(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Id&nbsp;int&nbsp;IDENTITY(<span class="js__num">1</span>,<span class="js__num">1</span>)&nbsp;NOT&nbsp;NULL,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;CompanyName&nbsp;NVARCHAR(MAX),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;FirstName&nbsp;varchar(<span class="js__num">50</span>)&nbsp;NOT&nbsp;NULL,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;LastName&nbsp;varchar(<span class="js__num">50</span>)&nbsp;NOT&nbsp;NULL,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;OrderDate&nbsp;DATETIME&nbsp;NULL,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;OrderApprovalDateTime&nbsp;DATETIME&nbsp;NULL,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;OrderStatus&nbsp;VARCHAR(<span class="js__num">20</span>)&nbsp;NULL,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;PRIMARY&nbsp;KEY&nbsp;CLUSTERED&nbsp;&nbsp;
(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Id&nbsp;ASC&nbsp;
)WITH&nbsp;(PAD_INDEX&nbsp;=&nbsp;OFF,&nbsp;STATISTICS_NORECOMPUTE&nbsp;=&nbsp;OFF,&nbsp;IGNORE_DUP_KEY&nbsp;=&nbsp;OFF,&nbsp;ALLOW_ROW_LOCKS&nbsp;=&nbsp;ON,&nbsp;ALLOW_PAGE_LOCKS&nbsp;=&nbsp;ON)&nbsp;ON&nbsp;[PRIMARY]&nbsp;
)&nbsp;ON&nbsp;[PRIMARY]&nbsp;
GO&nbsp;
&nbsp;
&nbsp;
/*&nbsp;Drop&nbsp;and&nbsp;Recreate&nbsp;Table&nbsp;*/&nbsp;
IF&nbsp;EXISTS(SELECT&nbsp;*&nbsp;FROM&nbsp;sys.sysobjects&nbsp;WHERE&nbsp;type&nbsp;=&nbsp;<span class="js__string">'U'</span>&nbsp;AND&nbsp;name&nbsp;=&nbsp;<span class="js__string">'OrderStatus'</span>)&nbsp;
BEGIN&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;DROP&nbsp;TABLE&nbsp;dbo.OrderStatus;&nbsp;
END&nbsp;
GO&nbsp;
&nbsp;
CREATE&nbsp;TABLE&nbsp;dbo.OrderStatus(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;id&nbsp;INT&nbsp;IDENTITY(<span class="js__num">1</span>,<span class="js__num">1</span>)&nbsp;NOT&nbsp;NULL,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Status&nbsp;NVARCHAR(MAX)&nbsp;NULL,&nbsp;
&nbsp;CONSTRAINT&nbsp;PK_OrderStatus&nbsp;PRIMARY&nbsp;KEY&nbsp;CLUSTERED&nbsp;&nbsp;
(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;id&nbsp;ASC&nbsp;
)WITH&nbsp;(PAD_INDEX&nbsp;=&nbsp;OFF,&nbsp;STATISTICS_NORECOMPUTE&nbsp;=&nbsp;OFF,&nbsp;IGNORE_DUP_KEY&nbsp;=&nbsp;OFF,&nbsp;ALLOW_ROW_LOCKS&nbsp;=&nbsp;ON,&nbsp;ALLOW_PAGE_LOCKS&nbsp;=&nbsp;ON)&nbsp;ON&nbsp;[PRIMARY]&nbsp;
)&nbsp;ON&nbsp;[PRIMARY]&nbsp;TEXTIMAGE_ON&nbsp;[PRIMARY]&nbsp;
GO&nbsp;
&nbsp;
&nbsp;
/*&nbsp;Add&nbsp;rows&nbsp;*/&nbsp;
INSERT&nbsp;INTO&nbsp;dbo.TestTable&nbsp;&nbsp;(CompanyName,FirstName&nbsp;,&nbsp;LastName,&nbsp;OrderDate,&nbsp;OrderStatus)&nbsp;
VALUES&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;(<span class="js__string">'Fast&nbsp;Miata'</span>,&nbsp;<span class="js__string">'Karen'</span>&nbsp;,&nbsp;<span class="js__string">'Payne'</span>&nbsp;,&nbsp;<span class="js__string">'20160101&nbsp;08:34:09&nbsp;AM'</span>&nbsp;,&nbsp;<span class="js__string">'Placed'</span>&nbsp;),&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;(<span class="js__string">'Best&nbsp;Little&nbsp;Coffee&nbsp;shop'</span>,&nbsp;<span class="js__string">'Bill'</span>&nbsp;,&nbsp;<span class="js__string">'Gallagher'</span>&nbsp;,&nbsp;<span class="js__string">'20160101&nbsp;11:12:00&nbsp;AM'</span>&nbsp;,&nbsp;<span class="js__string">'Placed'</span>&nbsp;),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;(<span class="js__string">'Guitar'</span><span class="js__string">'s&nbsp;are&nbsp;us'</span>,&nbsp;<span class="js__string">'Sean'</span>&nbsp;,&nbsp;<span class="js__string">'Hills'</span>&nbsp;,&nbsp;<span class="js__string">'20160101&nbsp;09:11:06&nbsp;AM'</span>&nbsp;,&nbsp;<span class="js__string">'Placed'</span>&nbsp;),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;(<span class="js__string">'Mary'</span><span class="js__string">'s&nbsp;cartering&nbsp;service'</span>,&nbsp;<span class="js__string">'Mary'</span>&nbsp;,&nbsp;<span class="js__string">'Adams'</span>&nbsp;,&nbsp;<span class="js__string">'20160102&nbsp;11:24:34&nbsp;AM'</span>&nbsp;,&nbsp;<span class="js__string">'Placed'</span>&nbsp;)&nbsp;,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;(<span class="js__string">'ZZZ&nbsp;Inc'</span>,&nbsp;<span class="js__string">'Frank'</span>&nbsp;,&nbsp;<span class="js__string">'Jenkins'</span>&nbsp;,&nbsp;<span class="js__string">'20160102&nbsp;13:10:00&nbsp;PM'</span>&nbsp;,&nbsp;<span class="js__string">'Placed'</span>&nbsp;)&nbsp;&nbsp;
&nbsp;
&nbsp;
INSERT&nbsp;INTO&nbsp;dbo.OrderStatus&nbsp;&nbsp;(&nbsp;Status&nbsp;)&nbsp;
VALUES&nbsp;(&nbsp;N<span class="js__string">'Approved'</span>&nbsp;),&nbsp;&nbsp;&nbsp;&nbsp;(&nbsp;N<span class="js__string">'Pending'</span>&nbsp;),&nbsp;&nbsp;&nbsp;&nbsp;(&nbsp;N<span class="js__string">'Hold'</span>&nbsp;),(&nbsp;N<span class="js__string">'Denied'</span>),(&nbsp;N<span class="js__string">'Placed'</span>),(&nbsp;N<span class="js__string">'Unknown'</span>)&nbsp;
&nbsp;
<span class="js__ml_comment">/*&nbsp;Drop&nbsp;and&nbsp;Recreate&nbsp;Trigger&nbsp;*/</span>&nbsp;
IF&nbsp;EXISTS(SELECT&nbsp;*&nbsp;FROM&nbsp;sys.sysobjects&nbsp;WHERE&nbsp;type&nbsp;=&nbsp;<span class="js__string">'TR'</span>&nbsp;AND&nbsp;name&nbsp;=&nbsp;<span class="js__string">'trTriggerApprovalDate'</span>)&nbsp;
BEGIN&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;DROP&nbsp;TRIGGER&nbsp;dbo.trTriggerApprovalDate;&nbsp;
END&nbsp;
GO&nbsp;
&nbsp;
SET&nbsp;ANSI_NULLS&nbsp;ON&nbsp;
GO&nbsp;
SET&nbsp;QUOTED_IDENTIFIER&nbsp;ON&nbsp;
GO&nbsp;
&nbsp;
--&nbsp;=============================================&nbsp;
--&nbsp;Description:&nbsp;&nbsp;&nbsp;&nbsp;Sets&nbsp;OrderApprovalDateTime&nbsp;to&nbsp;the&nbsp;current&nbsp;date-time&nbsp;when&nbsp;
--&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OrderStatus&nbsp;is&nbsp;et&nbsp;to&nbsp;Approved&nbsp;
--&nbsp;=============================================&nbsp;
CREATE&nbsp;TRIGGER&nbsp;trTriggerApprovalDate&nbsp;ON&nbsp;dbo.TestTable&nbsp;
AFTER&nbsp;UPDATE&nbsp;
AS&nbsp;
BEGIN&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;SET&nbsp;NOCOUNT&nbsp;ON;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;UPDATE&nbsp;dbo.TestTable&nbsp;SET&nbsp;OrderApprovalDateTime&nbsp;=&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CASE&nbsp;WHEN&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;t.OrderStatus&nbsp;=&nbsp;<span class="js__string">'Approved'</span>&nbsp;THEN&nbsp;GETDATE()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ELSE&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NULL&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;END)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;FROM&nbsp;&nbsp;dbo.TestTable&nbsp;AS&nbsp;t&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;INNER&nbsp;JOIN&nbsp;inserted&nbsp;i&nbsp;ON&nbsp;t.ID=i.ID&nbsp;AND&nbsp;i.OrderStatus=<span class="js__string">'Approved'</span>&nbsp;
END&nbsp;
&nbsp;
GO&nbsp;
&nbsp;
<span class="js__ml_comment">/*&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Do&nbsp;updates,&nbsp;invoke&nbsp;our&nbsp;trigger&nbsp;if&nbsp;you&nbsp;want&nbsp;to&nbsp;start&nbsp;with&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;various&nbsp;statuses,&nbsp;otherwise&nbsp;in&nbsp;the&nbsp;demo&nbsp;app&nbsp;set&nbsp;them.&nbsp;
*/</span>&nbsp;
--UPDATE&nbsp;dbo.TestTable&nbsp;SET&nbsp;OrderStatus&nbsp;=&nbsp;<span class="js__string">'Approved'</span>&nbsp;WHERE&nbsp;Id&nbsp;=&nbsp;<span class="js__num">1</span>&nbsp;
--UPDATE&nbsp;dbo.TestTable&nbsp;SET&nbsp;OrderStatus&nbsp;=&nbsp;<span class="js__string">'Pending'</span>&nbsp;WHERE&nbsp;Id&nbsp;=&nbsp;<span class="js__num">2</span>&nbsp;
--UPDATE&nbsp;dbo.TestTable&nbsp;SET&nbsp;OrderStatus&nbsp;=&nbsp;<span class="js__string">'Hold'</span>&nbsp;WHERE&nbsp;Id&nbsp;=&nbsp;<span class="js__num">3</span>&nbsp;
--UPDATE&nbsp;dbo.TestTable&nbsp;SET&nbsp;OrderStatus&nbsp;=&nbsp;<span class="js__string">'Approved'</span>&nbsp;WHERE&nbsp;Id&nbsp;=&nbsp;<span class="js__num">4</span>&nbsp;
--UPDATE&nbsp;dbo.TestTable&nbsp;SET&nbsp;OrderStatus&nbsp;=&nbsp;<span class="js__string">'Denied'</span>&nbsp;WHERE&nbsp;Id&nbsp;=&nbsp;<span class="js__num">5</span>&nbsp;
&nbsp;
<span class="js__ml_comment">/*&nbsp;Show&nbsp;our&nbsp;data&nbsp;*/</span>&nbsp;
SELECT&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Id,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;CompanyName,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;CONCAT(FirstName,<span class="js__string">'&nbsp;'</span>,LastName)&nbsp;AS&nbsp;Contact,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;FORMAT(CAST(OrderDate&nbsp;AS&nbsp;DATETIME),<span class="js__string">'MM/dd/yyyy&nbsp;hh:mm&nbsp;tt'</span>)&nbsp;AS&nbsp;Ordered,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;FORMAT(CAST(OrderApprovalDateTime&nbsp;AS&nbsp;DATETIME),<span class="js__string">'MM/dd/yyyy&nbsp;hh:mm&nbsp;tt'</span>)&nbsp;AS&nbsp;OrderApprovalDateTime,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;OrderStatus&nbsp;&nbsp;&nbsp;
FROM&nbsp;TestTable&nbsp;
&nbsp;
SELECT&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;id&nbsp;,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;[Status]&nbsp;
FROM&nbsp;dbo.OrderStatus&nbsp;
</pre>
</div>
</div>
</div>
<div class="endscriptcode">&nbsp;Results</div>
<div class="endscriptcode"><img id="168113" src="168113-figure3.jpg" alt="" width="608" height="280"></div>
<p>&nbsp;</p>
<p><span style="font-size:small">Note the commented out UPDATE statements, uncomment them, execute again and we get</span></p>
<p><img id="168114" src="168114-figure4.jpg" alt="" width="603" height="137"></p>
<p><span style="font-size:small">So now we know it works, the next step is to create a class to read back data and update a row.</span></p>
<p><span style="font-size:small">Screenshot of final product</span></p>
<p><img id="168115" src="168115-figure5.jpg" alt="" width="561" height="259"></p>
<p><span style="font-size:small">When traversing the DataGridView the ComboBox bottom right shows the current row order status, change it, press Set, press Edit Current (or should I say Save Current).</span></p>
<p><span style="font-size:small">The method (taken from our operation class) to save changes</span></p>
<p>&nbsp;</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js"><span class="js__sl_comment">///&nbsp;&lt;summary&gt;</span>&nbsp;
<span class="js__sl_comment">///&nbsp;Update&nbsp;a&nbsp;row&nbsp;in&nbsp;the&nbsp;table</span>&nbsp;
<span class="js__sl_comment">///&nbsp;&lt;/summary&gt;</span>&nbsp;
<span class="js__sl_comment">///&nbsp;&lt;param&nbsp;name=&quot;sender&quot;&gt;&lt;/param&gt;</span>&nbsp;
<span class="js__sl_comment">///&nbsp;&lt;returns&gt;&lt;/returns&gt;</span>&nbsp;
<span class="js__sl_comment">///&nbsp;&lt;remarks&gt;</span>&nbsp;
<span class="js__sl_comment">///&nbsp;We&nbsp;don't&nbsp;need&nbsp;a&nbsp;task&nbsp;based&nbsp;method&nbsp;but&nbsp;wanted&nbsp;to&nbsp;show</span>&nbsp;
<span class="js__sl_comment">///&nbsp;how&nbsp;since&nbsp;many&nbsp;desktop&nbsp;developers&nbsp;are&nbsp;not&nbsp;use&nbsp;to&nbsp;this</span>&nbsp;
<span class="js__sl_comment">///&nbsp;who&nbsp;are&nbsp;just&nbsp;learning.</span>&nbsp;
<span class="js__sl_comment">///&nbsp;&lt;/remarks&gt;</span>&nbsp;
public&nbsp;async&nbsp;Task&lt;TestTable&gt;&nbsp;UpdateItem(TestTable&nbsp;sender)&nbsp;
<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;using&nbsp;(DemoEntities&nbsp;entity&nbsp;=&nbsp;<span class="js__operator">new</span>&nbsp;DemoEntities())&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;entity.Entry(sender).State&nbsp;=&nbsp;EntityState.Modified;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;await&nbsp;entity.SaveChangesAsync();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">var</span>&nbsp;testTableItem&nbsp;=&nbsp;await(entity.TestTables.Where(item&nbsp;=&gt;&nbsp;item.Id&nbsp;==&nbsp;sender.Id).FirstOrDefaultAsync&lt;TestTable&gt;());&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">return</span>&nbsp;testTableItem;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;
<span class="js__brace">}</span></pre>
</div>
</div>
</div>
<div class="endscriptcode">&nbsp;</div>
<p><span style="font-size:small">Get the approval date&nbsp;</span></p>
<p></p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js"><span class="js__sl_comment">///&nbsp;&lt;summary&gt;</span>&nbsp;
<span class="js__sl_comment">///&nbsp;Get&nbsp;new&nbsp;approval&nbsp;date</span>&nbsp;
<span class="js__sl_comment">///&nbsp;&lt;/summary&gt;</span>&nbsp;
<span class="js__sl_comment">///&nbsp;&lt;param&nbsp;name=&quot;sender&quot;&gt;&lt;/param&gt;</span>&nbsp;
<span class="js__sl_comment">///&nbsp;&lt;returns&gt;&lt;/returns&gt;</span>&nbsp;
public&nbsp;DateTime?&nbsp;GetApprovalDate(TestTable&nbsp;sender)&nbsp;
<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;using&nbsp;(DemoEntities&nbsp;entity&nbsp;=&nbsp;<span class="js__operator">new</span>&nbsp;DemoEntities())&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">return</span>&nbsp;entity.TestTables.Where(item&nbsp;=&gt;&nbsp;item.Id&nbsp;==&nbsp;sender.Id).FirstOrDefault().OrderApprovalDateTime;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
<span class="js__brace">}</span></pre>
</div>
</div>
</div>
<div class="endscriptcode"><span style="font-size:small">&nbsp;Note that the Entity Framework model/classes are within a class project called by a windows form project, see code to see how it's implementing (fairly simple).&nbsp;</span></div>
<div class="endscriptcode"></div>
<div class="endscriptcode"><span style="font-size:small">As mentioned before, write SQL then code so for a test I wanted feedback to the changes e.g. which shows a grouped count of order statues</span></div>
<div class="endscriptcode">
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>SQL</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">mysql</span>

<div class="preview">
<pre class="js">SELECT&nbsp;&nbsp;COUNT(Id),&nbsp;OrderStatus&nbsp;
FROM&nbsp;[TriggerDemoOrderApproval].[dbo].[TestTable]&nbsp;
GROUP&nbsp;BY&nbsp;OrderStatus</pre>
</div>
</div>
</div>
<div class="endscriptcode"><span style="font-size:small">In code via Entity Framework</span>&nbsp;</div>
<div class="endscriptcode">
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js">public&nbsp;string&nbsp;GroupedStatus()&nbsp;
<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">var</span>&nbsp;sbGrouped&nbsp;=&nbsp;<span class="js__operator">new</span>&nbsp;<a class="libraryLink" href="https://msdn.microsoft.com/en-US/library/System.Text.StringBuilder.aspx" target="_blank" title="Auto generated link to System.Text.StringBuilder">System.Text.StringBuilder</a>();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;using&nbsp;(DemoEntities&nbsp;entity&nbsp;=&nbsp;<span class="js__operator">new</span>&nbsp;DemoEntities())&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IQueryable&lt;GroupCount&gt;&nbsp;results&nbsp;=&nbsp;entity.TestTables&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.GroupBy(item&nbsp;=&gt;&nbsp;item.OrderStatus)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Select(item&nbsp;=&gt;&nbsp;<span class="js__operator">new</span>&nbsp;GroupCount&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Key&nbsp;=&nbsp;item.Key,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Count&nbsp;=&nbsp;item.Count()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;foreach&nbsp;(<span class="js__statement">var</span>&nbsp;item&nbsp;<span class="js__operator">in</span>&nbsp;results)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sbGrouped.AppendLine($<span class="js__string">&quot;{item.Key}&nbsp;-&nbsp;{item.Count}&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">return</span>&nbsp;sbGrouped.ToString();&nbsp;
&nbsp;
<span class="js__brace">}</span></pre>
</div>
</div>
</div>
<div class="endscriptcode">&nbsp;</div>
</div>
<div class="endscriptcode"><strong><span style="font-size:small">Notes</span></strong></div>
<div class="endscriptcode"><span style="font-size:small">Only use triggers when they make sense.</span></div>
<div class="endscriptcode"><span style="font-size:small"><br>
</span></div>
<div class="endscriptcode"><span style="font-size:small">I did not use a DataGridViewComboBox because it takes away from the current topic. For seeing how to use a DataGridViewComboBox see my
<a href="https://code.msdn.microsoft.com/DataGridView-ComboBox-with-b62fe359" target="_blank">
MSDN code sample</a>.</span></div>
<div class="endscriptcode"></div>
<div class="endscriptcode"><span style="font-size:small">We can do the above with SqlClient data provider too if not into using Entity Framework.</span></div>
<div class="endscriptcode"></div>
<div class="endscriptcode"><strong><span style="font-size:small">IMPORTANT</span></strong></div>
<div class="endscriptcode"><span style="font-size:small">The Entity Framework model in app.config points to data source KARNS-PC, my SQL-Server and the initial catalog points to the table in Script.sql (in the solution). You need to make sure to change these
 to match your environment e.g. if your Server name is SQLExpress then replace KARENS-PC with that.</span></div>
</div>
<div class="endscriptcode"><span style="font-size:small"><br>
</span></div>
<p></p>
<p>&nbsp;</p>
<p><span style="font-size:small"><br>
</span></p>
<p>&nbsp;</p>
