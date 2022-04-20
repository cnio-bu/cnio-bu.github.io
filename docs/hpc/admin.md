# Cluster administration

## Rack layout

![Rack layout](../img/cluster1_racks.svg)

Diagram can be edited with [diagrams.net](https://app.diagrams.net/) using the
source code available [here](../img/cluster1_racks.drawio)

## Checking and restarting nodes
For a full list of all available nodes, type:

```
scontrol show node
```

A given node status can be checked out with:

```
scontrol show node NODENAME
```

Sometimes, a node state will change to _DRAIN_ for multiple reasons: i.e. slurm
could not gracefully kill a job on a given node. Drained nodes won't handle incoming
jobs until they are manually reset. To do so, the following command must be typed with
admin privileges: 

```
scontrol update NodeName=NODENAME State=RESUME
```
