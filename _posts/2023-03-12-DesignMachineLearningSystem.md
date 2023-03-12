---

toc: true
layout: post
description: My note of Design Machine Learning System
categories: [theory, machine_learning, DMLS]
title: Design Machine Learning System
image: images/DMLS.png

---

# Chapter 1: Overview of Machine Learning Systems

1. ML is not a magic tool that can solve all problems.

2. When developing an ML project, it's critical to understand the requirements from all the stakeholders and how strict these requirements are.

3. Typically, the percentiles for latency that we should look at are p90, p95, and p99.

4. Fairness is a big concern when training models.

5. While it's important to pursue research, most companies cannot afford it unless it leads to short-term business applications.

6. The trend shows that applications developed with the most/best data win.

# Chapter 2: Introduction to Machine learning Systems Design

1. Most businesses don't care about ML metrics unless they can move business metrics. Thus, if an ML system is built for a business, it must be motivated by business objectives.



# Chapter 7: Model Deployment and Prediction Service

## Batch Prediction vs Online Prediction

There are three main modes of prediction:

- Batch prediction -  using only batch features

- Online prediction - using only batch features

- Online prediction - using both batch features and streaming features

> Batch features: features computed from historical data, such as data in databases and data warehouses.
> 
> Streaming features: Features computed from streaming data.

## Model Compression

The process makes the model smaller. This makes the models fit on edge devices. Making models smaller often makes them run faster.

## ML on the Cloud and on the Edge

Running on the cloud costs a lot of money, while on edge requires a lot of optimization on the models' sizes.

# Chapter 8: Data Distribution Shifts and Monitoring

## Causes of ML System Failures

### Software failures

* Dependency failure

* Deployment failure

* Hardware failures

* Downtime or crashing

### ML-specific failures

* Production data differing from training data
  
  * Training data and unseen data should come from a similar distribution. This assumption is incorrect in most cases.

* Edge cases

* Degenerate feedback loops
  
  * Having visibility into how the model makes predictions - such as measuring the importance of each feature for the model can help detect the bias toward feature X in this case.

## Data Distribution Shifts

Data distribution shift refers to the phenomenon in supervised learning when the data a model works with changes over time, which causes the model's predictions to become less accurate as time passes.

* Covariate shift

* Label shift

* Concept shift

#### Detecting Data Distribution Shifts

- Monitor the model's accuracy-related-metrics such as accuracy, F1 score, recall, AUC-ROC, etc.
  
  - Using mean, min, max, std, etc. Check if they are different by a threshold. This method is used by TFDV.
  
  - Using Hypothesis Testing. Let consider the data from yesterday to be the source population and the data from today to be the target population and they are statistically different, it's likely that the underlying data distribution has shifted between yesterday and today.

## Monitoring and Observability

## ML-specific Metrics

There are four artifacts to monitor:

* a model's accuracy-related-metrics
  
  * This helps to decide whether a model's performance has degraded.

* predictions
  
  * Prediction distribution shifts are also a proxy for input distribution shifts.

* features
  
  * Feature validation: Great Expectation

* raw inputs
  
  * It is not easy to monitor as it can com from multiple sources in different formats, following multiple structures.
  
  * This tasks are usually done by the data team, not ML team.

## Monitoring Toolbox

* Logs

* Dashboards

* Alerts

## Observability

It helps to understand how the entire ML system, which includes ML model, works. 
