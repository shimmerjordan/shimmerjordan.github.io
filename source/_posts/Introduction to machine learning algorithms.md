---
title: Introduction to Machine Learning Algorithms
date: 2020-11-4 18:36:11
tags: 
	- Machine Learning
	- Algorithms
categories: Machine Learning
id: MLAIntro
---

There are too many algorithms involved in artificial intelligence, and there are quite a few used in machine learning, so how can we choose the right algorithm?

This blog provides a brief introduction to machine learning algorithms, with specific descriptions and notes to be added later. 

<!-- more -->

## Introduction

There are too many algorithms involved in artificial intelligence, and there are quite a few used in machine learning, so how can we choose the right algorithm?

That is the question. In the following, I will take you through what algorithms are available and how to choose one.

The outline of this Part

1. Classification from the perspective of machine learning problems

2. Classification from the functional perspective of the algorithm

3. Machine learning algorithm decision tree

## Classification from the perspective of machine learning problems

We start by classifying the machine learning problem itself in terms of the following types of algorithms.

### Supervised learning

A large part of the problem in machine learning falls under the category of supervised learning, which, to put it simply and colloquially, is that given a training sample in which each sample input x corresponds to a deterministic outcome y, we need to train a model (mathematically, an x → y mapping f) that allows us to make predictions about the outcome y′ given an unknown sample x′.

