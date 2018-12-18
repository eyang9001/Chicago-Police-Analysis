# Group 2 Checkpoint 2 Queries

To answer Question #5, we used the following query to pull in the average allegation count with the data\_officer table.

_select dta.\*, apy.avg\_allegation\_count_

_from data\_officer dta_

_left join (select t.officer\_id, round(avg(allegation\_per\_year), 2) as avg\_allegation\_count from (select officer\_id, EXTRACT(YEAR FROM start\_date) as file\_year, count(distinct allegation\_id) as allegation\_per\_year from data\_officerallegation_

_group by officer\_id, file\_year) as t group by t.officer\_id order by avg\_allegation\_count desc) apy on  dta.id = apy.officer\_id_

We then inputted this table to be cleaned in Trifacta, and joined with the &#39;All Sworn Employees&#39; dataset matching on officer name and DOB year. We grouped all of the officers in the same district and computed the average on the average number of complaints per year.

 ![](/02_Group/Checkpoint/2/images/c2queries1.png)

To integrate with the &#39;Arrest&#39; table, we counted the number of arrests in each district using (output named Arrest\_District\_Count.csv):

_select arr.&quot;ARR\_DISTRICT&quot;, count(arr.&quot;ARR\_DISTRICT&quot;)_

_from arrest\_data arr_

_group by arr.&quot;ARR\_DISTRICT&quot;_

Looking at this summarization of all districts, we noticed that the &#39;Arrest&#39; data did not have any data in districts 13, 21 or 23. So the officers in these districts will not be included in our downstream analyses.

In Trifacta, we joined this District Table on matching the District # for each officer. To do this, we had to extract and reformat to match the column in the &#39;Arrest&#39; table.

To get the District # from the &#39;UNITDESCR&#39; column in &quot;All Sworn Employees&quot;, we had to use a filter to only include rows that start with &#39;DISTRICT 0&#39;, and then using the fuction RIGHT(UNITDESCR, 2) to extract only the last 2 characters, and then re-formatted with NUMFORMAT() to get rid of the pre-fixed &#39;0&#39; in the single-digit numbers (converting &#39;02&#39; to &#39;2&#39;) so it matched the format in the Arrest table.

 ![](/02_Group/Checkpoint/2/images/c2queries2.png)

With this, we were able to then join the &#39;Arrest&#39; table on District Number:

 ![](/02_Group/Checkpoint/2/images/c2queries3.png)

To incorporate Crime data, we did a similar analysis. Counting the number of instances of crime grouped on district with this query (output named Crime\_By\_District.csv):

_select cr.&quot;District&quot;, count(cr.&quot;District&quot;)_

_from crime cr_

_group by cr.&quot;District&quot;_

_order by cr.&quot;District&quot; asc_

 ![](/02_Group/Checkpoint/2/images/c2queries4.png)

And this table was joined onto the overall table by matching District ID:

 ![](/02_Group/Checkpoint/2/images/c2queries5.png)

The data was then exported into Excel to generate the graph visualizations in the formal report
