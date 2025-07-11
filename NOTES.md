# Changes

## Global Changes

1. This commit change all ELB to internal only. Either change to `internet-facing` for testing or do port forward eg.

```sh
kubectl port-forward -n jupyterhub svc/proxy-public 8080:80
# DreamBooth Inference API
kubectl port-forward -n dogbooth svc/dogbooth-raycluster-zdpxm-head-svc 8000:8000
# Ray Dashboard
kubectl port-forward -n dogbooth svc/dogbooth-raycluster-zdpxm-head-svc 8265:8265
```

## Jark Stack

1. Jark Stack Companion Notes: [Deploy Generative AI Models on Amazon EKS | Containers](https://aws.amazon.com/blogs/containers/deploy-generative-ai-models-on-amazon-eks/)
2. Updated the syntax for `ray-service.yaml`

### How it works

The cluster is configured to automatically provision G5 instances for JupyterHub users through several key components working together.

`jupyterhub-values.yaml` configures each user pod to request GPU resources:

```
singleuser:
  extraResource:
    limits:
      nvidia.com/gpu: "1"  # Each user pod requests 1 GPU
  memory:
    guarantee: 24G          # Requires 24GB memory
  extraTolerations:
    - key: nvidia.com/gpu   # Can be scheduled on GPU nodes
      operator: Exists
      effect: NoSchedule
```

When a user logs into JupyterHub:

1. User Login → JupyterHub creates a pod named jupyter-user1
2. Pod Scheduling → Pod requests nvidia.com/gpu: "1" + 24GB memory
3. Karpenter Detection → No existing nodes can satisfy GPU + memory requirements
4. Instance Selection → Karpenter chooses G5 instance types (they have GPUs + enough memory)
5. Node Provisioning → New G5 instance is launched and joins the cluster
6. Pod Placement → jupyter-user1 pod is scheduled on the new G5 node
