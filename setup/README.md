# Phase 1

The main goal is to have a low priority, best-effort deployment that takes up 90% of the resources of a pod, then run a high priority, high QoS Guaranteed deployment and see how the low priority nodes are evicted.

## Steps

### 1) Create the project

```bash
oc apply -f project-request.yaml
```

### 2) Create low-priority deployment

To create a new deployment of 6 replicas (should take up a one node and overflow into the next one under the current configurations)

```bash
oc apply -f low-priority-pod.yaml
```

Head over to your openshift web portal, observability and try this prometheus query

```promql
kube_pod_container_resource_requests{namespace="phase-1", resource="memory"}
```

### 3) Create the high-priority deployment

```bash
oc apply -f high-priority-pod.yaml
```

it's set to 0 replicas by default.

### 4) Head over to the scenarios and try them
