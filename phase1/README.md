# Phase 1

The main goal is to have a low priority, best-effort deployment that takes up 90% of the resources of a pod, then run a high priority, high QoS Guaranteed deployment and see how the low priority nodes are evicted.

## Steps

### 1) Create the project

```bash
oc apply -f project-request.yaml
```

### 2) Create machineset

```bash
rosa create machinepool --cluster=<mycluster> --name=mymachinepool-potato --replicas=1 --labels=human-name=potato
```
