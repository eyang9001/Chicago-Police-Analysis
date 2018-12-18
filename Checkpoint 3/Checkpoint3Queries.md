# Group 2 - Checkpoint 3 Queries

## Question #8:  Do complaints tend to be filed against a cluster of police officers? How do these potential clusters change over time?

We want to cluster the officers that are involved in the same complaints. To do this, we first ran a query to list out pairs of officers that were involved in the same complaint. To test this, we limited our analysis to allegations that happened in 2015, to keep the amount of data manageable. We used this query (table named officer\_allegation\_network.csv):

_select n1.incident\_date, n1.allegation\_id, n1.disciplined, n1.officer\_id as &quot;officer1&quot;, n2.officer\_id as &quot;officer2&quot;_

_from (select da.incident\_date, doa.id, doa.allegation\_id, doa.officer\_id, doa.disciplined_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n1_

_cross join (select da.incident\_date, doa.allegation\_id, doa.officer\_id, doa.disciplined_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n2_

_where n1.allegation\_id=n2.allegation\_id and n1.officer\_id != n2.officer\_id and n1.incident\_date \&gt; &#39;2015-01-01&#39; and n1.incident\_date \&lt; &#39;2016-01-01&#39;;_

To use the connected components analysis, it needs a list of all of the officers in the analysis. This was acquired with the query (tabled named 2015\_allegation\_network\_officers.csv):

_select distinct n1.officer\_id_

_from (select da.incident\_date, doa.id, doa.allegation\_id, doa.officer\_id, doa.disciplined_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n1_

_cross join (select da.incident\_date, doa.allegation\_id, doa.officer\_id, doa.disciplined_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n2_

_where n1.allegation\_id=n2.allegation\_id and n1.officer\_id != n2.officer\_id and n1.incident\_date \&gt; &#39;2015-01-01&#39; and n1.incident\_date \&lt; &#39;2016-01-01&#39;;_

Then, we import each of these datasets into spark (databricks platform). Using the queries above, we can create a vertices and edges representation. The first query represents edges, and the second query represents vertices.

In order to use GraphX in Python, we utilized the Graphframe Package via the Library import functionality in Databricks. Using the &quot;Maven Coordinate&quot; option, Graphframe can be installed / attached directly to the cluster that we are currently working on, eliminating the need for additional configuration. Then the following code was used to create the graph, and run both connected components and a PageRank.





from functools import reduce

from pyspark.sql.functions import col, lit, when

from graphframes import \*

vertices = spark.table(&#39;allegation\_network\_officers\_csv&#39;)

vertices = vertices.withColumnRenamed(&quot;officer\_id&quot;, &quot;id&quot;)

edges = spark.table(&#39;officer\_allegation\_network\_csv&#39;)

edges = edges.drop(&quot;incident\_date&quot;,&quot;allegation\_id&quot;,&quot;disciplined&quot;)

edges = edges.withColumnRenamed(&quot;officer1&quot;, &quot;src&quot;)

edges = edges.withColumnRenamed(&quot;officer2&quot;, &quot;dst&quot;)

g = GraphFrame(vertices, edges)

sc.setCheckpointDir(&quot;/tmp/&quot;)

result = g.connectedComponents()

result.show()



results = g.pageRank(resetProbability=0.15, tol=0.01)

display(results.vertices)

## Question #9:  Are we able to see if once police officers potentially &quot;get away&quot; with certain behaviors, they are much more likely to keep committing the same acts repeatedly?

We needed a group of officers that we consider as &#39;bad offenders&#39; and compare to their peers. To get the list of &#39;bad offenders&#39;, we used the top 7% of officers with the highest number of complaints per year normalized by their district average. This was done by taking officers from the top three buckets from figure 1 of our checkpoint #2 analysis (table named &#39;Checkpoint\_3\_bad\_officers.csv). We uploaded this file into postgres as a new table called &#39;bad\_officers&#39;.

With this, we then wrote a query to calculate the time between each complaint for all of the officers. We used this query (output named officer\_allegation\_intervals.csv):

_select extract(epoch from n2.incident\_date - n1.incident\_date)/86400 as &quot;allegation\_interval\_days&quot;, case when n1.officer\_id in (select bo.id from bad\_officers bo) then &#39;True&#39; else &#39;False&#39; end as bad\_officer_

_from (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n1, (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n2_

_where n1.row\_number=n2.row\_number-1 and n1.officer\_id=n2.officer\_id;_

 ![](/02_Group/Checkpoint/3/images/ch3q1.png)

When creating a histogram of this, we found some outliers with the interval being around 16000 causing the graph to be very wide. So we added in a threshold to only look at intervals less than 1000 days (output named Filtered\_Intervals.csv):

_select extract(epoch from n2.incident\_date - n1.incident\_date)/86400 as &quot;allegation\_interval\_days&quot;, case when n1.officer\_id in (select bo.id from bad\_officers bo) then &#39;True&#39; else &#39;False&#39; end as bad\_officer_

_from (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n1, (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n2_

_where n1.row\_number=n2.row\_number-1 and n1.officer\_id=n2.officer\_id and extract(epoch from n2.incident\_date - n1.incident\_date)/86400 \&lt; 1000;_

In Spark, we displayed this as a histogram with:

 df2 = spark.table(&quot;Filtered\_Intervals\_csv&quot;)

 display(df2)

 ![](/02_Group/Checkpoint/3/images/Overall_Interval_Histograms.png)

To dig a bit deeper, we wanted to see if there was going to be a shift in the distribution when looking at the intervals between complaints when the officer got disciplined for the first allegation. To get these intervals, we used this query (output named Disciplined\_Intervals.csv):

_select extract(epoch from n2.incident\_date - n1.incident\_date)/86400 as &quot;allegation\_interval\_days&quot;, case when n1.officer\_id in (select bo.id from bad\_officers bo) then &#39;True&#39; else &#39;False&#39; end as bad\_officer_

_from (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n1, (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n2_

_where n1.row\_number=n2.row\_number-1 and n1.officer\_id=n2.officer\_id_ _and n1.disciplined = False_ _and extract(epoch from n2.incident\_date - n1.incident\_date)/86400 \&lt; 1000;_

 ![](/02_Group/Checkpoint/3/images/Disciplined_Interval_Histograms.png)

And to get the counter-set of intervals between complaints where the officer **did not** get disciplined for the first allegation, we used the query:

_select extract(epoch from n2.incident\_date - n1.incident\_date)/86400 as &quot;allegation\_interval\_days&quot;, case when n1.officer\_id in (select bo.id from bad\_officers bo) then &#39;True&#39; else &#39;False&#39; end as bad\_officer_

_from (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n1, (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n2_

_where n1.row\_number=n2.row\_number-1 and n1.officer\_id=n2.officer\_id_ _and n1.disciplined = True_ _and extract(epoch from n2.incident\_date - n1.incident\_date)/86400 \&lt; 1000;_

 ![](/02_Group/Checkpoint/3/images/Undisciplined_Interval_histograms.png)

To get the overall averages, we used the query:

_select_ _avg(extract(epoch from n2.incident\_date - n1.incident\_date)/86400)_ _as &quot;allegation\_interval\_days&quot;_

_from (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n1, (select \*, row\_number() over (order by doa.&quot;officer\_id&quot;, da.incident\_date asc)_

_from data\_allegation da_

_inner join data\_officerallegation doa on da.id=doa.allegation\_id) n2_

_where n1.row\_number=n2.row\_number-1 and n1.officer\_id=n2.officer\_id_ _and n1.disciplined = True_ _and n1.officer\_id_ _not in_ _(select bo.id from bad\_officers bo);_

Where we changed the &#39;not in&#39; to &#39;in&#39; to switch between sets of officers, and changed the &#39;n1.disciplined&#39; requirement to True and False.

|   | **Overall Average Interval (Days)** | **Average Interval After Disciplined Complaint (Days)** | **Average Interval After Non-Disciplined Complaint (Days)** |
| --- | --- | --- | --- |
| **Repeat Offenders** | 145 | 177 | 142 |
| **Non-Repeat Offenders** | 398 | 481 | 393 |
