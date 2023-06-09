# Scenario 3

Let's assume we have high priority pods filling up 2 nodes and no low priority pods. What would happen if we add a new high priority pod?

## Cleanup

Delete old pods from other scenarios

```bash
oc delete deployment low-priority-pod
oc delete deployment high-priority-pod
```

Run the high priority deployment again

```bash
oc apply -f high-priority-pod.yaml
```

## Setup

There should be no pods in the namespace, let's create some high priority ones!

```bash
oc scale --replicas=8 deployment high-priority-pod
```

> 🕣 **Time taken**: instantaneous

This is the current state of the cluster afterwards

```mermaid
graph TB
   subgraph NodeB
       hp8(High priority pod 8)
       hp7(High priority pod 7)
       hp6(High priority pod 6)
       hp5(High priority pod 5)
   end
   subgraph NodeA
       hp4(High priority pod 4)
       hp3(High priority pod 3)
       hp2(High priority pod 2)
       hp1(High priority pod 1)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9 highprio;
   class lp1,lp2,lp3,lp4,lp5,lp6,lp7,lp8,lp9 lowprio;
   class NodeA,NodeB,NodeC cluster;
```

Let's scale from 8 high priority replicas up to 10

```bash
oc scale --replicas=10 deployment high-priority-pod
```

But there is no space left in the 2 nodes, so:

1) The scheduler has no place to deploy the pods, so they're set as pending

    > 🕣 **Time taken**: instantaneous

2) The node autoscaler kicks in and spawns a new node

    > 🕣 **Time taken**: 7-15 minutes

3) The scheduler adds the pending pods

    > 🕣 **Time taken**: instantaneous

So it'll go from this:

```mermaid
graph TB
   subgraph NodeB
       hp8(High priority pod 8)
       hp7(High priority pod 7)
       hp6(High priority pod 6)
       hp5(High priority pod 5)
   end
   subgraph NodeA
       hp4(High priority pod 4)
       hp3(High priority pod 3)
       hp2(High priority pod 2)
       hp1(High priority pod 1)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9 highprio;
   class lp1,lp2,lp3,lp4,lp5,lp6,lp7,lp8,lp9 lowprio;
```

```mermaid
graph TB
   subgraph Pending to be scheduled
       hp10(High priority pod 10)
       hp9(High priority pod 9)
       hp10 ~~~ hp9
   end

   classDef highprio fill:#880000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class hp9,hp10 highprio;
```

To this:

```mermaid
graph TB
   subgraph NodeC
       hp10(High priority pod 10)
       hp9(High priority pod 9)
   end
   subgraph NodeB
       hp8(High priority pod 8)
       hp7(High priority pod 7)
       hp6(High priority pod 6)
       hp5(High priority pod 5)
   end
   subgraph NodeA
       hp4(High priority pod 4)
       hp3(High priority pod 3)
       hp2(High priority pod 2)
       hp1(High priority pod 1)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9,hp10 highprio;
   class lp1,lp2,lp3,lp4,lp5,lp6,lp7,lp8,lp9 lowprio;
   class NodeA,NodeB,NodeC cluster;
```

So there is a downtime for high priority pods since it's the only load and the scheduler cannot evict any low priority pods inmediately to replace them.

In this case we can't do anything but wait for the node autoscaler to kick in.
