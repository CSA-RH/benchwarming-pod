# Scenario 4: Overflow

We have 4 nodes up and running with an even load of both high and low priority pods. What happens if we overload the cluster, reaching the node autoscaler max count? We currently have our node autoscaler from 2 nodes to 5 nodes.

## Cleanup

Delete old pods from other scenarios

```bash
oc delete deployment benchwarmer
oc delete deployment high-priority-pod
```

Run the deployments again

```bash
oc apply -f benchwarmer.yaml
oc apply -f high-priority-pod.yaml
```

## Setup

Let's fill the cluster up!

```bash
oc scale --replicas=8 deployment benchwarmer
oc scale --replicas=8 deployment high-priority-pod
```

This is the current state of the cluster afterwards

```mermaid
graph TB
   subgraph NodeD
       lp8(Low priority pod 8)
       hp8(High priority pod 8)
       hp7(High priority pod 7)
       lp7(Low priority pod 7)
   end
   subgraph NodeC
       hp6(High priority pod 6)
       lp4(Low priority pod 4)
       lp5(Low priority pod 5)
       lp6(Low priority pod 6)
   end
   subgraph NodeB
       hp5(High priority pod 5)
       lp3(Low priority pod 3)
       hp4(High priority pod 4)
       hp3(High priority pod 3)
   end
   subgraph NodeA
       hp2(High priority pod 2)
       lp2(Low priority pod 2)
       lp1(Low priority pod 1)
       hp1(High priority pod 1)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9 highprio;
   class lp1,lp2,lp3,lp4,lp5,lp6,lp7,lp8,lp9 lowprio;
   class NodeA,NodeB,NodeC,NodeD cluster;
```

Great! Although we are pretty close to the 5 node maximum limit!

Let's add 3 more low priority pods

```bash
oc scale --replicas=13 deployment benchwarmer
```

> 🕣 **Time taken**: 6 minutes

Our setup now looks like this

```mermaid
graph TB
   subgraph Pending
       lp13(Low priority pod 13)
   end
   subgraph NodeE
       lp12(Low priority pod 12)
       lp11(Low priority pod 11)
       lp10(Low priority pod 10)
       lp9(Low priority pod 9)
   end
   subgraph NodeD
       lp8(Low priority pod 8)
       hp8(High priority pod 8)
       hp7(High priority pod 7)
       lp7(Low priority pod 7)
   end
   subgraph NodeC
       hp6(High priority pod 6)
       lp4(Low priority pod 4)
       lp5(Low priority pod 5)
       lp6(Low priority pod 6)
   end
   subgraph NodeB
       hp5(High priority pod 5)
       lp3(Low priority pod 3)
       hp4(High priority pod 4)
       hp3(High priority pod 3)
   end
   subgraph NodeA
       hp2(High priority pod 2)
       lp2(Low priority pod 2)
       lp1(Low priority pod 1)
       hp1(High priority pod 1)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   classDef lowpriopending fill:#1111a5,stroke:#fff,stroke-width:4px,color:#fff;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9,hp10,hp11,hp12,hp13,hp14 highprio;
   class lp1,lp2,lp3,lp4,lp5,lp6,lp7,lp8,lp9,lp10,lp11,lp12 lowprio;
   class lp13 lowpriopending;

   class NodeA,NodeB,NodeC,NodeD cluster;
```

Not an ideal scenario, but the only pod pending is a low priority, and all high priority pods are up and running.

What if we hit it with a high priority pod load? maybe 3 more pods

```bash
oc scale --replicas=13 deployment benchwarmer
```

> 🕣 **Time taken**: instantaneous (no new nodes can be created)

Since there is a pod pending, you'd think it'd get scheduled first, before the new payload. However the `PriorityClass` makes the scheduler prioritize the new pods

```mermaid
graph TB
   subgraph Pending
       lp7(Low priority pod 7)
       lp2(Low priority pod 2)
       lp5(Low priority pod 5)
       lp11(Low priority pod 11)
       lp10(Low priority pod 10)
       lp13(Low priority pod 13)
   end
   subgraph NodeE
       hp13(High priority pod 13)
       hp12(High priority pod 12)
       lp12(Low priority pod 12)
       lp9(Low priority pod 9)
   end
   subgraph NodeD
       hp11(High priority pod 11)
       lp8(Low priority pod 8)
       hp8(High priority pod 8)
       hp7(High priority pod 7)
   end
   subgraph NodeC
       hp10(High priority pod 10)
       hp6(High priority pod 6)
       lp4(Low priority pod 4)
       lp6(Low priority pod 6)
   end
   subgraph NodeB
       hp5(High priority pod 5)
       lp3(Low priority pod 3)
       hp4(High priority pod 4)
       hp3(High priority pod 3)
   end
   subgraph NodeA
       hp9(High priority pod 9)
       hp2(High priority pod 2)
       lp1(Low priority pod 1)
       hp1(High priority pod 1)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   classDef lowpriopending fill:#1111a5,stroke:#fff,stroke-width:4px,color:#fff;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9,hp10,hp11,hp12,hp13,hp14 highprio;
   class lp1,lp3,lp4,lp6,lp8,lp9,lp12 lowprio;
   class lp2,lp7,lp5,lp10,lp11,lp13 lowpriopending;

   class NodeA,NodeB,NodeC,NodeD cluster;
```