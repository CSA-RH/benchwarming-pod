# Scenario 2: Scaling up massively, needing 2 spare nodes

8 new high priority pods arrive, and there are not enough resources to accomodate them and the previous 3 high priority pods.

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

First of all, since we know that we will have sudden bursts of pods, which a single spare node might not be able to accomodate, we will add a second spare node

```bash
oc scale --replicas=2 deployment benchwarmer-pod
```

> 🕣 **Time taken**: 5 minutes

This is the current state of the cluster

```mermaid
%%{init: {"flowchart": { "useMaxWidth": false } }}%%
graph TB
   subgraph NodeC
       bw2(Benchwarmer pod 2)
   end
   subgraph NodeB
       bw1(Benchwarmer pod 1)
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
   class bw1,bw2 lowprio;
   class NodeA,NodeB,NodeC cluster;
```

Let's add 8 more high priority nodes:

```bash
oc scale --replicas=11 deployment high-priority-pod
```

Here is what will happen:

1) The 8 new pods will be scheduled to be deployed
    > 🕣 **Time taken**: instantaneous
2) The scheduler will try to evenly distribute the load between the nodes, there is plenty of room in Node B and C, and a little on Node A, so it will spread them accordingly.
    > 🕣 **Time taken**: instantaneous
3) Node B and C has benchwarming pods 1 and 2 respectively, which have an set of antiaffinity rules so they cannot be in a node with anything else, thus it will be evicted when the high priority pods arrive (since benchwarmer pods have `priorityClassName: low-priority` within their spec)
    > 🕣 **Time taken**: instantaneous

```mermaid
%%{init: {"flowchart": { "useMaxWidth": false } }}%%
graph TB
   subgraph NodeC
       hp11(High priority pod 11)
       hp10(High priority pod 10)
       hp9(High priority pod 9)
       hp8(High priority pod 8)
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

```mermaid
graph TB
   subgraph Pending to be scheduled
       bw2(Benchwarmer pod 2)
       bw1(Benchwarmer pod 1)
       bw1 ~~~ bw2
   end

   classDef lowpriopending fill:#1111a5,stroke:#fff,stroke-width:4px,color:#fff;
   classDef cluster fill:#fff,stroke:#bbb,stroke-width:2px,color:#326ce5;
   class bw1,bw2 lowpriopending;
```

1) Two pending pods tell the node autoscaler to provision a new node for each.
    > 🕣 **Time taken**: 5 minutes

## ✅ Final result

Despite the cloud provider requiring around 5 minutes to provision two new node2, we had 0 high priority pod downtime thanks to the "spare" nodes kept around by the benchwarming pod

```mermaid
%%{init: {"flowchart": { "useMaxWidth": false } }}%%
graph TB
   subgraph NodeE
       bw2(Benchwarmer pod 2)
   end
   subgraph NodeD
       bw1(Benchwarmer pod 1)
   end
   subgraph NodeC
       hp11(High priority pod 11)
       hp10(High priority pod 10)
       hp9(High priority pod 9)
       hp8(High priority pod 8)
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
   class bw1,bw2 lowprio;
   class NodeA,NodeB,NodeC,NodeD,NodeE cluster;
```
