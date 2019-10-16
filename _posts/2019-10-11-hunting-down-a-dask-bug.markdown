---
layout: single
title:  "Hunting down a Dask Bug - or What to Do if You're Randomly Losing Data"
date:   2019-05-28 20:00:00 +0100
tags: technology python data-engineering
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Andreas Merkel
author_profile: true
---
# Hunting down a Dask Bug - or What to Do if You're Randomly Losing Data

When introducing a new technology into your stack, you have to be prepared to discover
bugs. You are using someone else's work, and probably in a way that is, to some extent,
different to the way others have used it before you. And this does not finish when you
are done with your implementation and testing, but extends to the time when you
finally put it in production.

This is a story about how subtle bugs can manifest themselves after months of having
a technology in production, and how tricky it can be to track them down.

## The setup

We are currently in the process of adopting
[Dask/Distributed](https://distributed.dask.org) for distributed computing and are using
it, among other things, to work with data in flat files on BLOB storage (cf.
[Introducing Kartothek - Consistent parquet table management powered by Apache Arrow and
Dask](https://tech.jda.com/introducing-kartothek/).)
A typical use case we have for Dask/Distributed is reading a collection of flat files,
which we call *dataset*, doing some calculation on the data and writing it again to
another dataset.

We had been gradually switching more and more of our data pipelines to Dask/Distributed
in favor of a proprietary distributed computing framework that we want to phase out when
things started to get a tad mysterious and data began to disappear in an unexpected way.

## The incident

It started when one of our customers raised an incident complaining that we did not
provide any data for one of their stores via our API one day. We provide data to them
that is calculated on a daily basis, and so far, everything had been OK for months after
switching to Dask/Distributed for data processing.

The API provides the data from a dataset (let's call it _output dataset_), which is
filled from another dataset using Dask/Distributed (let's call this one _input dataset_)
by shuffling around the data. The shuffling is done because the input dataset is
organized in a way that reflects the customer's stores (basically one [Apache
Parquet](https://parquet.apache.org) file per store), while the output dataset is
organized by time (one Parquet file per day, containing data for all the stores).

## The solution (as it seemed)

A first investigation revealed that indeed, the store the customer complained about
was not contained in the ouptut dataset, whereas it was contained in the input dataset.
Therefore, the data must be lost somewhere in between during the shuffling.

We had a look ad the Distributed cluster and sure enough, we discovered it was in a
weird state. As it turned out, the day before, some of the hardware nodes hosting this
cluster had been rebooted. This affected the scheduler and two of the workers, while
the two other workers of the cluster were running on nodes that were not rebooted.
As it looked like, after the reboot of the scheduler, these two workers were not able to
connect to the new scheduler, so we ended up with a cluster with only two workers,
while the two old workers were running in an endless loop trying to connect.

Our assumption was that somehow, parts of the computations got scheduled to the invalid
workers which did not deliver a result and thus the data got lost.
To resolve this, we restarted the scheduler and all workers once again to get a
healthy cluster. We re-ran the data shuffling process from the input dataset
to the output dataset and checked that the store that had previously been missing was
contained in the output dataset now. Thus, we informed the customer that the problem
had been resolved.

## Data loss strikes again

Unfortunately, the customer got back to us, reporting that now a *different* store was
missing. Indeed, we could verify that the problem persisted and each time, a different
**random** store was missing. We also checked whether any changes had been done to the
system recently, but besides the node reboots, nothing had been changed for weeks,
in particular, we had been running exactly the same software versions.

At this point in time, it became clear to us that we had to dig deeper into this.
We began manually retracing each of the steps done during the data shuffling.
The first step is reading in the data from the input dataset into a Dask distributed
dataframe. A distributed dataframe can
be seen as a large dataframe that is divided into several partitions that potentially
live on different Distributed workers.
We could verify that after this step, all of the stores were still contained
in the data, so reading was not the problem. However, we made an interesting observation
at this point. After reading in the data, we have one partition
for each customer store, 221 in our case. And the store that was missing in the end was
contained in the 221st partition of the distributed dataframe. So for some reason, the
last partition seemed to be lost later on. And we were quite eager to find out how this
happened.

The next step after reading the data is repartitioning the distributed dataframe.
To be able to better handle the data during
later steps, we reduce the number of partitions, in this case to 23 partitions. And this
turned out to be the step during which the data was lost. So for some reason, while
repartitioning, the last of the input partitions was not considered.

## Looking for the root cause

Now we knew that the problem happened during repartitioning, we took a look at the
repartitioning code in Dask, and quickly enough found a piece of code that looked quite
suspicious ([https://github.com/dask/dask/blob/1.2.0/dask/dataframe/core.py#L438](
https://github.com/dask/dask/blob/1.2.0/dask/dataframe/core.py#L43866)):

```python
npartitions_ratio = df.npartitions / npartitions
new_partitions_boundaries = [int(new_partition_index * npartitions_ratio)
                             for new_partition_index in range(npartitions + 1)]
```

To map the old partition boundaries to the new partition boundaries, division
and multiplication are used. We are in Python 3 here, otherwise, the problem would have
manifested itself much more prominently. In Python 3, the `/` operator does a float
division. However, division and successive multiplication with the same value still
does not necessarily result in the original values. There are edge cases for which
the result is slightly different because of rounding errors, for instance, in case
of the last value of the list comprehension when mapping 221 to 23 partitions:

```python
>>> (221/23)*23
220.99999999999997
```

The casting to int that happens then will result in a value that is off by one:
```python
>>> int((221/23)*23)
220
```

This explains why the last partition got lost. For many other combinations of values,
this edge case is not triggered. In hindsight, we found out that the customer had
increased the number of stores for which to calculate data from 201 to 221, which
explains why we had not observed the error before.

## The fix

We were running Dask version 1.2.0, which was already quite dated at the time, so we
decided to take a shot and just try the newest Dask version to see if the problem had
been fixed in the meantime. While we did not find any bug report that went into the
direction, the changelog mentioned refactoring of the repartitioning code, so it seemed
worth a try. And indeed, with Dask 2.3.0, we did not observe the effect.

Some `git bisect` later, we discovered the following piece of code that had been
introduced during the refactoring (
[https://github.com/dask/dask/commit/c4b677086a20a73977ba5731b0798db0032cbba5#diff-492da7893fa5fa6ad3a6ef6cdef985f2R4821](https://github.com/dask/dask/commit/c4b677086a20a73977ba5731b0798db0032cbba5#diff-492da7893fa5fa6ad3a6ef6cdef985f2R4821)
):

```python
if new_partitions_boundaries[-1] < df.npartitions:
￼        new_partitions_boundaries.append(df.npartitions)
```

## The aftermath

While this confirmed to us we were safe with Dask > 2, some things were still left to be
done. To prevent a regression, we submitted a [pull request](https://github.com/dask/dask/pull/5433) to Dask that adds a test
making sure that repartitioning works correctly for the edge case.

Also, we had been using Dask 1.2.0 and its repartitioning code for months and were a bit
worried whether there were other instances of data loss. So we went over all the
input datasets and counted the partitions and the repartition ratios to check whether
we had run into the edge case, which was fortunately not the case.

We also took the learning of not being too quick at accepting something that obviously
is broken (in our case the cluster after the node reboots) as the cause for a problem.
While we did verify that the formerly missing store was in the output dataset after
fixing the cluster, we did not verify that **all** the expected stores were in the
output dataset, so we get hit by the fact that the effect was indeterministic (it
depended on the ordering of the input partitions, which is not always the same).
