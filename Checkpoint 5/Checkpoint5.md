# Group 2 Checkpoint 5 - Neural Networks

## Intro

For this checkpoint, our proposal mentioned several different ideas, but we decided to focus in on one specific goal that we mentioned - to build a model that predicts if an officer whom we had classified as a bad cop based on a history of his repetitive abusive behavior, was involved in a settlement case on the basis of the settlement description. In other words, we want to see if we can predict if a &quot;bad&quot; officer was involved in a settlement purely from the text description.

## Analysis

To tackle this task, we used the Keras API in Tensorflow to build a neural network. Our overall objective with this exercise was to evaluate if the behaviour leading to a settlement is an indicator of a repetitive behaviour pattern of the perpetrators, and whether the settlement description alone was enough to make this classification.

On the basis of the analysis done in the earlier checkpoints, we already had the data to classify the officers as good or bad. Using that classification and the relationship between the badge number and officer id, we were able to create a view using the settlement and settlement\_officer table that would indicate if a bad officer was involved in each of the settled cases. The SQL join required to create this dataset is included in the queries document.

**Figure 1** : **SQL View integrating Settlement Data with previous analyses on whether officers are bad or not (column &#39;is\_bad\_officer\_involved&#39;)**

<img src="/Checkpoint 5/images/ch5fig1.png">

When we matched the settlements with our list of repeat offenders, we realized only ~5% of the settlements involved the repeat offenders. This was counterintuitive at first because we figured they would account for more settlements due to the fact that they cause more trouble. But thinking about it more, we can attribute the low % as a reason for how the &#39;repeat offender&#39; officers were able to hide under the radar and stay employed on the force while committing offences simply by not being involved in settlements.

To feed the description text of each settlement into the neural net, we had to first embed the strings into a largely multi-dimensional vector. To do this, we used the universal sentence encoder in the tensorflow\_hub package, which can be found here: [https://tfhub.dev/google/universal-sentence-encoder/2](https://tfhub.dev/google/universal-sentence-encoder/2)

We choose this embedding algorithm for a variety of reasons, but the main two were that - it is a very high level call that allows us to use the Keras API, and it performs well on larger bodies of text compared to something more traditional like Word2Vec. The result of embedding the text content returned vectors with 512 dimensions for each settlement. In addition to this dataset, we appended a binary value of 0 or 1 to indicate the involvement of a bad officer, and this is what was fed into the neural net as the input. The model we developed is composed of 3 layers using the &#39;relu&#39; activation function and 300 units each, and a last layer using the sigmoid activation function with 1 unit.

## Results

Training this model on 80% of the dataset, we received 94.6% accuracy on the training set after 10 epochs, and 91.5% accuracy on the held-out testing set.

**Figure 2: Result of training and testing the Deep Neural Net (training accuracy of 94.7% and testing accuracy of 91.5%)**

<img src="/Checkpoint 5/images/ch5fig2.png">

## Conclusion

We tweaked the hyperparameters to check for any variations in the results but we got very consistent results. We tried adding significantly more units per layer and number of layers, which increased our training time, but again, we did not produce any significant change in the model. When looking at the predictions, we noticed that there was an extreme bias to classify the settlement as **not** involving a &quot;repeat offender&quot; officer. This led us to the conclusion that the model is not able to predict if there is a bad cop involved in the settlement, rather it is just assigning the mean value of the involvement field. Hence, we believe at this point that the description of the abuse that is available in the settlement record is not alone a good enough criterion to determine if the abuser has a pattern of overstepping. This in addition to the realization that only 5% of the settlements involved &#39;repeat offending&#39; officers are important insights into understanding how repeat offenders might try and hide their actions.
