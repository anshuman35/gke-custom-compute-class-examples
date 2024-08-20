# TGI Gemma model using custom compute class

1. Follow the general instructions in the [Google Cloud TGI Gemma tutorial](https://cloud.google.com/kubernetes-engine/docs/tutorials/serve-gemma-gpu-tgi) through HF credentials section (stop at Deploy TGI)

2. Deploy the custom compute class:

```bash
kubectl apply -f accelerator-class.yaml
```

3. On GKE Standard, you may need to [manually install the NVIDIA device drivers](https://cloud.google.com/kubernetes-engine/docs/how-to/gpus#installing_drivers) on your cluster

4. Deploy `gemma-2b-deploy.yaml` or `gemma-7b-deploy.yaml` (instead of using the manifests in the [Deploy TGI](https://cloud.google.com/kubernetes-engine/docs/tutorials/serve-gemma-gpu-tgi#deploy-tgi) section of the tutorial):

```bash
kubectl apply -f gemma-2b-deploy.yaml
```
or

```bash
kubectl apply -f gemma-7b-deploy.yaml
```

5. Continue with the rest of the tutorial