If the prediction result is a discrete value (often a category type, such as spam/general mail in the mail classification problem, such as users will/won't buy a product), then we call it a classification problem; if the prediction result is a continuous value (such as house price, stock price, etc.), then we call it a regression problem.

There are a range of machine learning algorithms that are used to solve supervised learning problems, such as the most classical ones for classification problems such as plain Bayes, logistic regression, support vector machines, etc.; for example, linear regression for regression problems and so on.

### Semi-supervised learning

This type of problem gives training data that is partly labeled and partly unlabeled. We want to learn the organizational structure of the data and also make predictions accordingly. The corresponding machine learning algorithms for such problems are Self-Training, Transductive Learning, Generative Model, and so on.

In general, the most common are the first two types of problems, and some machine learning algorithms corresponding to the first two types of problems are as follows.

![BgHB7D.png](https://s1.ax1x.com/2020/11/04/BgHB7D.png)

## Classifying algorithms in terms of their functionality

We can classify machine learning algorithms in terms of their common features (E.g., function, mode of operation). In the following, we'll go ahead and classify algorithms based on their commonality. It should be noted, however, that our following categorization method may have a strong bias towards classification and regression, which are the most commonly encountered problems.

### Regression Algorithms

![B23Sd1.png](https://s1.ax1x.com/2020/11/05/B23Sd1.png)

A regression algorithm is a class of algorithms that obtains the best way to combine the input features by minimizing the difference between the predicted value and the actual result value. For continuous value prediction, there is linear regression, etc. For discrete value/category prediction, we can also consider logistic regression as a type of regression algorithm:

- Ordinary Least Squares Regression(OLSR)
- Linear Regression
- Logistic Regression
- Stepwise Regression
- Locally Estimated Scatterplot Smoothing(LOESS)
- Multivariate Adaptive Regression Splines(MARS)

### Instance-based Algorithms

![B235lD.png](https://s1.ax1x.com/2020/11/05/B235lD.png)

By instance-based algorithms here, I mean the model we finally build that still has a strong dependence on the original data sample instance. These types of algorithms, when making predictive decisions, typically use some type of similarity criterion to compare the similarity of the sample to be predicted to the original sample, and then give the corresponding prediction. Common example-based algorithms include:

- k-Nearest Neighbour (kNN)
- Learning Vector Quantization (LVQ)
- Self-Organizing Map (SOM)
- Locally Weighted Learning (LWL)

### Decision Tree Algorithms

![B23x1S.png](https://s1.ax1x.com/2020/11/05/B23x1S.png)

A decision tree class algorithm will build a tree containing many decision paths based on the raw data features. The prediction phase selects the paths for decision making. Common decision tree algorithms include:

- Classification and Regression Tree (CART)
- Iterative Dichotomiser 3 (ID3)
- C4.5 and C5.0 (different versions of a powerful approach)
- Chi-squared Automatic Interaction Detection (CHAID)
- M5
- Conditional Decision Trees

### Bayesian Algorithms

![B28LDJ.png](https://s1.ax1x.com/2020/11/05/B28LDJ.png)

Bayesian-like algorithms are referred to here as algorithms that implicitly use Bayesian principles in classification and regression problems. Including.

- Naive Bayes
- Gaussian Naive Bayes
- Multinomial Naive Bayes
- Averaged One-Dependence Estimators (AODE)
- Bayesian Belief Network (BBN)
- Bayesian Network (BN)

### Clustering Algorithms

![B2GEVA.png](https://s1.ax1x.com/2020/11/05/B2GEVA.png)

What clustering algorithms do is to cluster the input samples into 'clusters' of 'data' around some center in order to discover some patterns in the structure of the data distribution. Commonly used clustering algorithms include.

- k-Means
- Hierarchical Clustering
- Expectation Maximisation (EM)

### Association Rule Learning Algorithms

![B2GRxO.png](https://s1.ax1x.com/2020/11/05/B2GRxO.png)

Association rule algorithms are a class of algorithms that attempt to extract, the rules that best explain the observed correlations between training samples, i.e., to gain knowledge of dependencies or associations between an event and other events, common association rule algorithms are.

- Apriori algorithm
- Eclat algorithm

### Artificial Neural Network Algorithms

![B2JeeJ.png](https://s1.ax1x.com/2020/11/05/B2JeeJ.png)

This is a class of algorithms inspired by the way neurons work in the human brain. It is important to mention that I have singled out "deep learning", and that the artificial neural network in this case favours more traditional perception algorithms, which include.

- Perceptron
- Back-Propagation
- Radial Basis Function Network (RBFN)

### Deep Learning Algorithms

![B2JMJx.png](https://s1.ax1x.com/2020/11/05/B2JMJx.png)

Deep learning is a very hot field of machine learning in recent years, and it usually has a deeper level and more complex structure compared to the artificial neural network algorithms listed above. This type of algorithm is widely used in computer vision.

- Deep Boltzmann Machine (DBM)
- Deep Belief Networks (DBN)
- Convolutional Neural Network (CNN)
- Stacked Auto-Encoders

### Dimensionality Reduction Algorithms

![B2JGOe.png](https://s1.ax1x.com/2020/11/05/B2JGOe.png)

In a way, dimensionality reduction algorithms are actually somewhat similar to clustering in that they are also attempting to discover the inherent structure of the original training data, but the dimensionality reduction algorithm is attempting, with less information (lower dimensional information) to summarize and describe much of the original information.

Interestingly, dimensionality reduction algorithms are generally useful in visualizing data, or in reducing the computational space of data. It is used as a machine learning algorithm, and many times it is used to process the data first and then imbibe other machine learning algorithms to learn. The main dimensionality reduction algorithms include.

- Principal Component Analysis (PCA)
- Principal Component Regression (PCR)
- Partial Least Squares Regression (PLSR)
- Sammon Mapping
- Multidimensional Scaling (MDS)
- Linear Discriminant Analysis (LDA)
- Mixture Discriminant Analysis (MDA)
- Quadratic Discriminant Analysis (QDA)
- Flexible Discriminant Analysis (FDA)

### Ensemble Algorithms

![B2JwfP.png](https://s1.ax1x.com/2020/11/05/B2JwfP.png)

Strictly speaking, it's not really a machine learning algorithm, but more of an optimization tool/strategy that usually combines multiple simple weak machine learning algorithms to make more reliable decisions. Take the classification problem, for example, it is intuitively understood that a single classifier can be wrong and unreliable, but if multiple classifiers vote, then it is much more reliable. Common approaches to model fusion enhancement include.

- Random Forest
- Boosting
- Bootstrapped Aggregation (Bagging)
- AdaBoost
- Stacked Generalization (blending)
- Gradient Boosting Machines (GBM)
- Gradient Boosted Regression Trees (GBRT)

## Decision trees for machine learning algorithms

![B2Jff0.png](https://s1.ax1x.com/2020/11/05/B2Jff0.png)

First of all, if the sample size is very small, there is no way for all machine learning algorithms to "learn" common rules and patterns from it, so getting more data is king. Then according to the problem is unsupervised learning and continuous/discrete value prediction, it is divided into four method classes: classification, clustering, regression and dimensional reduction, and each class has a different treatment according to the specific situation.

With this decision tree, it is easy to choose the right algorithm based on the data you have and the purpose you want to achieve.

And this image is from https://scikit-learn.org/stable/tutorial/machine_learning_map/

## Related work

The October 2014 issue of JMLR has a fabulous article, Do we Need Hundreds of Classifiers to Solve Real World Classification Problems? testing the performance of 179 classification models on 121 data from all of UCI. Random Forests and SVM (Gaussian kernel, with LibSVM version) were found to perform best.[Do we Need Hundreds of Classifiers to Solve Real World Classification Problems?](https://jmlr.org/papers/v15/delgado14a.html)

