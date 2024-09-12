# GKE Custom Compute Class Examples

This repo contains some example configurations for use with Google Kubernetes Engine's custom compute class feature. Please see the [custom compute class documentation](https://cloud.google.com/kubernetes-engine/docs/concepts/about-custom-compute-classes) for latest references.

## Watch a demo of Custom Compute Classes

[![Custom Compute Class Demo](https://img.youtube.com/vi/wxlE0FqeaNY/0.jpg)](https://www.youtube.com/watch?v=wxlE0FqeaNY)

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

## Statically defined node pool example
*Requires GKE Standard mode cluster* 

This example showcases priority rules featuring statically defined node pools (node pools you create, not NAP). This example also gives you a chance to see how fall-back priorities work by constraining the total number of nodes permissible in a given node pool. 

1. Edit the file `static-node-pools/screate-node-pools.sh` and add values for the `CLUSTER_NAME` and `LOCATION` variables. 

2. Create the node pools:

```bash
chmod 750 static-node-pools/*.sh
./static-node-pools/create-node-pools.sh
```

3. Deploy the custom compute class:
```bash
kubectl apply -f static-node-pools/sstatic-pools-class.yaml
```

4. Deploy the workload using the class:
```bash
kubectl apply -f static-node-pools/static-pools-deploy.yaml
```
In a separate shell instance, watch the pods, nodes, and node pools:
```
watch ./status.sh
```

This creates 10 replicas. Next, let's scale up the workload to see what happens when we hit the max nodes for`e2 spot` and fall back to `n2 spot`.

5. Scale up to 30 replicas:

```bash
 kubectl scale deployment test-workload --replicas 30
```

We now have 30 replicas with pods landing on the n2 nodes. Next, let's scale up the workload again to see what happens when we hit the max nodes for `n2 spot` and fall back to `n2d spot`.

6. Scale up to 100 replicas:

```bash
 kubectl scale deployment test-workload --replicas 100
```

7. To test active migration, we can now update the `e2-4-spot-pool` to max node size of 10. Edit the script to add your cluster name and location, then run:

```bash
./static-node-pools/update-e2-node-pool.sh
```

After a while, you'll see additional e2-standard-4 spot nodes being created and your workloads being migrated from n2 and n2d nodes.

8. Delete the workload and watch the scale down:
```bash
kubectl delete -f static-node-pools/static-pools-deploy.yaml
```

## GPU Accelerator example
See the README file in the `gpu-accelerator` directory