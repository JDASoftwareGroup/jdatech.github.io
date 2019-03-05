---
layout: single
title: "PyCon.DE 2018 Part 1: Scalable Scientific Computing using Dask"
date:   2019-01-08 12:32:08
tags: conference python
header:
  overlay_image: assets/images/data storm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Peter Hoffmann
author_profile: true
---

In Part 1 of the PyCon.DE 2018 series, we show how to avoid the limitation (only using the main memory of a single machine and most of the time also only a single CPU) of [Pandas](https://pandas.pydata.org/) and [NumPy](http://www.numpy.org/), by using [Dask](https://dask.org/). 

[PyCon.DE](https://pycon.de) is where Pythonistas in Germany can meet to learn about new and upcoming Python libraries, tools, software, and data science. In 2018 we had more than 500 participants in Karlsruhe, a city where approximately 3600 IT companies with more than 36000 jobs exist.  

Since Pandas and NumPy are two of the most used and best tools for data analysis and training machine learning models we want them to perform fast while using their intuitive APIs. Technically this should not be a problem, but both Pandas and NumPy are restricted to the main memory and most of the time also to a single CPU. To cross this boundary, we can use Dask to utilize multiple CPUs or even a cluster. 

Dask operates in parallel on data that does not need to fit into the main memory while using dynamic task schedulers to execute task graphs in parallel on the low level. For simulating NumPys lists and Pandas APIs, Dask uses high-level Array, Bag and DataFrame implementations, but the execution engines can also power custom, user-defined workloads. 

The workshop includes using `dask.array` and `dask.dataframe` to turn NumPy code into parallel/distributed code, showing examples which can be easily transferred and examples which are more complex, as well as a presentation of the tools Dask provides to inspect the execution graphs and the behavior of the distributed code. 

Uwe Korn is a Senior Data Scientist at the German RetailTec company Blue Yonder. His expertise is on building scalable architectures for machine learning services. Nowadays he focuses on the data engineering infrastructure that is needed to provide the building blocks to bring machine learning models into production. As part of his work to provide an efficient data interchange, he became a core committer to the Apache Parquet and Apache Arrow projects.