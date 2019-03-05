---
layout: single
title: "Blue Yonder at Apache Big Data Europe"
date:   2016-11-16 16:07:03
tags: conference
header:
  overlay_image: assets/images/2016-11-16-apache_big_data.png
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Manuel Baehr
---

At [Blue Yonder](https://www.blue-yonder.com/en), we make use of several Apache projects and actively contribute to three projects at the moment (Apache Aurora, Apache Arrow, and Apache Parquet) as project management committee (PMC) members and committers. The Apache Big Data Europe conference in Seville from November 14-16, 2016 is therefore the perfect place to present some of our learnings. Here we wanted to summarize the talks our engineers prepared for this conference. 

### Apache Parquet

[Uwe Korn](https://twitter.com/xhochy) presents  the origins and inner workings of the [Apache Parquet](http://parquet.apache.org/) format in [his talk](http://events.linuxfoundation.org/sites/events/files/slides/ApacheCon%20BigData%20Europe%202016%20-%20Parquet%20in%20Practice%20%26%20Detail_0.pdf). He puts together the reasons why you should use it as a preferred file format when doing analytics on transactional data, which is basically, **performance and interoperability**. 

  * For structured transactional data, the columnar representation of the data allows both efficient access as well as vectorized operations on its columns. Furthermore, compression and encoding of the raw data improves runtime and memory footprint of the workloads operating on this data. The ability to bring computations close to the I/O layer via query push-down is another way to improve performance.
  * Apache Parquet is a standard file format in the [Hadoop](http://hadoop.apache.org/) ecosystem. With a new [C++ implementation](https://github.com/apache/parquet-cpp) and the [Apache Arrow](https://arrow.apache.org/) project, it is now easily usable from Python and C++. This makes Apache Parquet an attractive choice on the most wide-spread platforms for doing data science, analytics and machine learning at large scale.

### Apache Airflow

[Christian Trebing](https://twitter.com/ctrebing) shows you in [his talk](http://events.linuxfoundation.org/sites/events/files/slides/get_in_control_of_your_workflow.pdf) why [Apache Airflow (incubating)](https://airflow.incubator.apache.org/) is such a breeze when it comes to managing multiple, heterogeneous workflows. Apache Airflow is a great way to **scale your data operations **by offering: 

  * A UI with runtime overview, Gantt charts, current state and a high-level log view
  * Extensibility using Python via plugins, operators and sensors
  * Flexible scheduling and the ability to react on errors between workflow steps
  * Advanced state management to handle exclusive resource reservation and concurrency
  * Lightweight deployment model with stateless python processes and a database for persisting state

### Apache Aurora

[Stephan Erb](https://twitter.com/ErbStephan) shows how you can leverage [Apache Aurora](http://aurora.apache.org/) as well as [Apache Mesos](http://mesos.apache.org/) to effectively implement a **DevOps approach for a large data science organization** [in his talk](http://events.linuxfoundation.org/sites/events/files/slides/presentation_5.pdf) . The following types of features enable such an endeavor. 

  * Multi-tenancy features baked into Apache Mesos and Apache Aurora, which make it possible to move as many workloads as possible onto a centrally operated cluster. On the lowest level, these are isolation features that ensure individual workloads do not affect each other in negative ways. Further up the stack, concepts like job environments (devel, staging, prod) and job tiers (preferred, revocable) enable a fine-grained operational model. Via multi-instance deployment, these features can be transferred to single-tenant systems and thus drive the adoption rate to shared infrastructure even further.
  * Operator features like preemption and resource quotas, maintenance primitives as well as resource oversubscription help you lower infrastructure costs while keeping the SLAs up.
  * Advanced user features, like rolling job updates with automatic rollback on failure, Docker/AppC image support, or service announcement, that enable the implementation of a continuous delivery model.

If you like the content of our talks or have further questions, please do not hesitate to contact us via [Twitter](https://twitter.com/BlueYonderTech). We are always eager to learn and improve.