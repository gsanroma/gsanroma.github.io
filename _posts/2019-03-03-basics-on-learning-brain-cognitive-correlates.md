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
![](/images/blog/2019-03-03-basics-on-learning-brain-cognitive-correlates/cog_age.png)

