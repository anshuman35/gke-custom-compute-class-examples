# GKE Custom Compute Class Examples

This repo contains some example configurations for use with Google Kubernetes Engine's custom compute class feature. Please see the [custom compute class documentation](https://cloud.google.com/kubernetes-engine/docs/concepts/about-custom-compute-classes) for latest references.

## Pre-requisites
The custom compute class feature works with both GKE Standard and Autopilot modes. It requires GKE version 1.30.3-gke.1451000 and later.

Eligible clusters automatically have the custom compute class CRD installed. No special flags are required at creation or update to use custom compute classes.

## Configure status.sh script
This script will output a list of node pools, nodes, and pods. Edit the script and update the two variables listed at the top:
* `CLUSTER_NAME`
* `LOCATION`

Set to executable:
```bash
chmod 750 status.sh
```

Note, it requires that you're authenticated to use `gcloud` and we're assuming you have kube config authorized for the cluster in question.

## machineFamily example
This folder contains a compute class and deployment which showcases the use of the `machineFamily:` priority rule type. This is a simple example that prioritizes `n2` over `n2d`, and requires a minimum of 4 core nodes. 

Deploy the class:

```bash
kubectl apply -f machineFamily/family-class.yaml
```

Deploy the workload which references this class:

```bash
kubectl apply -f machineFamily/family-deploy.yaml
```

Watch the nodes spin up and pods land:
```bash
watch ./status.sh
```

## machineType and Spot example
This example showcases the `machineType:` priority rule in combination with the `spot:` flag. The class prefers spot, but then falls back to on-demand instances. 

Deploy the class:

```bash
kubectl apply -f machineType/type-class.yaml
```

Deploy the workload which references this class:

```bash
kubectl apply -f machineType/type-deploy.yaml
```

Watch the nodes spin up and pods land:
```bash
watch ./status.sh
```

## Storage example
This example showcases storage options like bootdisk type, size, as well as using localSSD for ephemeral storage. 

Deploy the class:

```bash
kubectl apply -f storage/lssd-class.yaml
```

Deploy the workload which references this class:

```bash
kubectl apply -f storage/lssd-deploy.yaml
```

Watch the nodes spin up and pods land:
```bash
watch ./status.sh
```

You can  use `kubectl describe node <node name>` and look for this label: `cloud.google.com/gke-ephemeral-storage-local-ssd=true` to confirm your node is configured to use local SSD for ephemeral storage.

## GPU Accelerator example
See the README file in the `gpu-accelerator` directory