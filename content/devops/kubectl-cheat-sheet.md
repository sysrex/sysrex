---
title: "Kubectl Cheat Sheet"
date: 2021-08-12T09:12:40+01:00
tags: ["Kubernetes"]
summary: "Kubectl Cheat Sheet"
draft: false
---

alias k="kubectl"
complete -F __start_kubectl k


kubectl is self-documenting, which is another reason why it’s 100x better than a UI. If you’re confused about some command, just do k <command> --help. e.g. k label --help. There will be lots of examples in the help docs.
Minimize the use of imperative commands

There are many commands, like create, expose, scale, set image, rollout, etc. that I don’t use that much.

That’s because it’s better to deal with your Kubernetes configuration declaratively, and have a CI/CD system run the equivalent of kubectl apply -f ..

You should learn how to use these commands, but they shouldn’t be a regular part of your prod workflows. That will lead to a flaky system.


Basic Getting of Details

You can k get any Kubernetes API Resource.

```bash
$ k get pods
$ k get nodes
$ k get services
```

# You can sort too
```bash
$ k get pods --sort-by=.status.startTime
```

# Get with a label selector
```bash
$ k get pods -l app=kombucha-service
```


# Watch over time

```bash
$ k get pods -l app=kombucha-service --watch
```


# View the events
```bash
$ k get events
```

# Describe the pod
```bash
$ k describe pod <pod-name>
```

Describing the pod can help you figure out why it crashed. Perhaps you’re trying to inject a config map that doesn’t exist into its environment.

Furthermore, they often have abbreviations. To see what resources you can get and their abbreviations, use the api-resources command. This is helpful as well with custom resources.

```bash
$ k api-resources
NAME                              SHORTNAMES          APIGROUP                       NAMESPACED   KIND
bindings                                                                             true         Binding
componentstatuses                 cs                                                 false        ComponentStatus
configmaps                        cm                                                 true         ConfigMap
endpoints                         ep                                                 true         Endpoints
events                            ev                                                 true         Event
limitranges                       limits                                             true         LimitRange
.
.
.
csidrivers                                            storage.k8s.io                 false        CSIDriver
csinodes                                              storage.k8s.io                 false        CSINode
storageclasses                    sc                  storage.k8s.io                 false        StorageClass
volumeattachments                                     storage.k8s.io                 false        VolumeAttachment
```


If you’re unsure what some resource is, use k explain.

```bash
$ k explain deployment
KIND:     Deployment
VERSION:  apps/v1

DESCRIPTION:
Deployment enables declarative updates for Pods and ReplicaSets.

FIELDS:
apiVersion	<string>
APIVersion defines the versioned schema of this representation of an
object. Servers should convert recognized schemas to the latest internal
value, and may reject unrecognized values. More info:
<https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources>

kind	<string>
Kind is a string value representing the REST resource this object
represents. Servers may infer this from the endpoint the client submits
requests to. Cannot be updated. In CamelCase. More info:
<https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds>

metadata	<Object>
Standard object metadata.

spec	<Object>
Specification of the desired behavior of the Deployment.

status	<Object>
Most recently observed status of the Deployment.
```

Troubleshooting
Getting Logs

# Follow the logs of a pod
```
$ k logs -f <pod-name> -c <container-name>
```

# For only the tail, use `--tail=n`
```
$ k logs <pod-name> -c <container-name> --tail=50
```

# Multiple pods at once
```
$ k logs -l foo=bar
```

⭐ Debugging DNS and Services

This first tip is in the Debugging DNS Resolution article in the docs, but here is a quick imperative command to get the same result.

# Run a pod with DNS debugging utilities
$ k run dnsutils --image=gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 -- sleep 3600

# Run DNS commands from that pod
$ k exec -it dnsutils -- nslookup kubernetes.default
Server:		10.11.0.10
Address:	10.11.0.10#53

Name:	kubernetes.default.svc.cluster.local
Address: 10.11.0.1

If you’re wondering if you created a service correctly, you should also look into k get endpoints, and also double check the label selector. e.g.

$ k get svc alex-echo -o wide
NAME        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE    SELECTOR
alex-echo   ClusterIP   10.11.101.85   <none>        80/TCP    161d   app=alex-echo-server

$ k get pods -l app=alex-echo-server
NAME                                READY   STATUS    RESTARTS   AGE
alex-echo-server-7c47c5698c-zfhwg   1/1     Running   0          20h

What image is the pod running?

# What image is the pod running?
$ k get pod dnsutils -o jsonpath='{ .spec.containers[*].image }'
gcr.io/kubernetes-e2e-test-images/dnsutils:1.3

# Same thing, for a cron job
$ k get cj <cron-job-name> -o jsonpath='{ .spec.jobTemplate.spec.template.spec.containers[*].image }'

Decode and Copy a Secret

# Decode and copy a secret
$ k get secrets <secret-name> -o jsonpath='{ .data.<key> }' | base64 --decode | pbcopy

Metrics / Monitoring

# Is something hogging resources?
$ k top pods --all-namespaces --sort-by=memory
$ k top pods --all-namespaces --sort-by=cpu
$ k top nodes --sort-by=memory
$ k top nodes --sort-by=cpu

