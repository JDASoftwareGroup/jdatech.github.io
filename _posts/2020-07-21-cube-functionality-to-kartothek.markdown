---
layout: single
title:  "Introducing Cube Functionality To Kartothek"
date:   2020-07-21 06:00:00 +0500
tags: technology python data-engineering
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Marco Neumann & Usha Nemani
author_profile: true
---
# Introducing Cube Functionality To Kartothek

In our last blog we introduced [Kartothek](2019-05-28-introducing-kartothek.markdown), a table management python library powered by Dask. 
Our journey continued by adding a gem to our jewel. We empowered Kartothek with cube functionality. 
Kartothek already provides Dataset features, As a user do I really need a cube? Hang On!! 
Let us spend rest of our time understanding the story of “Cube”. 

## What is a Cube?
A Cube deals with multiple Kartothek datasets.

TODO: Why Cube functionality is developed?????

![Cube Image](/assets/images/2020-07-21-kartothek-cube.png)

Let us start with building a cube for **geodata**. Similar to Kartothek, 
we need a simplekv-based store backend along with an abstract cube definition.

```python
>>> from kartothek.core.cube.cube import Cube
>>> cube = Cube(
...     uuid_prefix="geodata",
...     dimension_columns=["city", "day"],
...     partition_columns=["country"]
...)
```

```python
>>> from kartothek.io.eager_cube import build_cube
>>> datasets_build = build_cube(
...   data=df_weather,
...   store=store,
...   cube=cube
...)
```

where **df_weather** is a pandas dataframe created from reading a csv file 
and **store** is the **simplekv** store of storefactory. (For more details, please refer our [Kartothek](2019-05-28-introducing-kartothek.markdown) blog.)


Similarly, we can **Extend** this cube by **Adding** new columns to the dataframes,  
**Append** new rows to the existing cube,  **Remove** partitions from cube,  **Delete** the entire cube,  
**Transform** the cube using **query** and **extend** operations.


The whole beauty of Cube does not come from storing multiple datasets,
but especially from retrieving the data  (**Querying**)  in a very comfortable way. 
Kartothek views the whole cube as a large, virtual DataFrame.
Dataset that provides the groundtruth about which Cells are in a Cube is called the **seed dataset**.
The seed dataset presents the groundtruth regarding rows, all other datasets are joined via a left join. 
Cube naturally supports **partition-by** semantic, which is more helpful for distributed backends.

## Query System
This section explain some technical details around this mechanism.

*  **Per Dataset Partitions**: First of all, all partition files for all datasets are gathered. Every partition file is represented by a unique label. 
For every dataset, index data for the Primary Indices (i.e partition columns) will be loaded and joined with the labels.  Also, pre-conditions are applied during that step. 
These are the conditions that can be evaluated based on index data (Partition Indices, Explicit Secondary Indices for dimension columns as well as index columns)
*  Now,  partition-by data is added (if not already present).  Finally, rows with identical partition information (physical and partition-by) are compactified.
 
*  **Alignment** : After data is prepared for every dataset, they are aligned using their physical partitions. 
Partitions that are present in non-seed datasets but are missing from the seed dataset are dropped.
In case pre-conditions got applied to any non-seed dataset or partition-by columns that are neither a Partition Column nor Dimension Column, 
the resulting join will be an inner join. This may result in removing potential partitions early.

*  **Re-Grouping** :  Now, the DataFrame is grouped by **partition-by**.

