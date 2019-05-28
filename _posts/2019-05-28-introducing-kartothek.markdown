---
layout: single
title:  "Introducing Kartothek - Consistent parquet table management powered by Apache Arrow and Dask"
date:   2019-05-28 20:00:00 +0100
tags: technology python data-engineering
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Florian Jetter
author_profile: true
---
# Introducing Kartothek - Consistent parquet table management powered by Apache Arrow and Dask

Productionizing Machine Learning is difficult and mostly not about Data Science
at all.

The amount of data we need to store and process to do Machine Learning is
continuously growing. While the size of the data increases, our expectations
about performance and cost are foolishly evolving in quite the opposite
direction. Especially for storing and consuming huge data consistently, existing
technologies are usually extremely complex, extremely costly or both. Striking a
balance between efficiency, cost and comfort is usually crucial to success.

In the past we've written a bit about some tools we've built and are using to
tackle this issue. You can read about how we built
[turbodbc](https://tech.jda.com/making-of-turbodbc-part-1-wrestling-with-the-side-effects-of-a-c-api/)
to efficiently access databases using ODBC drivers or about how we contribute
to, and use [Apache
Parquet](https://tech.jda.com/efficient-dataframe-storage-with-apache-parquet/)
as a file storage format. In this blog post, I want to show you how everything
comes together. We want to show you how we leverage [Apache
Arrow](https://arrow.apache.org) and [Apache
Parquet](https://parquet.apache.org) on a big scale by building efficient data
pipelines using [Kartothek](https://github.com/JDASoftwareGroup/kartothek) and
[Dask](http://dask.org).

## What is Kartothek?

A _real_ Kartothek, or slightly more modern a
[Zettelkasten](https://de.wikipedia.org/wiki/Zettelkasten) (Caution: German), is
a tool to organize citations to literature by using a box of index cards. In a
sense, a Kartothek tracks references to information but not the information
itself since the information itself would be too vast to store or access at a
single place.

_Our_ [Kartothek](https://github.com/JDASoftwareGroup/kartothek) is a table
management Python library built on [Apache Arrow](https://arrow.apache.org),
[Apache Parquet](https://parquet.apache.org) and is powered by
[Dask](http://dask.org). It offers a specification for storing tabular data
across multiple files in generic key-value stores, most notably cloud object
stores like Azure Blob Store, Amazon S3 or Google Storage. It is designed with
compatibility in mind to the implicit standard storage layout used by
[Dask](http://dask.org), [Spark](https://spark.apache.org),
[Hive](https://hive.apache.org) and more. The library offers building blocks to
assemble data pipelines to enable reading, writing and modifying parquet tables.
The essential idea is to track the state of a large dataset in few, ease to
understand files which only keep track of the metadata.

The following should give you an impression of what `Kartothek` has to offer:
* Consistent dataset state at all times
* Dataset state is only modified by atomic commits
* Fast, scalable read access without any locking mechanism
* Strongly typed and enforced table schemas using Apache Arrow
* `O(1)` remote storage calls to plan job dispatching
* Inverted indices for fast and efficient querying 
* Integrated predicate pushdown on partition and row group level using Apache
  Parquet
* Portable specification across frameworks and languages
* Seamless integration to pandas, Dask and the Python ecosystem

## Why build a table management library?

About two years ago, we were facing serious scalability issues, both in terms of
cost and performance, and we needed a way out. Back then, we were mostly relying
on a single, very powerful distributed in-memory database which suited our needs
extremely well. Our data volume has increased, our available processing window
has shrunk eventually, adding more and more high-end hardware to our system
wasn't an option anymore.

Looking for alternative storage technologies quickly led us down the road of
well established technologies like [Hive](https://hive.apache.org),
[Presto](https://prestodb.github.io) and the like but that was uncharted
territory for us. Our core business and tech stack is built around the Python
Data and Machine Learning stack. How can we build a reliable, easy to use bridge
between the worlds? We wanted to start small and not couple ourselves to a yet
unknown, heavy weight technology without knowing exactly where we want and need
to go (Every large scale architecture transformation is scary, after all). What
all of these technologies have in common is that they support the Parquet file
format and this is where our journey began.

We wanted to have a scalable technology running on commodity hardware with
native integration to [pandas](https://pandas.pydata.org). The storage layer
should use Parquet as a file format and be compatible to the big guns out there
to allow the exchange of data even beyond our micro cosmos of Python. Putting
everything together lead us to [Dask](https://dask.org) as a computation engine
and public cloud object stores (ABS, GCS, S3, etc.) as a storage technology.

Storing data distributed over multiple files in an object store allows for a
fast, cost efficient and highly scalable data infrastructure. A downside of
storing data simply in an object store is that the storages themselves offer
little to no guarantees beyond the consistency of a single file. In particular,
they cannot guarantee the consistency of your dataset as a whole. What happens
if jobs die, retry or try to write schema violating data?

If we demand a consistent state of our dataset at all times, we need to track
the state of the dataset ourselves. Explicit state tracking can be more than a
nuisance, though, if done correctly. If done right, this opens the possibility
for atomic updates, indexing and much more. This is what `Kartothek` does for
you behind the scenes, hidden behind an easy to consume, well integrated
interface


## An Example

First of all, `Kartothek` is built in such a way that data pipelines can be
assembled using different backends. The logic to describe the file management is
all modularized and can be plugged to whatever scheduling technology you prefer.
`Kartothek` ships with pipelines ready to be executed and in our example here I
want to give a quick glimpse of the `kartothek.io.dask.dataframe` module.

### Our first dataset

Everything starts with a `store` object. We're using the
[simplekv](https://github.com/mbr/simplekv) interface to access remote or local
storages since it offers a very simple and convenient API. The store itself is
initialized using [storefact](https://github.com/blue-yonder/storefact), also a
tools of our own making, since switching between stores is as simple as
replacing a string.
```python
>>> from storefact import get_store_from_url
# A store factory is any callable returning a `simplekv` store.
>>> def store_factory():
# This store would connect to an Azure Blob Store
... return get_store_from_url("hazure://<account_name>:<account_key>@container_name")
```
Consider we have already a `Dask` DataFrame of some sorts. This may be the
result of an analysis, a database download or something similar.
```python
>>> ddf.head()
A B C D E F
0 1.0 2013-01-02 1.0 3 test foo
1 1.0 2013-01-02 1.0 3 train foo
2 1.0 2013-01-02 1.0 3 test foo
3 1.0 2013-01-02 1.0 3 train foo
```
Now we want to store the data as a Kartothek dataset. We can use the `update_*`
functionality both to append or to create a new dataset.
```python
from kartothek.io.dask.dataframe import update_dataset_from_ddf
# Kartothek will take the dataset and extend the Dask task
# graph such that the final store operation is consistent
>>> delayed_tasks = update_dataset_from_ddf(
... ddf,
... dataset_uuid="my_first_dataset",
... store=store_factory,
... )
# Up until now nothing has happened. If you want to go through with it, execute the computations on your Dask cluster.
# Only if the entire run is successful, the data is committed. Otherwise, intermediate results are not committed and may
# be garbage collected later on.
>>> delayed_tasks.compute()
```
Reading data

There are multiple ways to read data. If you simply want to have it as a
`dask.dataframe` the following is the easiest.
```python
from kartothek.io.dask.dataframe import read_dataset_as_ddf
>>> ddf_loc1 = read_dataset_as_ddf(
... ddf,
... dataset_uuid="my_first_dataset",
... store=store_factory,
... predicates=[[("E", "==", "test")]],
... )
>>> ddf_loc1.compute()
A B C D E F
0 1.0 2013-01-02 1.0 3 test foo
2 1.0 2013-01-02 1.0 3 test foo
```
In this example you can also see the predicate pushdown at work. We specified a
set of predicates and `Kartothek` evaluates them for you, uses indices and
`Apache Parquet` statistics to retrieve only the necessary data. In this simple
example this is hardly impressive but when processing hundreds of GB or TB of
data this allows you to have extremely fast query times! The implementation
offers much more than just reading and writing. Lifecycle management, partition
compaction, indexing control and much more can be done with usually only very
few commands. If you want to know more, visit our
[documentation](https://kartothek.readthedocs.io/en/latest) or browse the code
directly on [github](https://github.com/JDASoftwareGroup/kartothek).

## Outlook

This is not the end of the line. In the upcoming weeks and months we'll
continuously expand the functionality of `Kartothek`. Here are a few highlights
of what's to come:

#### Snapshot isolation and time travel

A very strong feature we currently lack is proper concurrent write control. A
simple form of this is called [snapshot
isolation](https://en.wikipedia.org/wiki/Snapshot_isolation) and will be one of
the most important additions to the current implementation.

#### Extended support for different backends (e.g. PySpark)

While our poison is definitely `Dask` we see strong value in having integrations
into other scheduling engines. One of the most promising candidates would be
[PySpark](https://spark.apache.org) since `Spark` itself has been around for
quite some time now and is also well integrated in many companies and is loved
for its performance.

#### Compatibility with Apache Iceberg and/or Delta Lake

We know that we're not the only player in town. Most notably [Apache Iceberg
(incubating)](https://iceberg.apache.org) and [Delta Lake](https://delta.io)
offer very similar specifications and guarantees; set in the Java world. We
believe this synergy should be embraced and we will strive to standardize the
way tables are stored and will try to establish compatibility between the
different storage specifications. In the end, the real power of a tables in an
object store, data lake, etc. is if we can share them easily within an
organization.


The initial public release of `Kartothek` is version `3.0.0` and is available on
PyPI and conda-forge. If you're interested, check out the code on
[github](https://github.com/JDASoftwareGroup/kartothek). We’re welcoming
contributions of all kinds (documentation, build infra, bug fixes, features,
etc.) and are happy to help you with your first steps.

Stay tuned for more content about Kartothek to be published on our blog and make
sure to follow us on [twitter](https://twitter.com/TechJda).
