---
title: 'basics on learning brain cognitive correlates'
date: 2019-03-03
permalink: /posts/2019/03/basics-on-learning-brain-cognitive-correlates/
tags:
  - machine learning
  - neuroimaging
---

In this post I show how to find brain structures that are associated with cognitive abilities independently of factors such as age, sex and education.
In the process of solving this problem with standard statistical techniques we come across an issue related to the [curse of dimensionality](https://en.wikipedia.org/wiki/Curse_of_dimensionality).
Finally, I show how machine learning comes to hand to solve this issue.

Cognitive abilities can be measured across multiple domains such as _working memory_, _processing speed_, _episodic verbal memory_ and _executive function_.
Human abilities in such domains may be influenced or confounded by factors such as age, sex and education.
One may want to determine the importance of these factors on cognition and also how much left is determined by brain structure.
To answer the first question one may resort to regression analysis.

The image below shows plots of different cognitive abilities with respect to age for a subset of participants in the [Rhineland Study](https://www.rheinland-studie.de/).

![](/images/blog/2019-03-03-basics-on-learning-brain-cognitive-correlates/plot_cog_age.png)

We can see that cognitive abilities generally decrease with age.
In order to see how much cognitive abilities are determined by age, we can define one function for each domain that predicts cognition only from age.
Making the plausible assumption that the relation is linear, we may write:

![](/images/blog/2019-03-03-basics-on-learning-brain-cognitive-correlates/fn_cog_age.png)

This is a function to predict cognition using as input _x_, the age for a given subject.
We estimate the beta coefficients making up this function by using regression analysis as follows:

![](/images/blog/2019-03-03-basics-on-learning-brain-cognitive-correlates/fn_opt.png)

That is, we seek the function _f()_ that predicts more accurately (ie, with minimum squared difference) the cognitive abilities _y_ of our subjects (indexed with subscript _i_) using their age as input.
