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

4 new high priority pods arrive, and there are not enough resources to accomodate them and the previous 3 high priority pods.
