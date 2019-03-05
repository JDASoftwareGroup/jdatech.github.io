---
layout: single
title: "PyCon.DE Part 3: Connecting Python to Big Data Landscapes using Arrow and Parquet"
date:   2018-03-12 20:39:02
tags: conference python
header:
  overlay_image: assets/images/dataStorm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Peter Hoffmann
author_profile: true
---

At last year´s [PyCon.DE](https://de.pycon.org/), from October 25th till 27th, Blue Yonder held several interesting talks around Python. In case you missed out the event, we would like to share the best of our talks at the conference. While Python itself hosts a wide range of machine learning and data tools, other ecosystems like the [Hadoop](http://hadoop.apache.org/) world also provide beneficial tools that can be either connected via Apache Parquet files or in memory using Apache Arrow. The following talk shows recent developments that allow interoperation at speed.

{% include video id="IvLScEcoO8" provider="youtube" %}

Python has a vast amount of libraries and tools in its machine learning and data analysis ecosystem. Although it is clearly in competition with R here about the leadership, the world that has sprung out of the Hadoop ecosystem has established itself in the space of data engineering and also tries to provide tools for distributed machine learning. As these stacks run in different environments and are mostly developed by distinct groups of people, using them jointly has been a pain. While Apache Parquet has already proven itself as the gold standard for the exchange of DataFrames serialized to files, Apache Arrow recently got traction as the in-memory format for DataFrame exchange between different ecosystems. 

This talk will outlines how [Apache Parquet](http://parquet.apache.org) files can be used in Python and how they are structured to provide efficient DataFrame exchange. In addition to small code sample, this also includes an explanation of some interesting details of the file format. Additionally, the idea of [Apache Arrow](https://arrow.apache.org) is presented and taking Apache Spark (2.3) as an example to showcase how performance increases once DataFrames can be efficiently shared between Python and JVM processes. 

Uwe Korn is a Data Scientist at Blue Yonder. His expertise is on building architectures for machine learning services that are scalable and usable for multiple customers aiming at high service availability as well as rapid prototyping of solutions to evaluate the feasibility of his design decisions. As part of his work providing an efficient data interchange, he became a core committer to the Apache Parquet and Apache Arrow projects.