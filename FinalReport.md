# Chicago Police Department - Repeat Offenders Project

_Note: This document serves as the formal write up for our project, to meet the 10 page final submission requirement. For full queries, analysis files, and data outputs please visit our_ [_Github_](https://github.com/invinst/nu-collaboration/tree/master/02_Group) _page._

## **Section 0:** _Introduction_

In this project, our team set out to try and create a methodology to identify &quot;Repeat Offenders&quot; within the law enforcement system in Chicago. We often call these types of officers &quot;good&quot; and &quot;bad&quot; officers throughout the duration of the report for simplicity - although this simplification is only for the context of the report.

There were several different motivations behind this particular goal. The first is that in other abuse investigations performed in history, one common trait is that a small group of perpetrators often commit a disproportionate number of infractions, leading to what we call a &quot;bifurcation of mistreat&quot;. This is something that we wanted to examine in this case - as understanding the topography of this complex legal system is very important. Furthermore, this anomaly can often be caused by factors that are difficult to resolve - specifically, either inadvertent protection of the offenders due to the structure of the rules in place, or direct systematic protection from leaders in power. However, for the latter, leaders may be heavily incentivized to act in an unethical manner, thereby putting decision makers in very difficult situations at no fault of their own. These are two major issues that can be largely alleviated by outside intervention. Finally, such perpetrators often pose the highest overall risk to the city, especially from a financial standpoint. Hence, even a slight increase in determining such offenders can drastically affect overall performance of the entire system by reducing the settlement money paid out.

Overall this goal has major upsides for both the department and the citizens associated.

## **Section 1:** _Initial Investigation using Officer Allegation History_

The first step in this project was selecting the main measure to dive deep on. After combing through the available resources, average number of allegations (against an officer) per year was chosen. The histogram below shows a high level view of the distribution for all officers.

**Figure 1.1** : Percent of Officers vs. Average Number of Allegations per Year

 ![](/02_Group/Images/final11.png)

With this simple analysis a few things are evident. First off, the vast majority of officers have very few allegations filed against them on an annualized basis - indicating the suspected bifurcation. However, this is not enough to successfully segment officers given the diversity of different operating environments faced by officers at scale.

Next, the relationship between internal TRR reports filled out by officers and their association with citizen submitted allegations was examined. More specifically, the goal was to determine if there are police officers who have a significant number of allegations against them, which don&#39;t have corresponding TRR police reports. In order to match the allegations to corresponding reports, it was required that the reports were from the same officer, and the reported incident dates were similar. In order to find a reasonable date range threshold to accommodate the inexact timeline of the reports, while also avoiding potential false positives, the number of matches that occur as the date threshold changes was analyzed. Figure 1.2 shows these results.

**Figure 1.2** : Number of Matched Allegations and TRRs vs. Date Range Threshold

 ![](/02_Group/Images/final12.png)

These results showed that a large majority of the allegations don&#39;t correspond to TRR&#39;s in the database (depending on timeline chosen, well over 50%). This is something we would absolutely look into if we were to continue this project - and it is something we find suspicious. Moving forward we focused in on our most stable and robust metric, average number of allegations per year.

## **Section 2:** _Allegation Normalization_

To account for the variance in crime activity that different officers face, the information that was previously discovered in Figure 1.1 had to be normalized. In order to accomplish this task the &quot;All Sworn in Officers&quot; snapshot served as a base for the initial analysis.

First, the district average number of allegations per officer per year was computed in Trifecta using the previous statistic of average number of allegations per year. From there, each officer&#39;s current average number of allegations was divided by their associated district average. Hence, if there was a higher district average, an individual officer&#39;s normalized number of allegations should go down. Below is a histogram of the results in conjunction with the original analysis.

**Figure 2.1** : Percent of Officers vs. Average Number of Allegations Per Year

 ![](/02_Group/Images/final21.png)

Here we see some interesting, but largely expected results. Holistically, the number of officers with a large number of normalized allegations shifts downwards. However, this shift creates a sort of skewed left normalized distribution. We were fairly surprised to see this fact, and specifically we were interested that there was still a significant amount of officers with over 1 allegation per year on average following district normalization.

Our next step was to normalize the number of allegations for each officer by the crime and arrest rate in each district. To do this, the following metrics for each district were calculated: total number of allegations, officers, arrests, and instances of crime (latter two required external DB link).

Two graphs were created, which are shown below. Each point on both graphs represents a separate district. On the y axis of each graph, is the district allegations per officer normalized by the number of arrests or crimes per officer within the district. On the x axis is the number of arrests or crimes per officer in that district. This gives a sense of how having more crimes or arrests per officer (an indicator of likelihood to use force) correlates to how many allegations are filed against associated officers on a per capita basis.

**Figure 2.2** : District Allegations Normalized by Arrests
 ![](/02_Group/Images/final22.png)
 
**Figure 2.3** : District Allegations Normalized by Crime Data
 ![](/02_Group/Images/final23.png)

One interesting finding that we were not expecting, was the trend of relative allegations as crime and arrest rates increases. We were expecting there to be a higher proportion of allegations for the districts that were in areas that had high levels of arrests and crime when factoring in number of officers. But from our analysis, there is actually a negative correlation.

One overarching takeaway from this analysis was the importance of normalizing when dealing with large datasets spanning large areas and groups of individuals. We found that rates of crime varied heavily across districts, so it would be unfair to compare the rate of allegations without taking their relative rates into consideration. However, after normalizing, we found that districts with high crime rates actually have a lower rate of allegations relative to their crime rate per capita. Note, for further analysis we used a cutoff of 1.499 normalized allegations per year to segment officers.

## **Section 3:** _Group Behavioral Analysis_

Next, we dug into the group dynamic of the officers and answered two questions. First, how are allegations clustered in groups of officers (co-accusal)? Second, how do individual officers behave over time after receiving some number of allegations?

Co-accusals of officers was investigated from a graph perspective. Modeling each officer as a node, co-accusals were represented as an undirected edge with a weight of 1, and edge weight increased if a set of officers had more than one co-accusal.

Using GraphX in Spark, this representation was created and two core analyses were run. First, a connected components analysis to create distinct groups of officers was executed. In this analysis, an officer is added to a cluster if any connection is made to one or more officers in that particular cluster. This analysis benefitted from a one year time frame, because it was a sufficient lapse in time to create clusters - but not so long that all officers eventually were grouped into the same &quot;mega&quot; cluster. From this method, about 600 connected components were formed, from approximately 2,100 officers. The total number of officers in each cluster was then calculated, along with the total edge weight. We have plotted both metrics on the histogram below.

**Figure 3.1** : Connected Components Analysis (At Officer Level)

 ![](/02_Group/Images/final31.png)

Looking at this graph, some interested results were observed. First, it is clear that there are several clusters which have very high numbers of allegations for the given number of officers in that cluster, particularly in the 15 - 25 officer range (circled in maroon). This indicates that potential repeat offenders will actually work together over time. This is further demonstrated by adding a linear regression line to get a general sense of the average number of allegations (correlated to weight) per officer in each cluster.

To further examine the connectedness of the officers, the PageRank algorithm was deployed using Graphframe in Spark. Given that the process is very computationally expensive, the smaller time frame of a year allowed completion of the process in a reasonable amount of time. The same graph representation described above was used - but instead of weighting edges, each edge represented a weight of 1, and multiple edges from one node to another were allowed.

**Figure 3.2** : PageRank Analysis (At Officer Level)

  ![](/02_Group/Images/final32.png)

Looking at the figure of PageRank scores, a similar behavior can found. Here, we see there is an inflection point past about 1.2 PageRank score where officers are highly connected - i.e. they have many different claims against them with several different officers. This methodology represents an interesting way to identify high risk officers - and we would recommend potentially extending this process internally within the Chicago police department.

## **Section 4:** _Advanced Predictive Modeling with Machine Learning_

In terms of applying machine learning to our dataset, our goal was to explore our prior officer segmentation through supervised and unsupervised machine learning methods.

First, we explored utilizing k-means clustering. To complete this task, two of the previously computed attributes were used - average number of allegations per year and normalized number of allegations per year at an officer level. After running the K-Means algorithm and examining the two groups, we were very happy to see that there was only a difference of 887 officers out of about 14,000 officers in our original normalization cutoff analysis.

For our supervised method, a decision tree model was used to predict repeat offenders using the following features: demographic, tenure, rank, number of awards, district/beat, normalized allegations per year, and average allegations per year.

To do this, we performed joins on the overall table from checkpoint 2 with the normalized allegation counts, with the k-means clustering classification as the target output. With all of the relevant features for the prediction in one table, the columns were vectorized, which required string indexing on the attributes: Rank, Tenure and Race. By using the clustering classification as the output label, a decision tree model in Spark was created, with a 70/30 cross-validation split on our data.

The specific decision tree model utilized was a Gradient Boosted Forest (Trees in Spark) with Regression. There were 100 different trees created in our specific ensemble method. Using the feature importances attribute, the following values of significance for each attribute were obtained:

**Figure 4.1:** Gradient Boosted Forest feature importance

| Average\_Allegation\_Count: 0.4296 | Normalized\_Allegation\_Count: 0.2971 | Number\_of\_awards: .0988 |
| --- | --- | --- |
| Tenure: 0.0582 | Age: 0.0393 | Rank: 0.026 |
| District\_Number: 0.0073 | Gender: 0.0008 | Race: N/A |

Looking at the above significance, it was interesting that the non-normalized allegations resulted in a much higher significance compared to the normalized version. Additionally, we were surprised to see that the number of awards an officer had was the largest impact outside of our core indicators. We also did not receive results from Race - indicating that the attribute was thrown out due to insignificance, something we were very surprised at.

With all three of our models trained (K-means clustering, normalized allegation count, GBT regression), we wished to see how they compared to each other. We took a random sample of 1,000 officers from the Decision Tree output and matched our results to the previous models - our own evaluation  and the K-means clustering. We were very pleased to see that our GBT regression matched to our K-means results with an accuracy of about 99%, and about 93% to our original cutoff. This further validated that our original normalized model with a top 7% threshold cutoff was significant, and that our K-means results were an even better nuanced indicator.

## **Section 5:** _Advanced Predictive Modeling with Neural Networks_

To see if there are any trends or commonalities between the settlements &#39;repeat offenders&#39; are involved in, we built a model that predicts if a &quot;bad officer&quot; was involved based only off the settlement description.To tackle this objective, we used the Keras API in Tensorflow to build a neural network. Our overall objective with this exercise was to evaluate if the behaviour leading to any settlement was an indicator of a repetitive behaviour pattern of the perpetrators, and whether the settlement description alone was enough to make this classification.

Using the classification obtained from our K-means clustering analysis and the relationship between the badge number and officer id, a view using the settlement and settlement\_officer table that would indicate if a bad officer was involved in each of the settled cases was created.

When we matched the settlements with our list of repeat offenders, only ~5% of the settlements involved the repeat offenders cohort. This was counterintuitive at first because logically repeat offenders should account for more settlements due to the fact that they cause more trouble. However, diving deeper, one can attribute the low percentage as a reason for how the &#39;repeat offender&#39; officers are able to hide under the radar and stay employed on the force while committing a large number of offences.

To feed the description text of each settlement into the neural net, the strings needed to be embedded into a largely multi-dimensional vector. To do this, the universal sentence encoder in the tensorflow\_hub package was used, which can be found here: [https://tfhub.dev/google/universal-sentence-encoder/2](https://tfhub.dev/google/universal-sentence-encoder/2)

This embedding algorithm was chosen for a variety of reasons, but the main two were that - it is a very high level call that allows easy use of the Keras API, and it performs well on larger bodies of text compared to something more traditional like Word2Vec. The result of embedding the text content returned vectors with 512 dimensions for each settlement. In addition to this dataset, we appended a binary value of 0 or 1 to indicate the involvement of a bad officer, and this is what was fed into the neural net as the input. The model developed is composed of 3 layers using the &#39;relu&#39; activation function and 300 units each layer, and a final output layer using the sigmoid function with 1 unit.

Training this model on 80% of the dataset resulted in a 94.6% accuracy on the training set after 10 epochs, and 91.5% accuracy on the held-out test set. We tuned the hyperparameters to check for any potential optimization, but got very consistent results. We also tried adding significantly more units per layer and a number of additional layers, which increased our training time, but again, this did not produce any significant change in the test accuracy. When examining the predictions, we noticed that there was an extreme bias to classify the settlement as _not_ involving a &quot;repeat offender&quot; officer.

This led us to the conclusion that the model is not able to predict if there is a bad cop involved in the given settlement - and is simply just assigning the mean value of the involvement field. Hence, we believe at this point that the description of the abuse that is available in the settlement record is not alone a good enough criterion to determine if the abuser has a pattern of overstepping. If the project were to continue, it would be extremely interesting to rerun this model with a more evenly balanced dataset. Several different methods could be deployed to accomplish this - namely data stratification, and random sampling.

## **Section 6:** _Incentives Analysis - Punishment and Awards_

We next wanted to see how officers react towards negative and positive incentives. For the negative incentive, we analyzed the effect of getting reprimanded from an allegation. For the positive incentive, we analyzed the distribution of officers that receive awards.

To analyze the effect of reprimanding officers, the time interval in days between allegations for each officer was calculated. We then segmented the intervals by whether the officer was reprimanded for the initial allegation, and if the officer was categorized as Good or Bad. The allegation intervals for these two buckets of officers was aggregated, and the two histograms below show how they are distributed.

**Figure 6.1:** Histogram of intervals (days) between allegations after being Reprimanded (bad officers on right)

 ![](/02_Group/Images/final61.png)

**Figure 6.2:** Histogram of intervals (days) between allegations after not being Reprimanded (bad officers on right)

 ![](/02_Group/Images/final62.png)

These findings are summarized below by computing the overall averages for each interval:

**Figure 6.3:** Average Intervals Between Allegations

|   | **Overall Average Interval (Days)** | **Average Interval After Reprimanded Allegation (Days)** | **Average Interval After Non-Reprimanded Allegation (Days)** |
| --- | --- | --- | --- |
| **Repeat Offenders** | 145 | 177 | 142 |
| **Non-Repeat Offenders** | 398 | 481 | 393 |

Overall, this data supports the assertion that disciplinary action does have an effect on keeping officers in line. After being disciplined, on average, more time passes before they receive another allegation.

We then took on the task of exploring if there is any significant difference between the number of awards given to Bad Officers as compared to the Good Officers. Our hypothesis was that Bad Officers would not have as many awards comparatively - and if this was hypothesis proved wrong, awards would have been an indicator that there is a level of systemic hiding. As an initial gut check, we took a look at how many awards each of the officers have, and split them up between the two groups of officers, which is shown in figure 6.4

**Figure 6.4:** _Box and Whisker Plot of number of awards for Bad and Good Officers_

 ![](/02_Group/Images/final64.png)

Surprisingly, the Bad Officers have a higher number of awards on average. We suspect that the Good Officer average is pulled down by new officers who have not active long enough to accrue awards or complaints.

Within CPDB, each officer has a percentile ranking in four categories: Civilian Allegations, Honorable Mentions, Internal Allegations and Tactical Response Reports (TRRs).  For each of these metrics, we took a look at how the awards are disturbed across the bad vs. good officer distribution. Across each measure, having a higher percentile indicates that a given officer has received higher amounts of the associated value (higher number of allegations, higher number of honorable mentions, etc).

**Figure 6.5:** Award distributions across Honorable Mention, TRR, Civilian Complaint &amp; Internal Complaint Percentiles

 ![](/02_Group/Images/final651.png)

 ![](/02_Group/Images/final652.png)

As a baseline, we first plotted the award distribution across the &#39;Honorable Mention Percentile&#39;. As you can see, the result is as expected, and the awards increase as the percentile of honorable mention increases. It is likely the &#39;Honorable Mention Percentile&#39; was calculated by CPDB with award counts.

Looking at the award distribution across the TRR percentiles, it looks like there isn&#39;t much of a correlation between the TRR percentile of officers and the receival of awards. This makes sense to us because TRRs are created in result of officers documenting incidents that may or may not be a result of them being a Bad Officer. We would not go as far to say officers with more TRRs are more problematic. Hence, it makes sense how TRR percentile has little correlation with officer awards.

Looking at the Civilian Complaint percentiles, the overall distribution is surprising because there is non-intuitively a positive correlation between having civilian complaints and having more awards. Also, there is a larger proportion of awards going to the _&#39;Bad&#39;_ Officers as the percentile increases (on the x axis). One would think awards would be given less to officers that get the most complaints filed against them.

In contrast, there is a negative correlation when looking at internal allegation percentile distribution. We were not surprised by this result, as this distribution is what we were expecting for both civilian complaint and internal allegation percentiles. Stepping back, it seems that awards are given at least somewhat based upon the number of internal allegations against any given officer.This indicates that internal allegations have a strong effect in preventing officers from getting awards, but civilian complaints do not have the same effect. This is something that we would absolutely recommend looking into further if the project were to continue.

## **Section 7:** _Conclusion and Closing Thoughts_

Overall from our holistic analysis, we are confident in stating that there exists an outlier set of officers that exhibit a repetitive pattern of abusive behaviour against civilians. One of our most interesting finds is that the way awards are currently distributed does not significantly take into consideration the number of external allegations coming in against an officer, and focuses mainly on internal feedback. Two of our analyses (sections 4 &amp; 6) pointed to a high number of awards as an indicator of officers that are repeat offenders. This is troubling to us, as it is a critical sign that potential abusive officers are still being rewarded irrespective of their adverse behavior.

This investigation also shows that there is a possibility of a handful of officers working together when they exhibit troublesome behavior. Future work can focus on diving deeper on group behavior over time. This would be a strong element to determining if there is systematic hiding or inadvertent protection of repeat offenders occuring. Also, we found evidence which indicates that disciplinary action does deter Bad Officers from getting into trouble again, but is not enough to curtail overall behavior - this suggests that the department might need to alter how they punish such infractions. Preventative measures in terms of training and educating the officers of the consequences of problematic behaviour might go a long way, and could be a great low cost way of reducing risk.

Our team experienced a massive amount of personal learning during this project. Firstly, we learned how important managing and understanding provided data truly is - and how critical SQL mastery is to developing the necessary core concepts. Unlike typical homework assignments that involve pristine data sets, this project exposed us to a real world scenario where significant effort was put in to cleaning the data and determining the best way to pull multiple sources together. Completing tasks such as handling null values in the database, defining relations between different sources, and exploring alternative ways to complete our intended analysis was an amazing experience. Additionally, our group displayed how powerful &quot;out of the box&quot; methods can be when applying them correctly, and using interesting data. Overall we are extremely proud of the work completed, and would like to thank Professor Jennie Rogers for this opportunity!
