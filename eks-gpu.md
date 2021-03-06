# Create Amazon EKS cluster with GPU-enabled workers

This document explains how to create an Amazon EKS cluster with GPU-enabled workers.

1. Create EKS cluster with GPU nodes:

   ```
   eksctl create cluster eks-gpu --node-type=p3.8xlarge --timeout=40m
   ```

   By default, ssh access is not enabled. Here is how it can be done:

   ```
   eksctl create cluster eks-gpu \
      --node-type=p3.8xlarge \
      --timeout=40m \
      --ssh-access \
      --ssh-public-key arun-us-west2
   ```
   
1. Apply NVIDIA driver:

   ```
   kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.10/nvidia-device-plugin.yml
   ```

   This will be simplified after https://github.com/weaveworks/eksctl/issues/205 is fixed.

1. Get memory, CPU and GPU for each node in the cluster:

   ```
   kubectl get nodes "-o=custom-columns=NAME:.metadata.name,MEMORY:.status.allocatable.memory,CPU:.status.allocatable.cpu,GPU:.status.allocatable.nvidia\.com/gpu"
   ```

   Shows something like:

   ```
   NAME                                            MEMORY        CPU       GPU
   ip-192-168-101-177.us-west-2.compute.internal   251643680Ki   32        4
   ip-192-168-196-254.us-west-2.compute.internal   251643680Ki   32        4
   ```

   The maximum number of GPUs that may be scheduled to a pod is capped by the number of GPUs available per node. By default, pods are scheduled on CPU. 

