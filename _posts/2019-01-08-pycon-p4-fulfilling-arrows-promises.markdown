---
layout: single
title: "PyCon.DE 2018 Part 4: Fulfilling Apache Arrow's Promises: Pandas on JVM memory without a copy"
date:   2019-01-08 12:51:28
tags: conference python
header:
  overlay_image: assets/images/dataStorm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Peter Hoffmann
author_profile: true
---

In Part 4 of the PyCon.DE 2018 series, we will discuss how to use [Pandas](https://pandas.pydata.org/), [Apache Arrow](https://arrow.apache.org/) and Python to reduce the overhead of working with columnar data between different systems and presenting the performance benefits in a typical data engineering use-case. 

PyCon.DE is where Pythonistas in Germany can meet to learn about new and upcoming Python libraries, tools, software, and data science. In 2018 we had more than 500 participants in Karlsruhe, a city where approximately 3600 IT companies with more than 36000 jobs exist.  

Since early 2016, Apache Arrow has redefined the standard for columnar in-memory analytics with implementations in Java, C++, Python and many more. To many Python/Pandas users, it is known for reading Apache Parquet files, although the main benefit is its cross-language interoperability, e.g. in Python and R/Java by using feather and PySpark via the filesystem or network. Even though they remove serialization overhead and improve data sharing the data is still being copied when passed between processes. 

Pandas introduced the concept of ExtensionArrays in version 0.23 which allows the extension of DataFrames and Series with custom, user-defined types. By using Apache Arrow, we can, on one hand, implement ExtensionArrays that are not tied to Pandas’ internal BlockManager and on the other hand, use a much wider set of efficient types that we can expose as ExtensionArrays. 

To show the real-world benefits of this, we present a typical setup nowadays, where a JVM based engine queries from a large data pool and passes it into a machine learning model, normally implemented in Python using popular frameworks like CatBoost or Tensorflow. These query engines sometimes provide Python clients, but they are not performant on large result sets which we have in machine learning models, where we feed as many rows as possible into the model. 

To overcome this bottleneck, the engines provide efficient JDBC drivers, but the conversion from Java objects to Python objects in the JVM bridge will slow things down again. However, we will demonstrate that by using Arrow to retrieve a large result set in the JVM and then pass it on to Python will not result in these bottlenecks. 

Uwe Korn is a Senior Data Scientist at the German RetailTec company Blue Yonder. His expertise is on building scalable architectures for machine learning services. Nowadays he focuses on the data engineering infrastructure that is needed to provide the building blocks to bring machine learning models into production. As part of his work to provide an efficient data interchange, he became a core committer to the Apache Parquet and Apache Arrow projects.