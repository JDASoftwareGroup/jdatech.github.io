---
layout: single
title: "PyCon.DE 2018 Part 7: Strongly typed datasets in a weakly typed world"
date:   2019-01-08 12:54:28
tags: conference python
header:
  overlay_image: assets/images/data storm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Peter Hoffmann
author_profile: true
---

In Part 7 of our PyCon.DE 2018 series, we will have a look at the issues with exchanging data in a Pandas-driven environment, where types are rather unstable, using strongly typed Parquet Datasets / Hive Tables. In the end, there will be an RFC directed to the community. 

PyCon.DE is where Pythonistas in Germany can meet to learn about new and upcoming Python libraries, tools, software, and data science. In 2018 we had more than 500 participants in Karlsruhe, a city where approximately 3600 IT companies with more than 36000 jobs exist.  

Here at Blue Yonder, we want to have a feature-rich interface, flexibility and access to a large community and ecosystem for our daily data science and engineering work. Because of this, we use [Pandas](https://pandas.pydata.org/) together with Python as an underlying programming language quite a lot. 

Issues occur when we want to preserve and exchange data with different software stacks using [Parquet Datasets](https://parquet.apache.org/) / [Hive Tables](https://hive.apache.org/). Because parquet files and the schema information stored alongside them dictate very precise types, conflicts can happen when coming from a weakly typed world. For example, Pandas may convert integers to floats regularly without asking. This can get even more complex when datasets are written by multiple code versions or different software solutions over time, which will result in important questions regarding type compatibility. 

In the talk, we introduce the types used at the different layers (like NumPy, Pandas, Arrow and Parquet) and the transition between them, and after that, we present some examples of type compatibility which we have seen together with why and how we think they should be handled. It concludes with a Q&A which can be seen as the start of a potentially longer RFC process to align different software stacks (like Hive and Dask) to handle types in a similar way. 

Marco Neumann studied computer science at KIT (Karlsruhe, Germany), worked as a Tech Student at CERN, and is now a Data Scientist at Blue Yonder (Hamburg, Germany). He loves to travel and to exchange all kinds of ideas.