For more you can do with metrics, see here.
Executing commands on pods or nodes

# Execute a command on a pod
$ k exec -it nginx -- echo 'hello'
hello

# start a shell session
$ k exec -it nginx -- /bin/bash
root@nginx:/# echo hi!
hi!

# Starting a shell on a node is trickier
# I use the kubectl-node-shell (<https://github.com/kvaps/kubectl-node-shell>)
$ k node-shell <node> -- echo 123

Once you’ve started a shell session on a pod or node, curl, nslookup, ping, and env are useful.
What is the pod’s IP?

$ k get pod nginx -o=jsonpath='{ .status.podIPs[0].ip }'
10.11.151.203

Validate Kubernetes Resources

I’ve seen a lot of time wasted from people struggling to write YAML.

# Lint a Helm chart
# Good to put in pre-merge checks
$ helm template . | kubeval -

# Or just on a file
$ kubeval pod.yaml


Making Changes

Recall my note on not abusing the imperative commands. I’m serious about that.
Scale a Deployment or StatefulSet

$ k scale --replicas=n statefulset/<stateful-set-name>
$ k scale --replicas=n deployment/<deployment-name>

Restarting / Deleting Workflows

If a pod is not owned by a Deployment (really, a ReplicaSet), it will not come back after you delete it.

# Deleting pods owned by a deployment is safe — they will restart
$ k delete pod <deployment-pod-a>

# Same thing for Stateful Set pods
$ k delete pod <stateful-set-pod-0>

# Trigger a CronJob
$ k create job --from=cronjob/hello-world hello-world-now

# Now I want to clean up all my nginx pods that i've used for debugging
$ kubectl get pods -o custom-columns='NAME:metadata.name' --no-headers | grep nginx | xargs
nginx-alex nginx-bobby-cox nginx-freddie-freeman nginx-ozzie-albies nginx-ronald-acuna

# Use that now as a sub-command
$ k delete pod $(kubectl get pods -o custom-columns='NAME:metadata.name' --no-headers | grep nginx | xargs) --dry-run=client
pod "nginx-alex" deleted (dry run)
pod "nginx-bobby-cox" deleted (dry run)
pod "nginx-freddie-freeman" deleted (dry run)
pod "nginx-ozzie-albies" deleted (dry run)
pod "nginx-ronald-acuna" deleted (dry run)

Quickly Generate YAML

You can quickly generate a starting point for resource files with an imperative command + -dry-run=client -o yaml.

$ k run nginx --image=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: nginx
name: nginx
spec:
containers:
- image: nginx
  name: nginx
  resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  status: {}

# or to go straight to a file
$ k run nginx --image=nginx --dry-run=client -o yaml >> pod.yaml

This helped a lot on the CKA exam, because you basically just have kubectl and vim, as well as the Kubernetes docs. And there is definitely time pressure.
⭐ Run an echo server

$ k run echo-server --image=ealen/echo-server
pod/echo-server created

$ k port-forward echo-server :80
Forwarding from 127.0.0.1:60882 -> 80
Forwarding from [::1]:60882 -> 80
Handling connection for 60882

# Now I can hit that app locally
$ curl <http://localhost:60882> | jq .
{
"host": {
"hostname": "localhost",
"ip": "::ffff:127.0.0.1",
"ips": []
},
"http": {
"method": "GET",
"baseUrl": "",
"originalUrl": "/",
"protocol": "http"
},
"request": {
"params": {
"0": "/"
},
"query": {},
"cookies": {},
"body": {},
"headers": {
"host": "localhost:60882",
"user-agent": "curl/7.65.3",
"accept": "*/*"
}
},
"environment": {} # hiding this output
}


Plugins

Note - install any of these with krew.

$ kubectl krew install ctx
$ kubectl krew install ns

Kubectl has some awesome plugins. For example, I hate typing

$ k config use-context <context-name>

so I use the kubectx plugin. And I do

# select a context
$ k ctx <context-name>

# rename a context
$ k ctx atomic-commits=gke_atomic-commits_us-east1-c_r6ofyul
Context "gke_atomic-commits_us-east1-c_r6ofyul" renamed to "atomic-commits".

# delete a context
$ k ctx -d atomic-commits

I also like

    kubens - Same as kubectx, but for namespaces.
    ⭐ ksniff - Use tcpdump and Wireshark to start a remote capture on any pod. This is gold for debugging socket hangups.
    kubectl-node-shell - Run a shell on a Kubernetes node. Note that there are a lot of privileged things you won’t be able to do. For that you have to use the plugin to run a privileged container.
    kubecolor - I like having colorized output. I just make the alias k=kubecolor in my .zshrc to use this instead.

Additional Thoughts

    To check user privileges, look into k auth can-i.
    You can see all CRDs with k get crds.
    Set the default namespace with k config set-context --current --namespace=foo
    Sometimes when going between gcloud users and different Kubernetes contexts, kubectl will cache the wrong token. It can be helpful to automatically expire all of your users and force kubectl to re-authenticate. You do that like this. (You need yq installed.)

yq eval -i '.users[].user.auth-provider.config.expiry = "2020-01-01T12:00"' ~/.kube/config
