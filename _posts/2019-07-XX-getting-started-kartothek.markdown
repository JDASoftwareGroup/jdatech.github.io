---
layout: single
title:  "Getting started with kartothek"
date:   2019-07-XX XX:XX:XX +0100
tags: technology python data-engineering
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Lucas Rademaker
author_profile: true
---

# Getting started with _kartothek_
This post will give an overview of how to start _using_ kartothek.
For a description of what kartothek _is_ and what it attempts to accomplish,
check out [this blog post]( https://tech.jda.com/introducing-kartothek).

For starters, we will need to have our data as pandas dataframes. For example,
```python
import pandas as pd

df = pd.DataFrame(
    {
        "date": [pd.Timestamp(2020, 1, 1 + d // 5) for d in range(20)],
        "ints": range(0, 2000, 100),
        "floats": [f / 12 for f in range(20)],
        "bool": [True] * 15 + [False] * 5,
        "label": pd.Categorical(["ABC", "XYZ", "BBQ", "ACDC"] * 5),
        "reporter": "Johnny",
    }
)

df2 =  pd.DataFrame(
    {
        "date": [pd.Timestamp(2020, 1, 1 + d // 5) for d in range(20)],
        "ints": range(0, 2000, 100),
        "floats": [f % 12 + f / 12 for f in range(20)],
        "bool": [True] * 15 + [False] * 5,
        "label": pd.Categorical(["CBA", "XYZ", "BBQ", "DCAC"] * 5),
        "reporter": "Bobby",
    }
)
```
## Step 1: Defining our storage location
The next step is deciding where we want to store our data. If you have a small dataset, you can store it on your hard drive. But for large datasets that are used in a distributed fashion, kartothek also supports using blob storage such as Amazon's S3 or Azure Blob Storage. To accomplish this, kartothek uses [storefact](https://github.com/blue-yonder/storefact), which provides an abstraction layer for key-value stores.

In the example below, we will define our store to be a temporary directory on the filesystem.
```python
from functools import partial
from tempfile import TemporaryDirectory
from storefact import get_store_from_url

dataset_dir = TemporaryDirectory()

store_factory = partial(get_store_from_url, f"hfs://{dataset_dir.name}")
```
Note: this example requires a Python version equal or larger than `3.6`.

## Step 2: Writing data to storage
Before writing our data to storage, we can select a scheduling back-end which will affect how our data is handled.

The scheduling back-ends _currently supported_ by kartothek are:
* `dask`, suitable for handling large datasets. There are actually 3 distinct `dask` back-ends (click on the back-end names to see the original documentation):
    - _[`dask.distributed`](https://docs.dask.org/en/latest/delayed.html)_, can be used to run operations in a cluster
    - _[`dask.dataframe`](https://docs.dask.org/en/latest/dataframe.html)_, parallelizes Pandas DataFrames operations.
    - _[`dask.bag`](https://docs.dask.org/en/latest/bag.html)_.
* `eager`, runs all execution immediately and on the local machine.
* `iter`, executes operations on the dataset for each input dataframe. The standard format to read/write for this back-end is by providing a generator of dataframes.

For example, to store the dataframe previously defined using the `eager` back-end, we can execute:
```python
from kartothek.io.eager import store_dataframes_as_dataset

store_dataframes_as_dataset(
    dfs=[df1, df2], store=store_factory, dataset_uuid="my_dataset"
)
```
The signatures of these I/O functions are very similar across back-ends, although some back-ends may lack some of the functions.

We can also partition our data when writing to storage, to enable faster queries for a
particular column or group of columns. To do so, we can pass the columns by which the
dataset sould be partitioned to the ``partition_on`` keyword argument.
For example, to partition on the `"date"` column whereby column

```python
store_dataframes_as_dataset(
    dfs=[df1, df2],
    store=store_factory,
    dataset_uuid="partitioned_dataset",
    partition_on=["date"],
)
```

## Step 3: Reading data
Once we have written our data, we may eventually want to read it again. This is where we would use a `kartothek.io.*.read*` function.

Note: Since we have stored a plain dataframe previously, kartothek assumes the dataset contains only a single table and assigns `"table"` as the name of the table.

If we just want to read the dataframe back in the same way we inserted it, we can use the following code snippet:
```python
from kartothek.io.eager import read_table

read_table(store=store_factory, dataset_uuid="my_dataset", table="table")
```

The same example, but using `dask.delayed`, would look like this:
```python
# This returns a list of `dask.delayed` objects, where each object represents a kartothek partition
dfd = read_table_as_delayed(
    store=store_factory, dataset_uuid="my_dataset", table="table"
)
dfd[0].compute()
```

## Next steps
For more information, take a look at the [documentation](https://kartothek.readthedocs.io/en/latest/).