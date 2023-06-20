# Multi zone setup

This works much like the single-zone setup with one plus and "1 con"

- ➕ It runs on multiple AZ! that means that if you have 3 AZ in your cluster, one benchwarming pod will keep one benchwarming node per AZ
- ➖ You can have one and only one per AZ as of now.

To get it up and running:

## Setup

### 1) Create the project and change to it

Let's do our testing in a separate namespace

```bash
oc apply -f project-request-mz.yaml
oc project benchwarming-multizone-autoscaling
```

### 2) Ensure your machinepool on `rosa` is set to autoscale

```bash
rosa login
rosa list machinepool --cluster=<myClusterId>
```

If it's not set to autoscale, you can do it like so, let's set it from 3 to 6 nodes.

```bash
rosa edit machinepool --enable-autoscaling --min-replicas=3 --max-replicas=6 --cluster=<myClusterId> <myMachinePoolId>
```

### 2) Create the benchwarming pod

It's been created as a deployment in case you want to play around with scaling, but it only runs a single pod unless specified in a scenario

```bash
oc apply -f benchwarmer-mz-deployment.yaml
```

### 3) Create the high-priority deployment

```bash
oc apply -f high-priority-mz-deployment.yaml
```

it's set to 3 replicas by default.
