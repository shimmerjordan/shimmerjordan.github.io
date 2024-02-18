---
title: Comparison between Machine Learning Algorithms
date: 2020-11-06 18:36:11
tags: 
	- Machine Learning
	- Algorithms
categories: Machine Learning
id: MLAComparison
mathjax: true
---

# Comparison between Algorithms

It is better to teach a man to fish than to teach him to fish. This part will explain how to compare the performance of machine learning algorithms through "statistical tests". Once we have mastered this method, we can analyse the performance of the algorithms ourselves, without having to follow the crowd.

<!--more-->

Here I will begin by setting out the conclusions reached in this section as follows:

When comparing the performance of the two algorithms on multiple datasets:

- If the sample is paired (paired) and conforms to a normal distribution, the paired t-test is used preferentially (paired t test).
- If the sample does not conform to a normal distribution, but matches the pairing, use the Wilcoxon Signed Ranks test.
- If the samples are neither normally distributed nor paired, or even if the sample sizes are not equally large, try the Mann Whitney U test. it is worth noting that MW is used to process independent measures data, which is discussed on a case-by-case basis and analysed in depth later.

When comparing the performance of multiple algorithms on multiple datasets:

- If the sample meets the assumptions of ANOVA (experimental measure) (e.g., normal, equal variance), use ANOVA preferentially.
- If the sample does not meet the assumptions of ANOVA, use the Friedman test with the Nemenyi test for post-hoc.
- If the sample size is different, or if Friedman-Nemenyi cannot be used for a specific reason, try Kruskal Wallis with Dunn's test. It is important to note that this method is used to process independent measurements and is discussed on a case-by-case basis.

This part is structured as follows: (1-2) Reasons and pitfalls of algorithm comparison (3-4) How to compare two algorithms (5-6) How to compare multiple algorithms (7) How to choose a comparison method based on data characteristics (8) Introduction to the tool library.

## 1. Why do we need to compare algorithms' performance

George Box, a statistician, said: "All models are wrong, but some are useful" (All models are wrong, but some are useful). **In layman's terms, all algorithms have limitations, so there is no such thing as a "general-purpose optimal algorithm", only an algorithm that may be progressively optimal under certain circumstances.**

Therefore, it is important to evaluate algorithm performance and select the optimal algorithm. Unfortunately, statistical evaluation is not yet widespread in the field of machine learning, and many evaluations tend to be simple analyses on a single piece of data, thus proving to be of limited effectiveness.

## 2. Pitfalls in the assessment algorithm

First of all, we often talk about the need to choose a correct evaluation criteria, the common ones are: accuracy, recall, precision, ROC, Precision-Recall Curve, F1 and so on.

The choice of assessment criteria depends on the purpose and the characteristics of the dataset. On a more balanced dataset (where all types of data are approximately equal), there is little difference in the performance of these assessment criteria. And in cases where the data are heavily skewed, the selection of inappropriate assessment criteria, such as accuracy, can lead to results that look good, but are actually meaningless. For example, assuming the proportion of a rare blood group (2%), the model would only need to predict the entire sample as "non-rare", which would result in an accuracy of 98%, but would be meaningless. In this case, the choice of ROC or precision rate may be more appropriate. This knowledge is easy to understand and is described in many popular science books, so we won't repeat it.

Secondly we need to properly understand the measurement methods, common ones include:

- **Independent measures**: Observations from different samples are independent and not correlated.
- **Repeated measures**: The observations used in the sample are the same, it's just that the independent variables have different results on them.
- **Matched pair**: Use different observations in different samples, but try to make pairs of observations similar between samples.

For example, we want to analyse the effect of web browsing time (3 hours per day vs. 10 hours per day) on the performance of college students. If we use the same 20 students and observe the difference between their 3 and 10 hours per day, that's a repeated measure. If we choose 40 students, split them into two groups of 20 each, and then observe them separately that is an independent measurement. If we start with 20 students and then find 20 college students who are very similar to them and observe them in pairs that is paired similarity.

**We found that when the measurement is misinterpreted, the correct statistical means of analysis cannot be used.**

In this article we default that valuating the performance of different algorithms on **multiple identical datasets** is a **repeated measurement**, and exceptions will be discussed in Section 7. Also, the methods presented in this article can be used to **compare any evaluation criteria**, such as accuracy, precision, etc., and accuracy is discussed by default in this article.

## 3. Comparison of two algorithms: Inappropriate methods

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/11/05/BR5kDS.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Accuracy of the two algorithms on 14 datasets</div>
</center>

