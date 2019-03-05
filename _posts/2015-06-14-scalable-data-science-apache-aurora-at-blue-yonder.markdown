---
layout: single
title:  "Scalable Data Science: Apache Aurora at Blue Yonder"
date:   2015-06-14 12:09:38 +0100
tags: technology
header:
  overlay_image: assets/images/dataStorm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Stephan Erb
---

As a cloud-based provider, we at Blue Yonder care about the full lifecycle of predictive applications. It is not enough to build, train and run machine learning models once. We have to continuously operate and maintain them in order to keep pace with the evolving businesses of the customers we are empowering. 

From a **data science perspective**, maintenance work includes monitoring data and prediction quality, A/B testing of model improvements and ad hoc queries into data and predictions. While our data scientists care about statistics, we want to free them of being concerned with technicalities like maintenance of network equipment, operating system updates or even hardware failures. In order to save our data scientists from these tasks, we invest into a data science platform. 

The platform is built with a private compute cloud, which allows our data scientists to run multiple machine learning projects on the same physical cluster. It greatly improves our hardware utilization. 

The front end of our compute cloud consists of a domain-specific UI and REST interface. We use  [Apache Aurora](https://aurora.apache.org/), a proven and battle-tested service scheduler for the [Apache Mesos](https://mesos.apache.org/) cluster manager, as the base for the back end. Aurora excels at starting processes on a cluster and keeping them alive even in presence of hardware and software failures. 

  * By using Apache Aurora we gain a clear separation of concerns: Data scientists specify what they want to run using the UI or a set of [Ansible roles](http://docs.ansible.com/)
  * Our middleware translates this into a specification of how it will be run (e.g. number of instances, destination for log files, etc.)
  * Aurora then manages the placement and execution on the cluster, including the supervision of the spawned processes.

We use Aurora to run cron jobs, Python-based microservices and our distributed in-house batch processing framework for machine learning. We spawn the framework as two kinds of Aurora jobs: One for the master and one for the workers. 

When multiple batch processing frameworks are competing for resources, we instrument Aurora’s job updater to dynamically adjust the number of worker instances per framework according to load, fairness and other SLA criteria. 

For our data scientists, this elasticity culminates in faster iteration cycles for their experiments, as they can profit from the aggregate power of the shared compute infrastructure. For us operators, adopting Apache Aurora as a replacement for a custom cluster scheduler helped to **improve the resilience and maintainability** of our platform. When we want to update the kernel of a host, we can smoothly drain all tasks it’s running without any impact on our users. Furthermore, by using a mediator in front of Aurora, we gain the consistency and standardization required for seamless operations. This means we can know exactly which software versions are currently running on the cluster, which is a crucial ingredient of our compliance and security standard.