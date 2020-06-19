---
layout: single
title:  "Dask Usage at Blue Yonder"
date:   2020-06-19 10:00:00 +0100
tags: technology python data-engineering
header:
  overlay_image: assets/images/tech_gear_banner.jpg
  overlay_filter: 0.2
  show_overlay_excerpt: false
author: Andreas Merkel, Kshitij Mathur
author_profile: true
---
# Dask Usage at Blue Yonder

Back in 2017, Blue Yonder started to look into 
[Dask/Distributed](https://distributed.dask.org) and how we can leverage it to power
our machine learning and data pipelines.
It's 2020 now, and we are using it heavily in production for performing machine learning based
forecasting and optimization for our customers.
Time to take a a look at the way we are using Dask!

## Use cases

First, let's have a look at the use cases we have for Dask. The use of Dask at Blue Yonder
is strongly coupled to the concept of [Datasets](https://tech.jda.com/introducing-kartothek/),
tabular data stored as [Apache Parquet](https://parquet.apache.org/) files in blob storage. 
We use datasets for storing the input data for our computations as well as intermediate results and the final output data
served again to our customers.
Dask is used in managing/creating this data as well as performing the necessary computations.

The size of the data we work on varies strongly depending on the individual customer.
The largest ones currently amount to about one billion rows per day. 
This corresponds to 30 GiB of compressed Parquet data, which is roughly 500 GiB of uncompressed data in memory.

### Downloading data from a database to a dataset

One typical use case that we have is downloading large amounts of data from a database into a
dataset. We use Dask/Distributed here as a means to parallelize the download and the graph is
an embarassingly parallel one: a lot of independent nodes downloading pieces of data into a blob and one
reduction node writing the [dataset metadata](https://github.com/JDASoftwareGroup/kartothek)
(which is basically a dictionary what data can be found in which blob).
Since the database can be overloaded by too many parallel downloads, we need to limit concurrency.
In the beginning, we accomplished this by using a Dask cluster with a limited the number of workers.
In the meantime, we contributed a 
[Semaphore implementation](https://docs.dask.org/en/latest/futures.html?highlight=semaphore#id1)
to Dask for solving this problem.

### Doing computations on a dataset

A lot of our algorithms for machine learning, forecasting, and optimization work on datasets.
A source dataset is read, computations are performed on the data, and the result is written to
a target dataset.
In many cases, the layout of the source dataset (the partitioning, i.e., what data resides in which blob)
is used for parallelization. This means the algorithms work independently on the individual
blobs of the source dataset. Therefore, our respective Dask graphs again look very simple, with parallel nodes each performing
the sequential operations of reading in the data from a source dataset blob, doing some computation on it,
and writing it out to a target dataset blob. Again, there is a final reduction node writing the target
dataset metadata. We typically use Dask's Delayed interface for these computations.

![Simple Dask graph with parallel nodes and final reduction](/assets/images/2020-03-16-dask-usage-at-by-graph.png)

### Dataset maintenance

We use Dask/Distributed for performing maintenance operations like garbage collection on datasets.
Garbage collection is necessary since we use a lazy approach when updating datasets and only delete or update
references to blobs in the dataset metadata, deferring the deletion of no longer used blobs.
In this case, Dask is used to parallelize the delete requests to the storage provider (Azure blob service).
In a similar fashion, we use Dask to parallelize copy operations for backing up datasets.

### Repartitioning a dataset

Our algorithms rely on source datasets that are partitioned in a particular way. It is not always
the case that we have the data available in a suitable partitioning, for instance, if the data results
from a computation that used a different partitioning. In this case, we have to repartition the data.
This works by reading the dataset as a [Dask dataframe](https://docs.dask.org/en/latest/dataframe.html) 
repartitioning the dataframe using network shuffle, and writing it out again to a dataset.

## Ad hoc data analysis

Finally, our data scientists also use Dask for out-of-core analysis of data using Jupyter notebooks.
This allows us to take advantage of all the nice features of Jupyter even for data that is too large to
fit into an individual compute node's memory.

## Dask cluster setup at Blue Yonder

We run a somewhat unique setup of Dask clusters that is driven by the specific requirements
of our domain (environment isolation for SaaS applications and data isolation between customers). 
We have a large number of individual clusters
that are homogeneous in size with many of the clusters dynamically scaled. The peculiarities of this
setup have in some cases triggered edge cases and uncovered bugs, which lead us to submit a number
of upstream pull requests.

### Cluster separation

To provide isolation between customer environments we run separate Dask clusters per customer.
For the same reason, we also run separate clusters for production, staging, and development environments. 

But it does not stop there. 
The service we provide to our customers is comprised of several products that build upon each other and 
and maintained by different teams. We typically perform daily batch runs with these products running sequentially
in separated environments.
For performance reasons, we install the Python packages holding the code needed for the computations on each worker. 
We do not want to synchronize the dependencies and release cycles of our different products, which
means we have to run a separate Dask cluster for each of the steps in the batch run.
This results in us operating more than
ten Dask clusters per customer and environment, with most of the time, only one of the clusters being active
and computing something. While this leads to overhead in terms of administration and hardware resouces,
it also gives us a lot of flexibility. For instance, we can update the software on the cluster of one part of the compute pipeline
while another part of the pipeline is computing something on a different cluster.

### Some numbers

The number and size of the workers varies from cluster to cluster depending on the degree of parallelism
of the computation being performed, its resource requirements, and the available timeframe for the computation.
At the time of writing, we are running more than 500 distinct clusters.
Our clusters have between one and 225 workers, with worker size varying between 1GiB and 64GiB of memory.
We typically configure one CPU for the smaller workers and two for the larger ones.
While our Python computations do not leverage thread-level parallelism, the Parquet serialization part,
which is implemented in C++, can benefit from the additional CPU.
Our total memory use (sum over all clusters) goes up to as much as 15TiB.
The total number of dask workers we run varies between 1000 and 2000.

![Simple Dask graph with parallel nodes and final reduction](/assets/images/2020-03-16-dask-usage-at-by-n-workers.png)

## Cluster scaling and resilience

To improve manageability, resilience, and resource utilization, we run the Dask clusters on top
of [Apache Mesos](http://mesos.apache.org/)/[Aurora](http://aurora.apache.org/) 
and [Kubernetes](https://kubernetes.io/). This means every worker as well as the scheduler and client
each run in an isolated container. Communication happens via reverse proxies to make the communication
endpoints independent of the actual container instance.

Running on top of a system like Mesos or Kubernetes provides us with resilience since a failing worker 
(for instance, as result of a failing hardware node)
can simply be restarted on another node of the system. 
It also enables us to easily commission or decommission Dask clusters, making the amount of clusters we run
manageable in the first place.

Running 500 Dask clusters also requires a lot of hardware. We have put two measures in place to improve
the utilization of hardware resources: oversubscription and autoscaling.

### Oversubscription

[Oversubscription](http://mesos.apache.org/documentation/latest/oversubscription/) is a feature of Mesos
that allows allocating more resources than physically present to services running on the system.
This is based on the assumption that not all services exhaust all of their allocated resources at the same time.
If the assumption is violated, we prioritize the resouces to the more important ones.
We use this to re-purpose the resources allocated for production clusters but not utilized the whole time
 and use them for development and staging systems.

### Autoscaling

Autoscaling is a mechanism we implemented to dynamically adapt the number of workers in a Dask cluster
to the load on the cluster. This is possible since Mesos/Kubernetes . This
makes it really easy to add or remove worker instances from an existing Dask cluster.

To determine the optimum number of worker instances to run, we added the ``desired_workers`` metric to Distributed.
The metric exposes
the degree of parallelism that a computation has and thus allows us to infer how much workers a cluster should
ideally have. Based on this metric, as well as on the overall resources available and on fairness criteria
(remember, we run a lot of Dask/Distributed clusters), we add or remove workers to our clusters.
To resolve the problem of balancing the conflicting requirements for different resouces like RAM or CPUs
 using [Dominant Resource Fairness](https://cs.stanford.edu/~matei/papers/2011/nsdi_drf.pdf).

## Dask issues

The particular way we use Dask, especially running it in containers connected by reverse proxies and the fact
that we dynamically add/remove workers from a cluster quite frequently for autoscaling has lead us to hit some
edge cases and instabilities and given us the chance to contribute some fixes and improvements to Dask.
For instance, we were able to 
[improve stability after connection failures](https://github.com/dask/distributed/pull/3246) or when 
[workers are removed from the cluster](https://github.com/dask/distributed/pull/3366). 

If you are interested in our contributions to Dask and our commitment to the dask community, please
also check out our blog post [Karlsruhe to D.C. ― a Dask story](2020-05-28-dask-developer-workshop.markdown).

## Conclusion

Overall, we are very happy with Dask and the capabilities it offers. 
Having migrated to Dask from a proprietary compute framework that was developed within our company,
we noticed that we have had similar pain points with both solutions: running into edge cases and robustness
issues in daily operations. 
However, with Dask being an open source solution, we have the confidence that others can also profit from
the fixes that we contribute, and that there are problems we **don't** run into because other people have
already experienced and fixed them.

For the future, we envision adding even more robustness to Dask: 
Topics like scheduler/worker resilience (that is, surviving the loss of a worker or even the scheduler without losing computation results)
and flexible scaling of the cluster are of great interest to us.