The figure above shows the accuracy of the two decision tree methods $ (C4.5, C4.5+m) $ on 14 datasets. So how should we compare the two algorithms? Let's start with a few erroneous (and inappropriate) methods:

**Inappropriate Method 1: Average each algorithm over all data sets and compare the sizes.** 

Reason for error: Our expectation that the algorithms are wrong on different data sets is not the same, so averaging is not meaningful. In other words, the data is not proportional(commensurate).

**Inappropriate method 2: Performing a paired sample t test.** 

Obviously, the t test is a statistical method that can be used to see if the average difference between the two methods on each data point is not equal to zero, but this method is inappropriate for several reasons:

- As with the average, errors on different datasets do not match proportionality.
- t-test requires that the sample conform to a normal distribution, and obviously we cannot guarantee that the accuracy on different data sets will conform to a normal distribution.
- The t-test has certain requirements for sample size, usually requiring a minimum of >30 samples. In this example we only have 14, and in most cases we don't have 30 data for the experiment.
- Statistical results are susceptible to outliers because of the lack of proportionality.

**Inappropriate Method 3: Sign test** is a non-parametric test that has the advantage that there is no requirement for sample distribution and no requirement for normality. The comparison method is simple: see which algorithm is better on each dataset, and then count the total number of datasets where each algorithm prevails. In this example, for example, C4.5 is optimal on 2 datasets, with 2 ties and 10 worst. If we calculate the confidence interval for this result, we find that $ (p<0.05) $  is required to be optimal on at least 11 datasets. Thus the disadvantages of this method include:

- The -sign test is a very weak test, losing a lot of information by comparing only superiority and inferiority, losing quantitative information (quantitative), e.g. 0.1 < 0.9 and 0.1 < 0.11 have the same meaning. For this reason, critical values need to be very large, such as in this example where the critical value of α = 0.05 is 11 (Figure 2).
- Another problem is that, in the absence of quantitative information, it is often difficult to determine whether the "win" is due to randomness. For example, does 0.99<0.991 really mean that Algorithm A is better? One way to look at it is that a threshold needs to be defined, and that a difference is only better if it is greater than the threshold. However, the problem with this view is that, assuming that Algorithm A has a "slight advantage" over Algorithm B on 1000 datasets, do we need to doubt significance? So again, **the fundamental issue is that the sign test requires a large sample size in order to be significant.**

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/11/05/BRqMFS.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Threshold table for the sign test</div>
</center>

## 4. Comparison of the two algorithms: recommended methods

Given the generality, we need to use non-parametric tests. In other words, we need to ensure that we make no assumptions about the distribution of the sample, which is more general.

**Method 1: The Wilcoxon Signed Ranks Test (WS)** is a non-parametric version of the paired t-test, which also analyses whether the difference between pairs of data is equal to O, but by ranking them. In other words, it can be interpreted as a quantitative version of the Signed Ranks Test. The advantages are as follows:

- No parameters, no requirement that the sample conform to a normal distribution.
- Consistent with data proportionality, although qualitative (vs. paired t-test).
- There are certain quantitative properties, i.e. larger differences have a greater impact on the final result (compared to a sign test).

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/11/05/BROu8g.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Accuracy and ranking of the two algorithms on 14 datasets</div>
</center>

**Method 2 (see section VII for details): The Mann Whitney U test (MW)**, like the WS, is a non-parametric and study ranking test, with the following characteristics:

- It can be used to detect samples of different sizes, for example, the performance of Algorithm A on 8 datasets vs the performance of Algorithm B on 10 datasets.
- No paired sex requirement, cf. previous point.
- The comparison is between two sample distributions, so errors in different datasets should conform to a particular distribution and may not satisfy proportionality
- The assumptions made about the measurement methods are: **independent measurement**, which is not what we actually have.

In other words, MW is recommended only reluctantly when the sample sizes are different, because it does not fit the assumption of independent measurement. Errors (accuracy) in different datasets do not necessarily fit a particular distribution and may well not be proportionate, but are useful in specific situations, as detailed in Section 7.

**Conclusion: If the sample is paired and conforms to a normal distribution, the paired t test is preferred. If the sample is not normally distributed, but matches the pairing, use WS.**

## 5. Comparison of multiple algorithms: inappropriate methods

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/11/05/BROTdP.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Accuracy and ordering of four algorithms on 14 datasets</div>
</center>

The above figure provides the accuracy of the four algorithms $ (C4.5, C4.5+m, C4.5+cf, C4.5+m+cf) $ on the 14 datasets.

