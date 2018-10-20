# Chicago Police Department Project Proposal - Checkpoint 1

For checkpoint 1, we have explored the questions proposed in the description and summarization sections of our latest proposal. Our queries and high level results are shown below in a question by question format. Parts of these questions use results from our data integration, which we will build on further in the next checkpoint. Note, that on our proposal we quantified complaints and allegations in the following way.

- Complaint: A report indicating an incident, but not an explicit legal action
- Allegation: A legal filing derived from a complaint (or possibly not) accusing a
party of wrongdoing.

Given that our idea of complaints are defined as allegations in the CPDB, we can change the definitions moving forward. However, in answering the following questions, the above definitions are still used.

## Question #1
**Question:** First, what are the average number of complaints and allegations filed against each officer over a specific period of time, and what does the holistic distribution most resemble?_

The following query was used to get the average number of complaints per year from all officers in the cpdb. We have not yet looked at allegations, given that this will come from the data integration step.

SQL Query:

_select t.officer\_id, round(avg(allegation\_per\_year), 2) as avg\_allegation\_count from_

_(select officer\_id, EXTRACT(YEAR FROM start\_date) as file\_year, count(distinct allegation\_id) as allegation\_per\_year from data\_officerallegation_

_group by officer\_id, file\_year) as t group by t.officer\_id order by avg\_allegation\_count desc;_

**Table 1.1** : Top 10 Officer&#39;s by Average Complaints per Year

| **officer\_id** | **Avg\_allegation\_count (per year)** |
| --- | --- |
| 13788 | 11.56 |
| 12074 | 10 |
| 8658 | 8.64 |
| 7015 | 8.29 |
| 28805 | 8.2 |
| 9758 | 8.2 |
| 8562 | 7.95 |
| 13095 | 7.54 |
| 32265 | 7.21 |
| 5108 | 7 |

The histogram below shows a high level view of the distribution. With this simple analysis we can see a few things. First off, the vast majority of officers have very few complaints against them on an annualized basis. However, these is a fairly long tail, where about 2% of officers have a very high amount of complaints against them on average. We could certainly see how this analysis looks in more fine detail, but where we would like to focus our attention is how our second cohort of between 2 and 4 complaints per year breaks out once we normalize the data. It is clear that the vast majority of the 2% of trailing officers are likely to be repeat offenders, but this may change once normalized.

 ![](/Checkpoint1Images/q1.png)

## Question #2
**Question:** What is the range of complaints to allegations as a ratio for all officers? Are officers more likely to hold a specific position when they have allegations filed against them - i.e. what are the average number of complaints and acquittals filed for officers at different stages of tenure?_

We were able to get the total number of complaints with discipline (True Complaints) and without discipline (False Complaints) for officers at different positions. Once we incorporate the settlement data, we will be able to determine the number of complaints that converted to legal action.

SQL Query:

_select t2.officer\_id, t2.rank, t2.disciplined, count(distinct allegation\_id) as allegation\_count from_

_(select doa.\*, o.rank from data\_officerallegation doa, data\_officer o where doa.officer\_id=o.id) as t2 group by t2.officer\_id, t2.rank, t2.disciplined;_

Additionally we computed the % of complaints that were disciplined at each level. Interestingly, we see that as officers move up in position from a corporate structure standpoint, the percent of complaints converted to discipline goes down. This may be something to dig into as we move further into the project. Is there a trend in the actual type of complaint?

| **Officer Position** | **False Complaints** | **True Complaints** | **% True of Total Complaints** |
| --- | --- | --- | --- |
| Police Officer | 145695 | 13806 | 9% |
| Sergeant | 32877 | 2247 | 6% |
| Detective | 23323 | 1336 | 5% |
| Lieutenant | 7756 | 595 | 7% |
| Field Training Officer | 3511 | 226 | 6% |
| Captain | 1527 | 125 | 8% |
| Commander | 1330 | 62 | 4% |
| Deputy Chief | 506 | 20 | 4% |
| Chief | 269 | 8 | 3% |
| Director Of Caps | 90 | 2 | 2% |
| Deputy Superintendent | 59 | 1 | 2% |
| First Deputy Superintendent | 58 | 7 | 11% |
| Assistant Dep. Superintendent | 34 | 4 | 11% |
| Other | 25 | 1 | 4% |
| Assistant Superintendent | 20 | 2 | 9% |
| Superintendent Of Police | 19 | 1 | 5% |
| Superintendent&#39;S Chief Of Staff | 1 | 0 | 0% |

## Question #3
**Question:** How many police officers have a significant number of complaints against them that_** don&#39;t** _have corresponding TTR police reports?_

In order to match the complaints to corresponding TRR police reports, we used the criteria of them being for the same officer, and the reported incident dates are similar. In order to find a reasonable date range threshold to accommodate the fact that complaints and TRR don&#39;t get reported in the same timeline, while also avoiding potential false positives, we analyzed the number of matches that occur as the date threshold changes.

