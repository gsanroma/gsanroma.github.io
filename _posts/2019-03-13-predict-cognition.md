---
title: 'can we predict cognitive abilities from high-dimensional brain data using conventional (unpenalized) regression ?'
date: 2019-03-13
permalink: /posts/2019/03/predict-cognition/
tags:
  - machine learning
  - neuroimaging
---

In this post I explain the challenges in predicting cognitive abilities from high-dimensional brain data using conventional (unpenalized) regression, but the ideas here explained apply generally to the modeling of relationships between any input and output data in high dimensions.
Conventional regression techniques are not useful in high-dimensional settings.
I show how machine learning provides an explanation for this issue and proposes a solution.

Cognitive abilities can be measured across multiple domains such as _working memory_, _processing speed_, _episodic verbal memory_ and _executive function_.
One may wonder how they are determined by factors such as age and sex.
One may also wonder how much left is determined by brain structure.
To answer these questions, we can use regression analysis to assess the association between cognition and these factors or, put another way, to predict cognition using these factors as input.

To get an idea of the data, we plot different cognitive abilities with respect to age for a subset of participants in the [Rhineland Study](https://www.rheinland-studie.de/) (variables have been standardized to zero-norm, unit standard deviation).

![](/images/blog/2019-03-13-predict-cognition/plot_cog_age.png)

Let's define a function that predicts cognition based on age.

![](/images/blog/2019-03-13-predict-cognition/fn_cog_age.png)

(we are assuming a linear relationship between age and cognition)

In regression analysis, we need a set of examples with pairs of values for age (![](/images/blog/2019-03-13-predict-cognition/x.png)) and cognition (![](/images/blog/2019-03-13-predict-cognition/y.png)). 
We estimate the beta coefficients that make up the function as follows:

![](/images/blog/2019-03-13-predict-cognition/fn_opt.png)

That is, we seek the function _f()_ that predicts with minimum error (ie, squared difference) the cognitive abilities _y_ in our sample.
We plot the predictions for our sample in red below.
Errors are represented as black lines joining predictions with true values.
(we actually compute 4 different functions, one for each cognitive domain)

![](/images/blog/2019-03-13-predict-cognition/plot_cog_age_pred1.png)

We see that predictions look like a straight line.
This is because we have used a linear function.
Based on the slope, we may extract conclusions about the strength of this association.
We won't go into more detail on that here.
Let us only note that in general, the greater the magnitude of the beta coefficient (ie, the slope of the line), the more important is the associated factor in the prediction.

To further improve prediction, we may include other factors such as sex.

![](/images/blog/2019-03-13-predict-cognition/plot_cog_age_pred2.png)

Adding sex brings the predictions a bit closer to the true values.

If available, we may further include information about brain structure.
Luckily, we have information about thickness across 50+ cortical brain structures in our sample.
The cortical brain structures are depicted in different colors in the parcellation below:

![](/images/blog/2019-03-13-predict-cognition/rois.png)

Below we show the evolution of the prediction by gradually adding all the available variables, starting by age and sex and following with the cortical thickness measures.

![](/images/blog/2019-03-13-predict-cognition/lars_anim.gif)

As we include more variables the fit gets better until eventually we obtain perfect prediction in our sample.

At this point we have created 4 functions _f()_ (one for each cognitive domain) that receive age, sex and cortical thickness as input and predict cognitive performance as output.
Does this mean that we now exactly predict the cognitive abilities of any person given their age, sex and cortical measures ?
**Of course not**. 
As a counter-example, consider the how unlikely it is that one participant from our sample would perform exactly the same if they were to repeat the cognitive testing.
However, our function would always predict the same value, as long as age, sex and cortical thickness remain constant.

Let us make clear the distinction between **learning** and **prediction**.
- Learning involves the creation of the functions relating biological inputs to cognition using an example set of _training_ data
- Predicting means guessing the cognitive performance from the biological inputs, using the functions learned in the previous step

Note that prediction can be done in _the same_ or _another_ dataset as the one used for learning the functions.
To see how well the learned functions capture any meaningful relationship, let us predict onto a different set of participants.

![](/images/blog/2019-03-13-predict-cognition/plot_oos.png)

The errors are so large that do not even fit in the plot.
**What has happened ?**
With so many inputs, the function is flexible enough to behave in any way we want.
As result, it only learns relationships between inputs and output specific to the training data but that do not hold to other participants.
In machine learning terms, this is known as [overfitting](https://en.wikipedia.org/wiki/Overfitting).

**To make sure that high-dimensional functions (>10-12 dimensions) capture any meaningful relationship beyond the training dataset, we need to test for generalization.**
When using only a few variables the function is already constrained enough to be protected against over-fitting (although testing for generalization wouldn't harm in any case).

To understand this issue consider the following:
The true relationship between input and output (if any) hold in any representative sub-set of the data.
Therefore, the learning procedure that reveals such true relationship should yield stable models regardless of the training dataset.
Highly flexible models contradict this premise by yielding results that depend too much on the particular training set.
Note also that, too stable models (eg, a constant function) would also fail for the opposite reason, ie, not explaining anything useful about the data.
Therefore, we seek a trade-off between adaptability to the data and stability of the model across training sets.
This is known as the [bias-variance tradeoff](https://en.wikipedia.org/wiki/Bias%E2%80%93variance_tradeoff), a central issue in machine learning.

The bias-variance trade-off suggests that we should sacrifice data fitting for the sake of a more stable model across training sets.
Although it may not be obvious how allowing for training errors improves generalization, I would argue it is otherwise unreasonable to pursue a perfect fitting for the following reasons:
- _missing inputs_: we may be missing relevant biological factors that determine cognitive performance
- _wrong model assumptions_: the relationships between the inputs and outputs in our model (linear in our example) may not be correct
- _random errors_: the same person may perform differently in the same test for reasons that cannot be determined

Now that I hope I have convinced you, you may ask 1) how we control the degree of flexibility of our function and 2) how to select the appropriate degree ?
The first question relates to the topic of [regularization](https://en.wikipedia.org/wiki/Regularization_(mathematics)).
Best-feature-subset selection methods such as the [lasso](https://en.wikipedia.org/wiki/Lasso_(statistics)) address this issue by restricting the model to only use a sub-set of the inputs.

The second question on how to decide on the appropriate degree of flexibility relates to the topic of [model selection](https://en.wikipedia.org/wiki/Model_selection).
We want to select the model that will perform the best on new data and, for that, we need first to estimate the generalization error.
One of the most popular method to estimate the generalization error, and thereby inform model-selection, is [cross-validation](https://en.wikipedia.org/wiki/Cross-validation_(statistics)), which consists in performing _learning_ and _prediction_ in different partitions of the dataset.
 
In the following, I describe how we go about selecting the model for our problem:
- Partition the dataset in 3 parts for training, validation and test.
- Using the training set, we learn different versions of the model, by varying the degree of complexity (with the lasso).
- Using the validation set, we measure prediction accuracy, ie, how well the model learned on the training data predicts cognition on the validation set
- We select the complexity level with best prediction accuracy on the validation set and re-learn the model using both training and validation sets

Now, we can use the learned model to predict the cognitive performance on the data from the test set, which has not been used at all, and which represents new data that may come in the future.
The image below shows the prediction on a test set, by our model selected by the cross-validation procedure above.

![](/images/blog/2019-03-13-predict-cognition/plot_lasso.png)

Now that we know our model is not overfitted to the training data, we can inspect the selected variables to ascertain what brain structures are important for predicting cognition.
The image below shows the brain structures selected by lasso, where blue / red denote respectively that the  thickening / thinning of the corresponding structure is associated with higher cognitive performance in each respective domain.


![](/images/blog/2019-03-13-predict-cognition/ef_dorsal.png)
![](/images/blog/2019-03-13-predict-cognition/evm_dorsal.png)
![](/images/blog/2019-03-13-predict-cognition/ps_dorsal.png)
![](/images/blog/2019-03-13-predict-cognition/wm_dorsal.png)

