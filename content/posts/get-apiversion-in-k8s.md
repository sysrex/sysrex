---
title: "Get the apiVersion in Kubernetes"
date: 2020-06-12T19:19:40+01:00
tags: ["kubernetes"]
summary: "How to get the apiVersion in Kubernetes"
draft: false
---

Everytime I have to create a manifest I stumble onto identifying which resource I have to do for `apiVersion`. For some of them things I could probably figure it out but I'd rather not just guess it.


The format is `api_group/version` (unless you are working with resource in the core API group, in which case you omit the `api_group` portion). 

## Let's find the api group

We will run `kubectl api-resources`:

```bash
➜  ~ kubectl api-resources
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
```

*Output truncated to avoid the really long output*

In the third column, `APIGROUP`. If, for example, you're creating a manifest for a new `Deployment` you could `grep` the output of `kubectl api-resources` and see that the API group is `apps`.

## GLet's find the API group version

Because we now know the `APIGROUP` we will just get the version by using grep.

```bash
$ kubectl api-versions | grep -E "^apps/"
apps/v1
```

The result is the version we need.

## Result

We are now ready to put the `apiVersion` for the `Deployment` in the cluster.

```yaml
kind: Deployment
apiVersion: apps/v1
...
```
No more guessing the API version!