The threshold test used the SQL Query (substituting the &#39;30&#39; with other values):

_select a.id_

_from data\_allegation a_

_inner join data\_officerallegation oa on a.id = oa.allegation\_id_

_inner join trr\_trr trr on (oa.officer\_id = trr.officer\_id and (CAST(a.incident\_date as DATE) \&gt; CAST(trr.trr\_datetime as DATE) - 30) and (CAST(a.incident\_date as DATE) \&lt; CAST(trr.trr\_datetime as DATE) + 30))_

_group by a.id_

**Graph 3.1** : Threshold test of date range to find matches between Complaints and TRRs

![](/Checkpoint1Images/graph31.png)
 
As a result, the number of matches started to plateau after 30 days. We also figured that 30 days is enough of a grace period for civilians to file their allegations after incidents occur.

Using the 30 day threshold, we then found the number of allegations that **did not** have corresponding TRRs for each officer using this SQL Query:

_select awt.officer\_id, awt.&quot;#\_Complaints\_Without\_TRR&quot;, atrr.&quot;#\_Total\_Complaints&quot;_

_From (select oa.officer\_id, count(oa.id) as &quot;#\_Complaints\_Without\_TRR&quot;_

_from data\_allegation a_

_INNER JOIN data\_officerallegation oa on a.id = oa.allegation\_id_

_LEFT JOIN trr\_trr trr on (oa.officer\_id = trr.officer\_id and (CAST(a.incident\_date as DATE) \&gt; CAST(trr.trr\_datetime as DATE) - 30) and (CAST(a.incident\_date as DATE) \&lt; CAST(trr.trr\_datetime as DATE) + 30))_

_where trr.trr\_datetime is NULL_

_group by oa.officer\_id_

_order by count(oa.id) DESC) awt_

_inner join (select oo.officer\_id, count(oo.id) as &quot;#\_Total\_Complaints&quot; from data\_officerallegation oo group by oo.officer\_id order by count(oo.id) desc) atrr on awt.officer\_id = atrr.officer\_id;_

**Table 3.1** : Top 10 officers by Number of Complaints Without Corresponding TRR

| **officer\_id** | **#\_Complaints\_Without\_TRR** | **#\_Total\_Complaints** |
| --- | --- | --- |
| 8562 | 173 | 175 |
| 21837 | 134 | 137 |
| 17816 | 124 | 136 |
| 28805 | 123 | 123 |
| 8138 | 114 | 132 |
| 29033 | 110 | 114 |
| 21468 | 108 | 125 |
| 4807 | 108 | 109 |
| 32166 | 106 | 109 |
| 13788 | 104 | 104 |

With these results, it looks like the large majority of the complaints don&#39;t correspond to TRRs in the database. We will need to do some more investigation to determine the reason for this, whether this is because the data sets for the complaints and TRRs are over different date ranges, or if this is accurate and we need to find a statistical model to enumerate the outliers.

## Question #4
**Question:** We would also like to see the average number of complaints that are filed for off-duty officers, and how this relates to the other metrics._

The following query allows us to compute the total number of complaints that each officer received from both a on-duty perspective and an off-duty perspective.

SQL Query:

_select officer\_id,_

_(select count(\*) from trr\_trr tbl2 where tbl1.officer\_id = tbl2.officer\_id and tbl2.officer\_in\_uniform is true) as trr\_count\_in\_uniform,_

_(select count(\*) from trr\_trr tbl3 where tbl1.officer\_id = tbl3.officer\_id and tbl3.officer\_in\_uniform is false) as trr\_count\_not\_in\_uniform_

_from trr\_trr tbl1 group by officer\_id, trr\_count\_in\_uniform, trr\_count\_not\_in\_uniform_

_order by trr\_count\_not\_in\_uniform desc;_

_       _

We can certainly use this information to readily compute the average. However, we first wanted to look at how the on-duty vs. off-duty distribution for top offenders. Below are two tables with top 10 officers by on-duty complaints and the top 10 officers by off-duty complaints as well.

**Table 4.1:** Top 10 Officers by Number of On-Duty Complaints

| **Officer Number** | **Complaints On-Duty** | **Complaints Off-Duty** | **% Complains in Uniform** |
| --- | --- | --- | --- |
| 14400 | 61 | 1 | 98% |
| 32105 | 56 | 0 | 100% |
| 31030 | 49 | 4 | 92% |
| 28968 | 48 | 3 | 94% |
| 31795 | 47 | 1 | 98% |
| 13873 | 46 | 10 | 82% |
| 13605 | 45 | 1 | 98% |
| 30091 | 43 | 3 | 93% |
| 13778 | 43 | 2 | 96% |
| 23132 | 42 | 5 | 89% |

**Table 4.2:** Top 10 Officers by Number of Off-Duty Complaints

| **Officer Number** | **Complaints On-Duty** | **Complaints Off-Duty** | **% Complains in Uniform** |
| --- | --- | --- | --- |
| 2470 | 11 | 45 | 20% |
| 15631 | 19 | 37 | 34% |
| 32118 | 30 | 35 | 46% |
| 10152 | 24 | 35 | 41% |
| 25983 | 10 | 35 | 22% |
| 31898 | 2 | 34 | 6% |
| 10583 | 39 | 33 | 54% |
| 28329 | 15 | 32 | 32% |
| 32402 | 17 | 31 | 35% |
| 25717 | 9 | 31 | 23% |

Looking at the above tables closely, we can see that the officers who have the most complaints overall, should really be examined further, as they may just be operating in an extremely high crime environment. This will be very important to keep in mind when normalizing the data in the coming checkpoints.

## Conclusion

The most difficult part of this assignment was first getting acclimated with the table structures and knowing which joins were necessary for each analysis. Additionally, learning SQL for those of us who weren&#39;t versed in this language took the most time and effort. But overall our questions were reasonably answered by the data we have at our disposal. Moving forward we expect difficulties to shift towards data integration, interpretation and cleaning, as opposed to access and extraction.
