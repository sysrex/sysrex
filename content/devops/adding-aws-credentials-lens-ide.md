---
title: "Adding aws credentials to lens ide"
date: 2020-01-15T20:11:30+01:00
tags: ["AWS", "Kubernetes"]
summary: "How to add aws credentials to lens ide"
draft: false
---

I am managing several K8s clusters and I regularly switch between them, in order to keep connected on all of them at the same time I am adding the kubeconfig with env variables to avoid switching profiles in [LensApp](https://k8slens.dev)
### Run the following commands 

```yaml

env:
    - name: AWS_ACCESS_KEY_ID
      value: my-key-id
    - name: AWS_SECRET_ACCESS_KEY
      value: my-key-secret

```

Add the above code block and paste the kubeconfig as text into [LensApp](https://k8slens.dev).