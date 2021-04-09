---
title: "Deploy Apache Flink on AWS EKS"
date: 2020-08-15T12:31:40+01:00
tags: ["AWS", "EKS", "Big Data"]
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

Once the cluster is up and running we will need to start adding the required dependencies:

1. First lets add the oidc integration for the service account used by the controller
```bash
eksctl --profile sysrex  utils associate-iam-oidc-provider --cluster eks-flink-demo --approve --region=eu-west-2
```

2. We will then create an IAM policy that will give access to the pod with the ALB Ingress Controller
to create and manage the ALB
```bash
aws --profile sysrex iam create-policy --policy-name ALBIngressControllerIAMPolicy --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json --region=eu-west-2
```
If this fails for you as it did for me a couple of times, I just:
```bash
wget https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json
```
and then run it from local like this:
```bash
aws --profile sysrex iam create-policy --policy-name ALBIngressControllerIAMPolicy --policy-document file://iam-policy.json --region=eu-west-2
```

3. And now we create the ingress role, you will need the ARN of the IAM policy created in the previous step

```bash
eksctl --profile sysrex create iamserviceaccount --name alb-ingress-controller --namespace kube-system --override-existing-serviceaccounts --approve --cluster eks-alb-testing --attach-policy-arn arn:aws:iam::*********:policy/ALBIngressControllerIAMPolicy --region=eu-west-2
```

4. Create the ALB Service Ingress Account 

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
```
you can check if the account has been created with:

```bash
kubectl -n kube-system get serviceaccounts | grep alb
```

5. Start the ALB Ingress Controller

I usually get the example and edit it to fit my needs:

```bash
wget https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml
```

Inside you will need the following added under spec:

```yaml
...
    spec:
      containers:
        - name: alb-ingress-controller
          args:
```

- --cluster-name=eks-flink-demo
-  --aws-vpc-id=vpc-xxxxxxxxxx
-  --aws-region=eu-west-2


now, let's apply it:

```bash
kubectl apply -f alb-ingress-controller.yaml
```

and get the logs:

```bash
kubectl logs -n kube-system deployment.apps/alb-ingress-controller
```

6. Now for the Flink Deployment itself:

we will apply all the configurations pulled from their website at [Apache Flink](https://ci.apache.org/projects/flink/flink-docs-release-1.11/ops/deployment/kubernetes.html)

```bash
flink-configuration-configmap.yaml
jobmanager-rest-service.yaml
jobmanager-service.yaml
jobmanager-session-deployment.yaml
task-manager-session-deployment.yaml
taskmanager-query-state-service.yaml
```

after this is done you should be able to check for the deployments:

```bash
➜  flink kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
flink-jobmanager    1/1     1            1           20h
flink-taskmanager   2/2     2            2           20h
```

and for the services:

```bash
➜  flink kubectl get svc
NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP                       PORT(S)                                        AGE
flink-jobmanager                LoadBalancer   172.20.96.2      <none>                            6123:30750/TCP,6124:30768/TCP,8081:32587/TCP   16h
flink-jobmanager-rest           NodePort       172.20.186.224   <none>                            8081:30081/TCP                                 16h
flink-taskmanager-query-state   NodePort       172.20.178.170   <none>                            6125:30025/TCP                                 16h
kubernetes                      ClusterIP      172.20.0.1       <none>                            443/TCP                                        17h
```

You will notice currently there is no ALB set, so we will modify the `jobmanager-service.yaml` and set it to be


```yaml
spec:
  type: LoadBalancer
```


and that's all folks, run `kubectl get svc` and you will see the endpoint of the ALB created.