**Inappropriate Method 1:** One view is whether we can extend the comparison of two algorithms to multiple algorithms. Suppose there are k algorithms, can we compare them two-by-two and compute them $ \frac{(1+(k-1)) \times (k-1)}{2} = \frac{k^2-k}{2} times to get a matrix. This is a classic multiple hypothesis testing problem, and this exhaustive approach generally assumes independence between the different comparisons - which is generally unrealistic and needs to be corrected, so I won't repeat it.

**Inappropriate method 2: Repeated measures ANOVA** is a classical statistical method used to make comparisons between multiple samples Yes, it can be seen as a multiple extension of the t-test. ANOVA is not suitable for comparing algorithm performance for the following reasons:

- There is an assumption of normality for the sample distribution, however the accuracy on different data sets often does not meet this assumption.
- Different samples have the same overall population variance.

Unfortunately, the performance of the algorithm we want to compare does not fit this case, so ANOVA is not suitable.

## 6. Comparison of algorithms: recommended methods

We need to find a way to solve the problems mentioned in part 5 at the same time, and this method requires:

- Non-parametric, does not make assumptions about the distribution of data.
- No need, or as little dependence as possible, or the ability to automatically correct for errors caused by comparing two to two

Demšar<sup>[1]</sup> recommends the non-parametric multivariate hypothesis test Friedman test, which is also a test based on rank, which assumes that all samples are ranked with equal mean values. Specifically, we first rank the different algorithms on each dataset and finally compute the mean value of the ranking of Algorithm A on all datasets. If there is no performance difference between all algorithms, then the mean ranking of their performance should be equal so that we can choose a specific confidence interval to determine whether the difference is significant or not.

Assuming that we find a statistically significant (p<0.05) with the Friedman test, we also need to continue to do post-hoc analysis (post-hoc). In other words, the Friedman test can only tell us if there is a significant difference between algorithms, but not exactly which algorithms have performance differences. To locate the specific differing algorithms, we also need to perform post-hoc analysis.

The post-hoc that generally accompanies the Friedman test is the Nemenyi test, which indicates whether there is a significant difference between the two. We also generally visualize the Nemenyi results, such as the figure below.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/11/05/BRzXbd.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;"> Nemenyi's comparison of 10 algorithms, with NS representing not significant</div>
</center>

Another point worth mentioning is that even if Friedman proves that the performance of the algorithms is significantly different, Nemenyi does not necessarily specify which algorithms are different, because Nemenyit is weaker than Friedman, and it is not possible to analyse pairs of algorithms that must be analysed.

**Method 2 (see section 7):** As with the two-by-two comparison, there are certain circumstances that prevent us from using Friedman-Nemenyi when comparing multiple samples. This method is characterized by:

- It can be used to test samples of different sizes, for example, Algorithm A's performance on 8 datasets vs. Algorithm B's performance on 10 datasets vs. Algorithm C's performance on 20 datasets.
- The assumption about the measurement method is that it is **independent**, which is not the case in our reality.

## 7. Repeated measurements and independent measurements.

In Part II, we analyze repeated versus independent measurements and assume that the comparison of machine learning performance should be **based on "repeated measurements", i.e., all algorithms are evaluated on the same dataset.**

Under this assumption, we recommend the parameter-free: Wilcoxon for comparing two algorithms and Friedman-Nemenyi for comparing multiple algorithms.

However, the assumption of **"repeated measurements" is not necessarily true**. For example, if we have only one data, and we sample from the data, we get a number of related test sets 1,2,3.... .n, and used to test different algorithms.

- Algorithm A: Test Set 1,2
- Algorithm B: Test Set 3,4,5,6
- Algorithm N...

In this case, we can compare two algorithms with the Mann Whitney U test and Kruskal-Dunn with multiple algorithms. **And it is worth noting that this is common with synthetic data, such as data sampled from a Gaussian distribution.** Therefore, it is important to specifically analyse how the data are measured and then decide how to evaluate them.

## 8. Tool library and implementation

We know that all of these tests are available on R. Let's focus on the libraries available on Python. Fortunately, all of the tests mentioned above are available in the Python tool library.

- Scipy [Statistical functions](https://docs.scipy.org/doc/scipy/reference/stats.html) : Wilcoxon，Friedman，Mann Whitney
- [scikit-posthocs](https://pypi.org/project/scikit-posthocs/): Nemenyi，Dunn's test

## Reference

[1] Demšar, J., 2006. Statistical comparisons of classifiers over multiple data sets. Journal of Machine learning research, 7(Jan), pp. 1-30.

Here the properties of the various algorithms are compared and tabulated as follows:

![B2Yx5n.jpg](https://s1.ax1x.com/2020/11/05/B2Yx5n.jpg)