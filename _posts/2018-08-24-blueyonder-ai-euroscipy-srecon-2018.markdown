---
layout: single
title: "Meet blueyonder.ai @ EuroSciPy & SRECon 2018"
date:   2018-08-24 08:57:54
tags: conference python
header:
  overlay_image: /assets/images/2018-08-24_meet_by.png
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Peter Hoffmann
author_profile: true
---

A group of Blue Yonder data scientists and engineers will be at the [EuroSciPy 2018](https://www.euroscipy.org/2018/)  in Trento, Italy from August 28th to September 1st as well as at the [SRECon18](https://www.usenix.org/conference/srecon18europe) from August 29th to 31st in Düsseldorf, Germany. EuroSciPy is the largest European conference for the Python langue in scientific research. We are going to present 3 talks about the daily usage of Python to build AI based supply chain solutions, which enable retailers to respond quicker to changing market conditions and customer dynamics, boosting revenues and increasing margins. SRECon is a gathering of engineers who care deeply about site reliability, systems engineering, and working with complex distributed systems at scale. Don´t miss the Blue Yonder talks: 

## Keynote [Felix Wick](https://twitter.com/WickFelix): How to drive business decisions with AI

On Thursday, [August 30 at 9:15 at EuroSciPy](https://www.euroscipy.org/2018/descriptions/k1.html) Bringing AI technology into core business processes is one of the key challenges for its large-scale industry adoption. When looking for possible applications of automated decision-making based on machine learning, the retail industry is an ideal example. It is characterized by fine margins and retailers face increasing customer expectations as well as extreme pressure from innovative competitors. At the same time, the established players lag in tech adoption. Two important subjects in the retail business that benefit considerably from AI applications are replenishment and price optimization. These deal with the questions of optimal order quantities and price changes. For replenishment, optimal can mean few out-of-stock situations or little waste of perishable products. For pricing, it can be maximization of revenue or profit via the price elasticity of demand. 

## [Patrick Muehlbauer](https://twitter.com/tmuxbee): Data visualizations for the web with Altair and Vega(-Lite)

On Thursday, [August 30 at 14:00 at EuroSciPy](https://www.euroscipy.org/2018/descriptions/Data%20visualizations%20for%20the%20web%20with%20Altair%20and%20Vega\(-Lite\).html) Visualizations help analyzing and reason about data and are often used by us to understand the effects of our own models. But often they are not only for us. There are also visualizations that have to be provided to our customers to support them with their daily business. There are lots of great tools for building data science dashboards in the Python data visualization landscape. Most of them make it very simple for data scientists but when the dashboards have to be part of a customer facing product, other roles like frontend developers get involved. Tools that are a pleasure for one role might be a burden for the other. We will show how Altair and Vega help improving the collaboration between data scientists and frontend developers. 

## [Peter Hoffmann](https://twitter.com/peterhoffmann): Apache Parquet as a columnar storage for large datasets

On Thursday, [August 30 at 14:30 at EuroSciPy](https://www.euroscipy.org/2018/descriptions/Apache%20Parquet%20as%20a%20columnar%20storage%20for%20large%20datasets.html) Apache Parquet is a binary, efficient columnar data format. It uses various techniques to store data in a CPU and I/O efficient way like row groups, compression for pages in column chunks or dictionary encoding for columns. Index hints and statistics to quickly skip over chunks of irrelevant data enable efficient queries on large amount of data. Apache Parquet files can be read into Pandas DataFrames with the two libraries fastparquet and Apache Arrow. While Pandas is mostly used to work with data that fits into memory, Apache Dask allows us to work with data larger then memory and even larger than local disk space. Data can be split up into partitions and stored in cloud object storage systems like Amazon S3 or Azure Storage. Using Metadata from the partiton filenames, parquet column statistics and dictonary filtering allows faster performance for selective queries without reading all data. This talk will show how use partitioning, row group skipping and general data layout to speed up queries on large amount of data. 

## [Stephan Erb](https://twitter.com/ErbStephan):  SLA-driven container updates and host maintenance in Apache Aurora

On Wednesday, [29 August, 2018 at SRECon](https://www.usenix.org/conference/srecon18europe/program) Measuring service levels generates insights. Insights are crucial, but automated decisions beat insights any time. We illustrate this point using the Apache Aurora container orchestrator. Knowing the uptime of a service is great, but the ability to derive decisions from user-de ned service levels is even better, e.g. to perform updates and maintenance as fast as possible - but not faster.