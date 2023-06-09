# Scenario 2

We start from [🔗 Scenario 1](./scenario1.md), but de-scaling back: What if we delete half the high priority load?

## Cleanup

If you have done scenario 1, skip this step, if not then clean it up via:

Delete old pods from other scenarios

```bash
oc delete deployment low-priority-pod
oc delete deployment high-priority-pod
```

Run the deployments again

```bash
oc apply -f low-priority-pod.yaml
oc apply -f high-priority-pod.yaml
```

scale low priority pods

```bash
oc scale --replicas=4 deployment high-priority-pod
```

## Setup

This is the current state of the cluster

```mermaid
graph TB
   subgraph NodeC
       lp2(Low priority pod 2)
       lp1(Low priority pod 1)
   end
   subgraph NodeB
       hp3(High priority pod 3)
       hp2(High priority pod 2)
       hp1(High priority pod 1)
       lp6(Low priority pod 6)
   end
   subgraph NodeA
       hp4(High priority pod 4)
       lp5(Low priority pod 5)
       lp4(Low priority pod 4)
       lp3(Low priority pod 3)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9 highprio;
   class lp1,lp2,lp3,lp4,lp5,lp6,lp7,lp8,lp9 lowprio;
   class NodeA,NodeB,NodeC cluster;
```

Let's scale from 4 high priority replicas down to 2

```bash
oc scale --replicas=2 deployment high-priority-pod
```

> 🕣 **Time taken**: instantaneous

```mermaid
graph TB
   subgraph NodeC
       lp2(Low priority pod 2)
       lp1(Low priority pod 1)
   end
   subgraph NodeB
       hp1(High priority pod 1)
       lp6(Low priority pod 6)
   end
   subgraph NodeA
       hp4(High priority pod 4)
       lp5(Low priority pod 5)
       lp4(Low priority pod 4)
       lp3(Low priority pod 3)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9 highprio;
   class lp1,lp2,lp3,lp4,lp5,lp6,lp7,lp8,lp9 lowprio;
   class NodeA,NodeB,NodeC cluster;
```

Now we wait for the node autoscaler to do its job and move the pods around the nodes to save resources, downscaling our node list from 3 to 2

> 🕣 **Time taken**: 10 minutes

leaving the cluster like this:

```mermaid
graph TB
   subgraph NodeB
       lp2(Low priority pod 2)
       lp1(Low priority pod 1)
       hp1(High priority pod 1)
       lp6(Low priority pod 6)
   end
   subgraph NodeA
       hp4(High priority pod 4)
       lp5(Low priority pod 5)
       lp4(Low priority pod 4)
       lp3(Low priority pod 3)
   end
 
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   classDef subg padding: 50%;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9 highprio;
   class lp1,lp2,lp3,lp4,lp5,lp6,lp7,lp8,lp9 lowprio;
   class NodeA,NodeB,NodeC cluster;
```
