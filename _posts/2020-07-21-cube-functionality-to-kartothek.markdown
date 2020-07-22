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
Our journey continued by adding a gem to our jewel. We empowered Kartothek with multiple-dataset functionality. 
Kartothek already provides Dataset features, As a user do I really need a multiple-dataset? Hang On!! 
Let us spend rest of our time understanding the story of **Cube (kartothek that supports multiple datasets)**. 

## What is a Cube?
A Cube deals with multiple Kartothek datasets.

Let us start with building a cube for **geodata**. Similar to Kartothek, 
we need a [simplekv](https://simplekv.readthedocs.io/)-based store backend along with an abstract cube definition.

**df_weather** is a pandas dataframe created from reading a csv file.

```python
>>> from io import StringIO
>>> df_weather = pd.read_csv(
...     filepath_or_buffer=StringIO("""
... avg_temp     city country        day
...        6  Hamburg      DE 2018-01-01
...        5  Hamburg      DE 2018-01-02
...        8  Dresden      DE 2018-01-01
...        4  Dresden      DE 2018-01-02
...        6   London      UK 2018-01-01
...        8   London      UK 2018-01-02
...     """.strip()),
...     delim_whitespace=True,
...     parse_dates=["day"],
... )
```


```python
>>> from kartothek.core.cube.cube import Cube
>>>##we are creating a geodata cube instance
>>> cube = Cube(
...     uuid_prefix="geodata",
...     dimension_columns=["city", "day"],
...     partition_columns=["country"]
...)
```

We use the simple **klee2.io.eager_cube** backend to store the data:

```python
>>> from kartothek.io.eager_cube import build_cube
>>> datasets_build = build_cube(
...   data=df_weather,
...   store=store,
...   cube=cube
...)
```


where **store** is the **simplekv** store of storefactory. (For more details, please refer our [Kartothek](2019-05-28-introducing-kartothek.markdown) blog.)

We just have preserved a single Kartothek dataset. Lets print the content of seed dataset.

```python
>>> print(", ".join(sorted(datasets_build.keys())))
seed
>>> ds_seed = datasets_build["seed"].load_all_indices(store)
>>> print(ds_seed.uuid)
geodata++seed
>>> print(", ".join(sorted(ds_seed.indices)))
city, country, day
```

Finally, let’s have a quick look at the store content. Note that we cut out UUIDs and timestamps here for documentation purposes:

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

Thus, A cube can be visualised as a metadataset of multiple datasets as shown in below image.
 
 
![Cube Image](/assets/images/2020-07-21-kartothek-cube.png)


Similarly, we can **Extend** this cube by **Adding** new columns to the dataframes.

## Extend Operation

Now let’s say we also would like to have longitude and latitude data in our cube.

```python
>>> from klee2.io.eager import extend_cube
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

```
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
Note that for the second dataset, no indices for **city** and **day** exists. 
These are only created for the seed dataset, since that datasets forms the groundtruth about which city-day entries are part of the cube.
(Dataset that provides the groundtruth about which Cells are in a Cube is called the **seed dataset**).

If you look at the file tree, you can see that the second dataset is completely separated. This is useful to copy/backup parts of the cube:

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
## Query

The whole beauty of Cube does not come from storing multiple datasets,
but especially from retrieving the data  (**Querying**)  in a very comfortable way. 
Kartothek views the whole cube as a large, virtual DataFrame.
The seed dataset presents the groundtruth regarding rows, all other datasets are joined via a left join. 
Cube naturally supports **partition-by** semantic, which is more helpful for distributed backends.

 
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

 
Similarly, we can **Transform** the cube using **query** and **extend** operations,
 **Append** new rows to the existing cube.
 
## Remove and Delete Operations

You can **Remove** entire partitions from the cube using the remove operation.

```python
>>> from klee2.io.eager import remove_partitions
>>> datasets_after_removal = remove_partitions(
...     cube=cube,
...     store=store,
...     klee_dataset_ids=["latlong"],
...     conditions=(C("country") == "UK"),
... )
>>> query_cube(
...     cube=cube,
...     store=store,
... )[0]
   avg_temp  avg_temp_country_min      city country        day   latitude  longitude
0         8                   6.0   Dresden      DE 2018-01-01  51.050407  13.737262
1         4                   4.0   Dresden      DE 2018-01-02  51.050407  13.737262
2         6                   6.0   Hamburg      DE 2018-01-01  53.551086   9.993682
3         5                   4.0   Hamburg      DE 2018-01-02  53.551086   9.993682
4         6                   6.0    London      UK 2018-01-01        NaN        NaN
5         8                   4.0    London      UK 2018-01-02        NaN        NaN
6        20                   NaN  Santiago      CL 2018-01-01        NaN        NaN
7        22                   NaN  Santiago      CL 2018-01-02        NaN        NaN 
```

You can also **Delete** entire datasets (or the entire cube).

```python
>>> from klee2.io.eager import delete_cube
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
0         8   Dresden      DE 2018-01-01  51.050407  13.737262
1         4   Dresden      DE 2018-01-02  51.050407  13.737262
2         6   Hamburg      DE 2018-01-01  53.551086   9.993682
3         5   Hamburg      DE 2018-01-02  53.551086   9.993682
4         6    London      UK 2018-01-01        NaN        NaN
5         8    London      UK 2018-01-02        NaN        NaN
6        20  Santiago      CL 2018-01-01        NaN        NaN
7        22  Santiago      CL 2018-01-02        NaN        NaN
```

## Additional Features of Cube

*	**Multiple-datasets**: When mapping multiple parts (tables or datasets) to Kartothek, using multiple datasets allows users to copy, backup and delete them separately. 
Index structures are bound to datasets.
*	**Seed-Based Join System / Partition-alignment**: When data is stored in multiple parts (tables or datasets), the question is how to expose it to the user during read operations.
 Seed based Join marks a single part as seed which provides seed dataset in the cube, all other parts are just additional columns.
 Cube uses lazy approach of seed based join, 
 since it better supports independent copies and backups of datasets and also simplifies some of our processing pipelines (e.g. geolocation data can blindly be fetched for too many locations and dates.)	


## Command Line Interface (CLI)
Kartothek features a **command line interface (CLI)** for some cube operations. 
To use it, create a  **skv.yml** file that describes [storefact](https://github.com/JDASoftwareGroup/storefact) stores and use commands to gather information of the required cube.

Sample skv.yml:
```
dataset:
   type: hfs
   path: path/to/data
```

Here we use **geodata** cube to get some information.
```shell
>>>kartothek_cube geodata info  (gives geodata cube info)
>>>kartothek_cube geodata stats  (for cube scan)
>>>kartothek_cube --help  (To get list of  ktk_cube commands)
```

## Outlook
TODO:What features are coming up next ??
