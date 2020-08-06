---
layout: single
title:  "Introducing Cube Functionality To Kartothek"
date:   2020-07-21 06:00:00 +0500
tags: technology python data-engineering
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Usha Nemani & Marco Neumann
author_profile: true
---
# Introducing Cube Functionality To Kartothek

Last year, we introduced [Kartothek](../introducing-kartothek/), a table management python library powered by Dask. 
Our journey continued by adding another gem to our open source treasure: We empowered Kartothek with multi-dataset functionality.
 
You might think: "Kartothek already provides dataset features, do I really need multiple dataset interfaces?" Hang on,
 we will present you the story of **Kartothek cubes**, an interface that supports multiple datasets. 

Imagine a typical machine learning workflow, which might look like this:
* First, we get some input data, or source data. In the context of Kartothek cubes, we will refer to the source data as seed data or seed dataset.
* On this seed dataset, we might want to train a model that generates predictions.
* Based on these predicitons, we might want to generate reports and calculate KPIs.
* Last, but not least, we might want to create some dashboards showing plots of the aggregated KPIs as well as the underlying input data.
What we need for this workflow is not a table-like view on our data, but a view on everything that we generated in these different steps.
 
Kartothek Cubes deals with multiple Kartothek datasets, loosely modeled after [Data Cubes](https://en.wikipedia.org/wiki/Data_cube).

Cubes offer an interface to query all of the data without performing complex join operations manually each time. 

Because kartothek offers a view on our cube as a large virtual pandas DataFrame, querying the whole dataset is very comfortable.

## How to use Cubes?

Let us start with building the cube for **geodata**. Similar to Kartothek, 
we need a [simplekv](https://simplekv.readthedocs.io/)-based store backend along with an abstract cube definition.

**df_weather** is a pandas dataframe created from reading a csv file.

```python
>>> from io import StringIO
>>> import pandas as pd
>>> df_weather = pd.read_csv(
...     filepath_or_buffer=StringIO("""
... avg_temp     city country        day
...        6  Hamburg      DE 2020-07-01
...        5  Hamburg      DE 2020-07-02
...        8  Dresden      DE 2020-07-01
...        4  Dresden      DE 2020-07-02
...        6   London      UK 2020-07-01
...        8   London      UK 2020-07-02
...     """.strip()),
...     delim_whitespace=True,
...     parse_dates=["day"],
... )
```

Now, we want to store this dataframe using the cube interface.
 
To achieve this, we have to specify the cube object first by providing some meta-information about our data.
The `uuid_prefix` serves as identifier for our dataset. The `dimension_columns` are the dataset's primary keys, so all rows within this datset have to be unique with respect to the `dimension_columns`.
The `partition_columns` specify the columns which are used to physically partition the dataset.

```python
>>>#we are creating a geodata cube instance
>>> from kartothek.core.cube.cube import Cube
>>> cube = Cube(
...     uuid_prefix="geodata",
...     dimension_columns=["city", "day"],
...     partition_columns=["country"]
...)
```

We use the simple **kartothek.io.eager_cube** backend to store the data:

```python
>>> from kartothek.io.eager_cube import build_cube
>>> datasets_build = build_cube(
...   data=df_weather,
...   store=store,
...   cube=cube
...)
```


where **store** is the **simplekv** store of storefactory. (For more details, please refer our [Kartothek](2019-05-28-introducing-kartothek.markdown) post.)

We have just preserved a single Kartothek dataset. Let's print the content of seed dataset.

```python
>>> print(", ".join(sorted(datasets_build.keys())))
seed
>>> ds_seed = datasets_build["seed"].load_all_indices(store)
>>> print(ds_seed.uuid)
geodata++seed
>>> print(", ".join(sorted(ds_seed.indices)))
city, country, day
```

Now we have a quick look at the store content. Note that we cut out UUIDs and timestamps here for brevity:

```python
>>> import re
>>> def print_filetree(s, prefix=""):
...     entries = []
...     for k in sorted(s.keys(prefix)):
...         k = re.sub("[a-z0-9]{32}", "<uuid>", k)
...         k = re.sub("[0-9]{4}-[0-9]{2}-[0-9]{2}((%20)|(T))[0-9]{2}%3A[0-9]{2}%3A[0-9]+.[0-9]{6}", "<ts>", k)
...         entries.append(k)
...     print("\n".join(sorted(entries)))
>>> print_filetree(store)
geodata++seed.by-dataset-metadata.json
geodata++seed/indices/city/<ts>.by-dataset-index.parquet
geodata++seed/indices/day/<ts>.by-dataset-index.parquet
geodata++seed/table/_common_metadata
geodata++seed/table/country=DE/<uuid>.parquet
geodata++seed/table/country=UK/<uuid>.parquet
```

We can **extend** this cube by adding new columns to the dataframes.

## Extend Operation

Now let’s say we also would like to have longitude and latitude data in our cube.

```python
>>> from kartothek.io.eager_cube import extend_cube
>>> df_location = pd.read_csv(
...     filepath_or_buffer=StringIO("""
...    city country  latitude  longitude
... Hamburg      DE 53.551086   9.993682
... Dresden      DE 51.050407  13.737262
...  London      UK 51.509865  -0.118092
...   Tokyo      JP 35.652832 139.839478
...     """.strip()),
...     delim_whitespace=True,
... )
```

```python
>>> datasets_extend = extend_cube(
...   data={"latlong": df_location},
...   store=store,
...   cube=cube,
... )
```

This results in an extra dataset:

```python
>>> print(", ".join(sorted(datasets_extend.keys())))
latlong
>>> ds_latlong = datasets_extend["latlong"].load_all_indices(store)
>>> print(ds_latlong.uuid)
geodata++latlong
>>> print(", ".join(sorted(ds_latlong.indices)))
country
```
Note that for the second dataset, no indices for **city** and **day** exist. 
These are only created for the seed dataset, since that dataset forms the groundtruth about which `city-day` entries are part of the cube.

If you look at the file tree, you can see that the second dataset is completely separated. This is useful to copy/backup parts of the cube.

```python
>>> print_filetree(store)
geodata++latlong.by-dataset-metadata.json
geodata++latlong/table/_common_metadata
geodata++latlong/table/country=DE/<uuid>.parquet
geodata++latlong/table/country=JP/<uuid>.parquet
geodata++latlong/table/country=UK/<uuid>.parquet
geodata++seed.by-dataset-metadata.json
geodata++seed/indices/city/<ts>.by-dataset-index.parquet
geodata++seed/indices/day/<ts>.by-dataset-index.parquet
geodata++seed/table/_common_metadata
geodata++seed/table/country=DE/<uuid>.parquet
geodata++seed/table/country=UK/<uuid>.parquet
```
## Querying Cubes

The seed dataset presents the groundtruth regarding rows, all other datasets are joined via a left join. 

 
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

 
## Transform

Query and extend operations can be combined to build powerful transformation pipelines. To better illustrate this we will use **dask.bag_cube** for the example.

```python
>>> from kartothek.io.dask.bag_cube import (
...     extend_cube_from_bag,
...     query_cube_bag,
... )
>>> def transform(df):
...     df["avg_temp_country_min"] = df["avg_temp"].min()
...     return {
...         "transformed": df.loc[
...             :,
...             [
...                 "avg_temp_country_min",
...                 "city",
...                 "country",
...                 "day",
...             ]
...         ],
...     }
>>> transformed = query_cube_bag(
...     cube=cube,
...     store=store_factory,
...     partition_by="day",
... ).map(transform)
>>> datasets_transformed = extend_cube_from_bag(
...     data=transformed,
...     store=store_factory,
...     cube=cube,
...     ktk_cube_dataset_ids=["transformed"],
... ).compute()
>>> query_cube(
...     cube=cube,
...     store=store,
...     payload_columns=[
...         "avg_temp",
...         "avg_temp_country_min",
...     ],
... )[0]
   avg_temp  avg_temp_country_min     city country        day
0         8                     6  Dresden      DE 2020-07-01
1         4                     4  Dresden      DE 2020-07-02
2         6                     6  Hamburg      DE 2020-07-01
3         5                     4  Hamburg      DE 2020-07-02
4         6                     6   London      UK 2020-07-01
5         8                     4   London      UK 2020-07-02
```
Notice that the `partition_by` argument does not have to match the cube `partition_columns` to work. 
You may use any indexed column. Keep in mind that fine-grained partitioning can have drawbacks though, 
namely large scheduling overhead and many blob files which can make reading the data inefficient.

```python
>>> print_filetree(store, "geodata++transformed")
geodata++transformed.by-dataset-metadata.json
geodata++transformed/table/_common_metadata
geodata++transformed/table/country=DE/<uuid>.parquet
geodata++transformed/table/country=DE/<uuid>.parquet
geodata++transformed/table/country=UK/<uuid>.parquet
geodata++transformed/table/country=UK/<uuid>.parquet
```

## Append
New rows can be added to the cube using an append operation.

```python
>>> from kartothek.io.eager_cube import append_to_cube
>>> df_weather2 = pd.read_csv(
...     filepath_or_buffer=StringIO("""
... avg_temp     city country        day
...       20 Santiago      CL 2020-07-01
...       22 Santiago      CL 2020-07-02
...     """.strip()),
...     delim_whitespace=True,
...     parse_dates=["day"],
... )
>>> datasets_appended = append_to_cube(
...   data=df_weather2,
...   store=store,
...   cube=cube,
... )
>>> print_filetree(store, "geodata++seed")
geodata++seed.by-dataset-metadata.json
geodata++seed/indices/city/<ts>.by-dataset-index.parquet
geodata++seed/indices/city/<ts>.by-dataset-index.parquet
geodata++seed/indices/day/<ts>.by-dataset-index.parquet
geodata++seed/indices/day/<ts>.by-dataset-index.parquet
geodata++seed/table/_common_metadata
geodata++seed/table/country=CL/<uuid>.parquet
geodata++seed/table/country=DE/<uuid>.parquet
geodata++seed/table/country=UK/<uuid>.parquet
```
Notice that the indices where updated automatically.

```python
>>> query_cube(
...     cube=cube,
...     store=store,
... )[0]
   avg_temp  avg_temp_country_min      city country        day   latitude  longitude
0         8                   6.0   Dresden      DE 2020-07-01  51.050407  13.737262
1         4                   4.0   Dresden      DE 2020-07-02  51.050407  13.737262
2         6                   6.0   Hamburg      DE 2020-07-01  53.551086   9.993682
3         5                   4.0   Hamburg      DE 2020-07-02  53.551086   9.993682
4         6                   6.0    London      UK 2020-07-01  51.509865  -0.118092
5         8                   4.0    London      UK 2020-07-02  51.509865  -0.118092
6        20                   NaN  Santiago      CL 2020-07-01        NaN        NaN
7        22                   NaN  Santiago      CL 2020-07-02        NaN        NaN
 
```

## Remove and Delete Operations

You can **remove** entire partitions from the cube using the remove operation.

```python
>>> from kartothek.io.eager_cube import remove_partitions
>>> datasets_after_removal = remove_partitions(
...     cube=cube,
...     store=store,
...     ktk_cube_dataset_ids=["latlong"],
...     conditions=(C("country") == "UK"),
... )
>>> query_cube(
...     cube=cube,
...     store=store,
... )[0]
   avg_temp  avg_temp_country_min      city country        day   latitude  longitude
0         8                   6.0   Dresden      DE 2020-07-01  51.050407  13.737262
1         4                   4.0   Dresden      DE 2020-07-02  51.050407  13.737262
2         6                   6.0   Hamburg      DE 2020-07-01  53.551086   9.993682
3         5                   4.0   Hamburg      DE 2020-07-02  53.551086   9.993682
4         6                   6.0    London      UK 2020-07-01        NaN        NaN
5         8                   4.0    London      UK 2020-07-02        NaN        NaN
6        20                   NaN  Santiago      CL 2020-07-01        NaN        NaN
7        22                   NaN  Santiago      CL 2020-07-02        NaN        NaN 
```

You can also **delete** entire datasets (or the entire cube).

```python
>>> from kartothek.io.eager_cube import delete_cube
>>> datasets_still_in_cube = delete_cube(
...     cube=cube,
...     store=store,
...     datasets=["transformed"],
... )
>>> query_cube(
...     cube=cube,
...     store=store,
... )[0]
   avg_temp      city country        day   latitude  longitude
0         8   Dresden      DE 2020-07-01  51.050407  13.737262
1         4   Dresden      DE 2020-07-02  51.050407  13.737262
2         6   Hamburg      DE 2020-07-01  53.551086   9.993682
3         5   Hamburg      DE 2020-07-02  53.551086   9.993682
4         6    London      UK 2020-07-01        NaN        NaN
5         8    London      UK 2020-07-02        NaN        NaN
6        20  Santiago      CL 2020-07-01        NaN        NaN
7        22  Santiago      CL 2020-07-02        NaN        NaN
```

## Cube Features in Kartothek

*	**Multiple-datasets**: When mapping multiple parts (tables or datasets) to Kartothek, using multiple datasets allow users to copy, backup and delete them separately.
 Index structures are bound to datasets. This was not possible with the existing multi-table (within a single dataset) feature present in kartothek.
 We intend to phase out the multi-table single dataset functionality soon.

*	**Seed-Based Join System / Partition-alignment**: When data is stored in multiple parts (tables or datasets), the question is how to expose it to the user during read operations.
 Seed-based join marks a single part as seed which provides seed dataset in the cube, all other parts are just additional columns.
 Cube uses lazy approach of seed based join, since it better supports independent copies and backups of datasets and also simplifies some of our processing pipelines
 (e.g. geolocation data can blindly be fetched for too many locations and dates.)	


## Outlook
In the upcoming months we’ll continue to expand the Kartothek functionality. Here are a few highlights of what's next:

- **API cleanup:** The API surface of kartothek grew organically over the years and we plan to re-design it. 
While doing so, we will incorporate our learnings regarding API design and will also prune some features that are not needed anymore or that did not match their expectations (e.g. the original multi-table design).
- **Ecosystem integration:** At this point in time, there are multiple dataset formats (e.g. [Apache Arrow](https://arrow.apache.org/docs/python/dataset.html), 
[Apache Iceberg](https://iceberg.apache.org/), [Delta Lake](https://delta.io/)) and we will investigate how to evolve kartothek as a library and as a format to align better with the ecosystem and enable new features (like schema migrations and time travel), 
while providing the stability and safety that our users require.
- **Query Planning:** Currently the kartothek query planner solely relies on file-level information (file names for primary indices and separate index files for secondary indices). 
It would be great to also use the RowGroup-level statistics as specified in [Apache Parquet](https://github.com/apache/parquet-format) to improve query performance.
We will have a look at [the work Dask already did](https://github.com/dask/dask/blob/55445565c3746f97f2bc50d5628a484576cef90e/dask/dataframe/io/parquet/core.py) in this area.
