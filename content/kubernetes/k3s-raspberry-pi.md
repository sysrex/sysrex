---
title: "K3s Raspberry pi"
date: 2021-04-12T09:16:40+01:00
tags: ["Kubernetes"]
summary: "Installing Kubernetes K3s on Raspberry Pi Cluster"
draft: false
---


For a while now I was after getting my own self hosted Kubernetes cluster, and with the Raspberry Pi it is now possible. This is a note to self in case I will have to replicate

My Bill of materials

- Gigabyte Brix J1900 for the master node
- Raspberry PI 3 Model b+. for the 8 workers
- Gigabit Ethernet Switch.
- Multiport USB Charging HUB. (I used a 10 port anker hub)
- Cluster Case (Keeps everything nice and clean)
- Micro SD Card (128gb each)


After installing Raspi OS, I always expand the filesystem, set the hostname and add the folloring at the end of `/boot/cmdline.txt`

```bash
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

Then I just reboot and then use k3sup to prepare  my cluster.
It is important to mention, that you need to have your ssh key in the nodes and setup passwordless sudo for the user pi.

From my machine I use the k3sup binary and I run on the master node:

```bash

export IP=192.168.0.100
k3sup install --ip $IP --user pi

```

Then on the worker nodes:

```bash

export SERVER_IP=192.168.0.100
export USER=pi

k3sup install --ip $SERVER_IP --user $USER

```


See it in action

```bash
➜ ~ kubectl get nodes
NAME      STATUS   ROLES    AGE   VERSION
master    Ready    master   15h   v1.19.9+k3s1
worker0   Ready    <none>   11h   v1.19.9+k3s1
worker4   Ready    <none>   11h   v1.19.9+k3s1
worker7   Ready    <none>   11h   v1.19.9+k3s1
worker1   Ready    <none>   11h   v1.19.9+k3s1
worker3   Ready    <none>   11h   v1.19.9+k3s1
worker2   Ready    <none>   11h   v1.19.9+k3s1
worker5   Ready    <none>   11h   v1.19.9+k3s1
worker6   Ready    <none>   11h   v1.19.9+k3s1
```
Have fun!