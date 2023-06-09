# Pod scaling test

This repository was created with the goal of showcasing some scenarios where two sets of pods compete for resources:

- High priority pods: Very important! should not be evicted and should run asap
- Low priority pods: We should take a best effort approach and try to fill any resources that the high priority pods leave unused.

## Assumptions

To demo's sake, we will assume the following:

- Cluster has node autoscaling from 2 to 5 nodes
- Nodes have 12GB of RAM each
- All pods **request** 2GB of RAM

## Setup

Head over to the setup folder and create all the resources in the `.yaml` files.

You should end up with 2 nodes, one filled with 5 low priority pods and the other with a single low priority pod. No high priority pods yet.

## Scenario 1: Scaling up, needing a new node

[🔗 Link](./scenario1.md)

4 new high availability pods arrive, and there are not enough resources to accomodate them and the previous 6 low priority nodes.

## Scenario 2: Scaling down, removing underused node

[🔗 Link](./scenario2.md)

We no longer require this many nodes, let's downscale the cluster by removing unused nodes and proving there is no downtime

## Scenario 3: Overflowing on only high priority pods

[🔗 Link](./scenario3.md)

The cluster only has high priority pods, what if we add more but there are no resources available?

## Scenario 4: Overflowing even loads, reaching node scaling limit

[🔗 Link](./scenario4.md)

We have 4 nodes up and running with an even load of both high and low priority pods. What happens if we overload the cluster, reaching the node autoscaler max count? We currently have our node autoscaler from 2 nodes to 5 nodes.