*  **Intra-Partition Joins** :
A simple explanation of the join logic would be: “The coordinates (cube cells) are taken from the seed dataset, all other information is add via a left join.”
Because the user is able to add conditions to the query and because we want to utilize predicate pushdown in a very efficient way,
we define another term: restricted dataset. These are datasets which contain non-Dimension Column and non-Partition Column to 
which users wishes to apply restrictions (via conditions or via partition-by). 
Because these restrictions always need to apply, we can evaluate them pre-join and execute an inner join with the seed dataset.
 
 
 ```python
>>> from kartothek.io.eager_cube import query_cube
>>> dfs = query_cube(
...     cube=cube,
...     store=store,
...     partition_by="country",
... )
>>> dfs[0]
   avg_temp     city country        day   latitude  longitude
0         8  Dresden      DE 2020-07-01  51.050407  13.737262
1         4  Dresden      DE 2020-07-02  51.050407  13.737262
2         6  Hamburg      DE 2020-07-01  53.551086   9.993682
3         5  Hamburg      DE 2020-07-02  53.551086   9.993682
>>> dfs[1]
   avg_temp    city country        day   latitude  longitude
0         6  London      UK 2020-07-01  51.509865  -0.118092
1         8  London      UK 2020-07-02  51.509865  -0.118092
```

The query system also supports selection and projection:

```python
>>> from kartothek.core.cube.conditions import C
>>> from kartothek.io.eager_cube import query_cube
>>> query_cube(
...     cube=cube,
...     store=store,
...     payload_columns=["avg_temp"],
...     conditions=(
...         (C("country") == "DE") &
...         C("latitude").in_interval(50., 52.) &
...         C("longitude").in_interval(13., 14.)
...     ),
... )[0]
   avg_temp     city country        day
0         8  Dresden      DE 2020-07-01
1         4  Dresden      DE 2020-07-02
```


## Additional Features of Cube

*	**Multi-dataset with Single Table**: When mapping multiple parts (tables or datasets) to Kartothek, using multiple datasets allows users to copy, backup and delete them separately. 
Index structures are bound to datasets which feels more consistent.
*	**Explicit physical Partitions**: We have decided for explicit physical partitions since we have seen that this data model works well for our current data flow. 
It allows quick and efficient re-partitioning to allow row-based, group-based, and timeseries-based data processing steps, while keeping the technical complexity rather low (compared to an automatic + dynamic partitioning).
It also maps well to multiple backends we planned to use.
*	**Update Granularity Partition-wise**: Entire partitions can overwrite old physical partitions. Deletion operations are partition-wise.
*	**Seed-Based Join System / Partition-alignment**: When data is stored in multiple parts (tables or datasets), the question is how to expose it to the user during read operations.
 Seed based Join marks a single part as seed which provides seed dataset in the cube, all other parts are just additional columns.
 Cube uses lazy approach of seed based join, 
 since it better supports independent copies and backups of datasets and also simplifies some of our processing pipelines (e.g. geolocation data can blindly be fetched for too many locations and dates.)	

## Dask And Yamal Support

The following features are supported by different backends:


| Feature | Eager | Dask.Bag | Dask.DataFrame |	Yamal |
| :-------: | :-----: | :--------: | :--------------: | :-----: |
| Build	  |    yes    |    yes     |      yes         |   yes   |
| Extend  |	 yes  |	  yes	 |     yes        |	 yes  |
| Remove  |  yes  |   no	 |     no         |   no  |
| Append  |  yes  |   yes	 |     yes        |	 yes  |
| Delete  |  yes  |   yes    |     no         |  yes  |
| Copy	  |  yes  |   yes    |	   no         |	 yes  |
| Query	  |  yes  |   yes    |	   yes        |  yes  |
| Stats	  |  yes  |   yes	 |     no	      |  yes  |
| Cleanup |	 yes  |   yes    |	   no         |  yes  |
| ------- | ----- | -------- | -------------- | ----- |


## New Features to Kartothek :
Kartothek features a **command line interface (CLI)** for some cube operations. 
To use it, create a  **skv.yml** file that describes storefact stores and use commands to gather information of the required cube.
Here we use geodata cube to get some information.

```python
>>>kartothek_cube geodata info  (gives geodata cube info)
>>>kartothek_cube geodata stats  (for cube scan)
>>>kartothek_cube --help  (To get list of  ktk_cube commands)
```

## Outlook:
TODO:What features are coming up next ??
