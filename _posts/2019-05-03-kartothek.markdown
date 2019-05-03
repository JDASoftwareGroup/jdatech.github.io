---
layout: single
title:  "Introducing Kartothek - Consistent parquet table management powered by Apache Arrow and Dask"
date:   2019-05-03 20:00:00 +0100
tags: technology python data-engineering data-science
header:
  overlay_image: assets/images/dataStorm_72DPI.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Florian Jetter
author_profile: true
---
Productionizing Machine Learning is difficult and mostly not about Data Science at all.

The amount of data we need to store and process to do Machine Learning is continuously growing. While the size of the data becomes bigger, our expectations about performance and cost are foolishly evolving in quite the opposite direction. Especially for storing and consuming huge data consistently, existing technologies are usually extremely complex, extremely costly or both. Hitting the right spot between efficiency, cost and comfort is usually crucial to be successful.

In the past we've written a bit about some tools we've built and are using to tackle this. You can read about how we built [turbodbc](https://tech.jda.com/making-of-turbodbc-part-1-wrestling-with-the-side-effects-of-a-c-api/) to efficiently access databases using ODBC drivers or about about how we contribute to, and use [Apache Parquet](https://tech.jda.com/efficient-dataframe-storage-with-apache-parquet/) as a file storage format. Here I want to show you how everything comes together. I want to show you how we leverage Apache Arrow and Apache Parquet on a big scale by building efficient data pipelines using `Kartothek` and [Dask](http://dask.org).

# What is Kartothek?

`Kartothek` is a table management Python library built on [Apache Arrow](https://arrow.apache.org), [Apache Parquet](https://parquet.apache.org) and is powered by [Dask](http://dask.org). It offers a specification for storing tabular data accross multiple files in generic key-value stores, most notably cloud blob stores like Amazon S3, Google Storage or Azure Blob Store. It is designed with compatibility in mind to the implicit standard storage layout used by [Dask](http://dask.org), [Spark](https://spark.apache.org), [Hive](https://hive.apache.org) and more. The library offers building blocks to assemble data pipelines to read, write or modify parquet tables.

The following should give you an impression of what `Kartothek` has to offer:

* Consistent dataset state at all times
* Dataset state is only modified by atomic commits
* Read and write access without the need for any locking mechanism
* Strongly typed and enforced table schemas using Apache Arrow
* `O(1)` remote storage calls to plan job dispatching
* Inverted indices for fast and efficient query planing
* Integrated predicate pushdown on row group level using Apache Parquet
* Portable accross frameworks and languages
* Seemless integration to pandas, dask and the Python ecosystem


# Why build a table management library?

About two years ago we were facing serious scalability issues, both in terms of cost and performance, and we needed a way out. Back then, we were mostly relying on a single, very powerful distributed in-memory database which suited our needs extremely well. Our data got bigger, our time windows shrunk and eventually, adding more and more high-end hardware to our system wasn't an option anymore.

Looking for alternative storage technologies quickly led us down the road of well established technologies like [Hive](https://hive.apache.org), [Presto](https://prestodb.github.io) and the like. The unfortunate thing is that these technologies require a rather complex setup to get working efficiently and on top of that, let's face it, they are written in Java. Usually I argue that a good developer shouldn't care about languages but it's more than that; it's about frameworks, it's about the community. Ultimately, we strongly prefer to have a very good integration our existing technology stack. Essentially, we wanted a native integration to `pandas`.

The direction is set: We want to have a scalable technology built on comodity hardware with native integration to `pandas`. Putting everything together leads you ultimately to `dask` as a computation engine. As others proved already, storing data distributed over multiple files in an object store (S3, ABS, GCS, etc.) allows for a fast, cost efficient and highly scalable data infrastructure. A downside of storing data simply in an object store is that the storages themselves offer little to no guarantees beyond the consistency of a single file. In particular, they cannot guarantee the consistency of your dataset as a whole. If we demand a consistent state of our dataset at all times we need to do track the state of the dataset ourself. Explicit state tracking can be more than a nuisance, though, if done correctly.


# Let's have a look

First of all, `Kartothek` is built in such a way that the pipelines can be assembled using multiple different backends. The logic to describe the management logic is all modularized and can be plugged together with whatever scheduling technology you prefer.
`Kartothek` ships with pipelines ready to be executed and in our example here I want to give a quick glimpse of the `kartothek.io.dask.dataframe` module.

## Our first dataset

Everything starts with a `store` object. We're using the [simplekv](https://github.com/mbr/simplekv) interface to access remote or local storages since it offers a very simple API

```python
>>> from storefact import get_store_from_url

# A store factory is any callable returning a `simplekv` store.
>>> def store_factory():
...     return get_store_from_url("hazure:<uri_to_your_container>")
```


Consider we have already a `dask` DataFrame of some sorts. This may be the result of an analysis, a database download or something similar.

```python
>>> ddf.head()
     A          B    C  D      E    F
0  1.0 2013-01-02  1.0  3   test  foo
1  1.0 2013-01-02  1.0  3  train  foo
2  1.0 2013-01-02  1.0  3   test  foo
3  1.0 2013-01-02  1.0  3  train  foo
```

Now we want to store the data as a kartothek dataset. We can use the `update_*` functionality both to append or to create a new dataset.

```python
from kartothek.io.dask.dataframe import update_dataset_from_ddf

# Kartothek will take the dataset and extend the task
# graph such that the final store operation is consistent
>>> delayed_tasks = update_dataset_from_ddf(
...     ddf,
...     dataset_uuid="my_first_dataset",
...     store=store_factory,
... )

# Up until now nothing has happened. If you want to go through with it, execute the computations on your dask cluster.
# Only if the entire run is successful, the data is commited. Otherwise, intermediate results
# are rejected
>>> delayed_tasks.compute()
```

### Reading data

To read data there are multiple ways of doing so. If you simply want to have it as a `dask.dataframe` the following is the easiest.

```python
from kartothek.io.dask.dataframe import read_dataset_as_ddf
>>> ddf_loc1 = read_dataset_as_ddf(
...    ddf,
...    dataset_uuid="my_first_dataset",
...    store=store_factory,
...    predicates=[[("E", "==", "test")]],
... )
>>> ddf_loc1.compute()
     A          B    C  D      E    F
0  1.0 2013-01-02  1.0  3   test  foo
2  1.0 2013-01-02  1.0  3   test  foo
```
In this example you can also see the predicate pushdown at work. We specified a set of predicates and `Kartothek` evaluates them for you, uses indices and `Apache Parquet` statistics to only retrieve as much data as necessary.

The implementation offers much more than just reading and writing. Lifecycle management, partition compaction, indexing control and much more can be done with usually only very few commands. If you want to know more, visit our [documentation](https://kartothek.readthedocs.io/en/latest) or browse the code directly on [github](https://github.com/JDASoftwareGroup/kartothek).


# Outlook

This is not the end of the line. In the upcoming weeks and months we'll continuously expand the functionality of kartothek. Here are a few highlights of what's to come:

* Snapshot isolation and time travel
* Extended support for different backends (e.g. PySpark)
* Compatibility with Apache Iceberg and/or Delta Lake
