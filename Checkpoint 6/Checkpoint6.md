# Chicago Police Department - Checkpoint 6 - Visualization

To continue our analysis on repeat offending officers, we took on the task of exploring if there is any significant difference between the number of awards given to officers we have labeled as &#39;repeat offenders&#39; as compared to the others. For the purposes of simplification, we have labeled the &#39;repeat offenders&#39; as **&#39;Bad&#39;** in the visualizations, and the other cohort as **&#39;Good&#39;**. Our hypothesis was that Bad officers will not have as many awards comparatively - and if this hypothesis is proved wrong, this may be an indicator that there is a level of systemic hiding of these repeat offenders.

As an initial gut check, we took a look at how many awards each of the officers have, and split them up between the two groups of officers. As expected, the _&#39;Bad&#39;_ officers received less awards on average, as depicted in Figure 1. However, the range of the two groups is not that different, which was surprising. Though the majority of the bad officers are clustered at a lower number of awards, there still are a significant number of bad officers that have high levels of awards.

**Figure 1:** _Box and Whisker Plot of number of awards for bad and good officers_

<img src="/Checkpoint 6/images/OfficerAwards.png" width="80%">

We then took a look by &#39;Award Type&#39; to see how each award is distributed across the two groups of officers. Our results are displayed below in Figure 2. As seen in the chart, there are two award types that have a higher percentage given out to _&#39;Bad&#39;_ officers: _Chicago Police Leadership Award_ (22% given to Bad officers) and _Thomas Wortham IV Military And Community Service Award_ (25% given to Bad officers). Overall, these two awards are rare, but it is interesting that a relatively high proportion are still given to officers we deem as _&#39;Bad&#39;_.

**Figure 2:** Award Distribution across Bad and Good Officers by Award Type

<img src="/Checkpoint 6/images/AwardType.png" width="80%">

We then began taking a look at the award distribution across the percentile rankings included in the CPDB. Within CPDB, each officer has a percentile ranking in four categories: Civilian Allegations, Honorable Mentions, Internal Allegations and Tactical Response Reports (TRRs).  For each of these metrics, we took a look at how the awards are spread across the bad vs. good officer distribution. Across each measure, having a higher percentile indicates that a given officer has received higher amounts of the associated value (higher number of allegations, higher number of honorable mentions,etc).

As a baseline, we first plotted the award distribution across the &#39;Honorable Mention Percentile&#39;.

**Figure 3:** Award distribution vs. Honorable Mention Percentile

<img src="/Checkpoint 6/images/HonerableMentionPercentile.png" width="80%">

As you can see in Figure 3, the result is as expected, and the awards increase as the percentile of honorable mention increases. It is likely the &#39;Honorable Mention Percentile&#39; was calculated by CPDB with award counts.

From here, we took a look at the award distribution across the Civilian Complaint percentiles.

**Figure 4:** Award distribution across Civilian Complaint Percentile 
<img src="/Checkpoint 6/images/ComplaintPercentile.png" width="80%">

The overall distribution is surprising because it looks like there is a positive correlation between having civilian complaints and having more awards. We incorrectly hypothesized this would be a negative correlation. Also, there is a large proportion of awards going to the _&#39;Bad&#39;_ officers as the percentile increases (on the x axis). This makes sense given that the bad officers have high Civilian Complaint percentiles.

Next, we looked at the award distribution across the Internal Allegations percentiles:

**Figure 5:** Award distribution across Internal Allegation Percentile

<img src="/Checkpoint 6/images/InternalPercentile.png" width="80%">

From this figure, we can conclude that the officers who get more internal allegations get significantly less awards. We were not surprised by this result, as this distribution is what we were expecting for both civilian complaint percentile as well as internal allegation percentile. It seems that awards are given at least somewhat based upon the number of internal allegations there are against an officer.

Lastly, we looked at the award distribution across the TRR percentiles:

**Figure 6:** Award distribution across TRR Percentile

<img src="/Checkpoint 6/images/TRRPercentile.png" width="80%">

From this, it looks like there isn&#39;t much of a correlation between the TRR percentile of officers and them receiving awards. This makes sense to us because TRRs are created in result of officers documenting incidents that may or may not be a result of them being a bad officer. We would not go as far to say officers with more TRRs are more problematic. And because of this, it makes sense how TRR percentile has little correlation with officer awards.

**Conclusion**

The most significant analysis in this checkpoint is comparing the distribution of awards across the Civilian Complaint percentile and the Internal Allegations percentile. From the results, it looks like internal allegations have a strong effect in preventing officers from getting awards, but civilian complaints do not have the same effect.

**Lessons Learned**

Using Tableau, our biggest issue was using the location polygons in CPDB. We initially wanted to create a map representation of the allegations between the good and bad officers, but were unable to get the spatial info from CPDB to work with Tableau. Also, it took a while to get the formatting of the axes in the figures to be big enough to be readable when included in our writeup. As for positives, Tableau was very user friendly and made it easy for us to do exploratory analyses at a very quick and efficient pace.
