---
title: "Deploy Apache Flink on AWS EKS"
date: 2020-08-15T12:31:40+01:00
tags: ["AWS", "EKS"]
summary: "How to deploy Apache Flink on AWS EKS with ALB ingress"
draft: false
---

First thing is first, we will use the `eksctl` tool to create the EKS Cluster 

```yaml

---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: eks-flink-demo
  region: eu-west-2
# traits of worker nodes
nodeGroups:
  - name: eks-flink-workers
    instanceType: m5.large
    desiredCapacity: 5
    minSize: 3
    maxSize: 15
    iam:
      # polices added to worker node role
      withAddonPolicies:
        # allows read/write to zones in Route53
        externalDNS: true
        # required for ALB-ingress
        albIngress: true
```

```bash
eksctl create cluster \
  --configfile eks-flink-demo.yaml \
  --kubeconfig eks_flink_demo_kubeconfig.yaml
```
