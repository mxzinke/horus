# Horus

Job & Event distribution without single Point of Failure.

> Warning, this is currently just a test project, like a proof of concept.

# Requirements

We had the following points in our mind.

- infinite horizontal scalability
- scaling through sharding, not replication
- masterless architecture
- no eventloss / dataloss
- continue subscription after failure on same event
- possibility to schedule an event
- no guaranteed global order of events
- client can be publisher and subscriber on lightweight methods
- simple usage on client SDK

# Concept 2



# Concept

The main issue on job distribution in a distributed (cloud) system with many workers is, that you still have a single point
of failure. These SPoF are very often software like Redis. You could do a lot to remove these SPoF's out of you system by
creating a Redis cluster or something like that, but still you have to live with the risk of loosing a job.

This risk could be removed by using a piece software like this one. It makes it possible to select one or more master nodes
under multiple nodes. These master nodes are then managing the job distribution.

With this basic concept it should be possible to spawn up hundreds of worker-nodes in you cloud, without thinking of failing
nodes.

## How does the cluster finds the master?

Internally we do search in the given network (IP-Range), for all online clients. On start up, the worker node is acting as
a master and is trying to tell every other client that he is a slave. The client then could fail or succeed, this depends
on an internally random value, which is generated on start up.

This basically means, that the worker is looking for other workers on start up and evaluates if there is another worker
with a higher value. If there are at least 3 workers have a higher value then its own, he is a slave. Otherwise, he is master.
This means also, there are always 3 masters (or less). If there is a slave with an equal value, he is regenerating the value
imminently. This syncing is always just happening on start up and other nodes are always just informed when they are asked
about their number.

If now one of the master worker are now crashing, the slave nodes are noticing that he is down and now every slave is checking
which node has the next higher value and then can replace him.

## How do the master workers syncing the jobs?

Because every master does know who are the other master workers, they can easily sync their jobs and their state easily
to the other masters. This is the case, for each update to a job and his lifecycle.

## How does the lifecycle of a job look like?

Each job does have a name and a topic. On each worker you can specify how many jobs he can work on in parallel.

1. It is a new job, created through an interval-job or through a worker.
2. A worker is looking for a next job and will acquire the next job on one of the masters
   (the master then is trying to persist a wish of acquire to all other masters, this could also fail)
   
The job is then reserved for a given time to the worker. If the worker is not able to do his work in the given time, the
job will be back free to acquire. Also, the worker is able to tell the master is not yet finished with is work but still
on it.
