# Scenario 1: Scaling up, needing a new node

4 new high priority pods arrive, and there are not enough resources to accomodate them and the previous 3 high priority pods.

## 🧹 Cleanup (if you have done other scenarios before)

Delete old pods from other scenarios

```bash
oc delete deployment benchwarmer
oc delete deployment high-priority-pod
```

> ⚠️ Wait for pods to be terminated and make let the node autoscaler do its work dialing the nodes back to 2!

Run the deployments again

```bash
oc apply -f benchwarmer.yaml
oc apply -f high-priority-deployment.yaml
```

## 📝 Setup

This is the current state of the cluster

```mermaid
%%{init: {"flowchart": { "useMaxWidth": false } }}%%
graph TB
   subgraph NodeB
       bw(Benchwarmer pod)
   end
   subgraph NodeA
       hp3(High priority pod 3)
       hp2(High priority pod 2)
       hp1(High priority pod 1)
   end
 
   classDef plain fill:#ddd,stroke:#fff,stroke-width:4px,color:#000;
   classDef lowprio fill:#326ce5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef highprio fill:#ee0000,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9,hp10,hp11,hp12,hp13,hp14,hp15,hp16,hp17,hp18,hp19,hp20 highprio;
   class bw lowprio;
   class NodeA,NodeB cluster;
```

Let's add 4 more high priority nodes:

```bash
oc scale --replicas=7 deployment high-priority-pod
```
> 🕣 **Time taken**: instantaneous

Here is what will happen:

1) The 4 pods will be scheduled to be deployed
    > 🕣 **Time taken**: instantaneous
2) The scheduler will try to evenly distribute the load between the nodes, there is plenty of room in Node B and a little on Node A, so it will spread them accordingly.
    > 🕣 **Time taken**: instantaneous
3) Node B has the benchwarming pod, which has an antiaffinity rule so it cannot be in a node with anything but its own type of pod, so it will be evicted when the high priority pods arrive (since they have `priorityClassName: high-priority` within their spec)
    > 🕣 **Time taken**: instantaneous

```mermaid
%%{init: {"flowchart": { "useMaxWidth": false } }}%%
graph TB
   subgraph NodeB
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
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9,hp10,hp11,hp12,hp13,hp14,hp15,hp16,hp17,hp18,hp19,hp20 highprio;
   class bw lowprio;
   class NodeA,NodeB,NodeC,NodeD,NodeE cluster;
```

```mermaid
graph TB
   subgraph Pending to be scheduled
       bw(Benchwarmer pod)
   end

   classDef lowpriopending fill:#1111a5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class bw lowpriopending;
```

1) A pending pod tells the node autoscaler to provision a new node for it.
    > 🕣 **Time taken**: 7 minutes

## ✅ Final result

Despite the cloud provider requiring around 7 minutes to provision a new node, we had 0 high priority pod downtime thanks to the "spare" node kept around by the benchwarming pod

```mermaid
%%{init: {"flowchart": { "useMaxWidth": false } }}%%
graph TB
   subgraph NodeC
       bw(Benchwarmer pod)
   end
   subgraph NodeB
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
   class hp1,hp2,hp3,hp4,hp5,hp6,hp7,hp8,hp9,hp10,hp11,hp12,hp13,hp14,hp15,hp16,hp17,hp18,hp19,hp20 highprio;
   class bw lowprio;
   class NodeA,NodeB,NodeC,NodeD,NodeE cluster;
```
