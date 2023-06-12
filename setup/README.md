# Phase 1

The main goal is to have a low priority, best-effort deployment that takes up 90% of the resources of a pod, then run a high priority, high QoS Guaranteed deployment and see how the low priority nodes are evicted.

## Steps

### 1) Create the project and change to it

Let's do our testing in a separate namespace

```bash
oc apply -f project-request.yaml
oc project benchwarming-autoscaling
```

### 2) Ensure your machinepool on `rosa` is set to autoscale

```bash
rosa login
rosa list machinepool --cluster=<myClusterId>
```

If it's not set to autoscale, you can do it like so, let's set it from 2 to 5 nodes.

```bash
rosa edit machinepool --enable-autoscaling --min-replicas=2 --max-replicas=5 --cluster=<myClusterId> <myMachinePoolId>
```

### 2) Create the benchwarming pod

It's been created as a deployment in case you want to play around with scaling, but it only runs a single pod unless specified in a scenario

```bash
oc apply -f benchwarmer-deployment.yaml
```

### 3) Create the high-priority deployment

```bash
oc apply -f high-priority-deployment.yaml
```

it's set to 3 replicas by default.

### 4) Head over to the scenarios and try them

## Initial setup

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
