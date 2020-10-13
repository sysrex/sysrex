---
title: "K3s Raspberry pi"
date: 2019-11-24T14:16:40+01:00
tags: ["Kubernetes"]
summary: "Installing Kubernetes K3s on Raspberry Pi Cluster"
draft: false
---


For a while now I was after getting my own self hosted Kubernetes cluster, and with the Raspberry Pi it is now possible. This is a note to self in case I will have to replicate
What you need (Must Haves)

- Raspberry PI (2/3). Minimum of 2 (1 master and 1 worker)
- Power Cable (Micro USB, same as the old android chargers)
- Ethernet Cable (Can be done with wireless, but ethernet cable makes it easier)
- Micro SD Card (Minimum of 8gb)

What you want (Nice to have)

- Raspberry PI 3 Model b+. 4 (1 master and 5 worker)
- Gigabit Ethernet Switch.
- Short (1 ft) ethernet cable
- Short (1 ft) Power cable
- Multiport USB Charging HUB. (I used a 10 port anker hub)
- Cluster Case (Keeps everything nice and clean)
- Micro SD Card (32gb each)

Software Requirement

- Etcher
- Raspbian Buster Lite


7. You should see something similar. Go to Network Options > Hostname. And change the hostname to anything you want. I named mine k8s-master-1, k8s-node-1, k8s-node-2 and k8s-node-3.

8. Once you are done with that, you should set up Localization Options. Although it is optional for our use-case. It is just good practice. Also without it you won’t be able to enable wifi.

9. Once done the pi will reboot.

10. ssh in again. We will give our device a static ip so it keeps the ip between reboots.

cat >> /etc/dhcpcd.conf

Find your device IP. It will be in the form of x.x.x.x. You can find your device ip (on a mac) using ifconfig | grep inet . My mac ip was 192.168.0.26 so my ip format is like 192.168.0.x .

Paste the following code block

interface eth0
static ip_address=x.x.x.y/24
static routers=x.x.x.1
static domain_name_servers=1.1.1.1

Where x.x.x is same as your device ip and y is the ip you want. I put 100 for master and 101,102 and 103 for the rest. Reboot with sudo reboot

When the devices turn on they should reflect the new ips.

This concludes the initial setup. Next we do the general device setup.
Common Setup

Now we need to setup all the RPi's to make them ready to act as a Kubernetes node.

Installing docker:
{{<highlight shell>}}
curl -sSL get.docker.com | sh && \
sudo usermod pi -aG docker && \
newgrp docker
{{</highlight>}}

Disabling swap. Kubernetes requires swap to be disabled. Please note on Raspbian Buster you have to uninstall dphys.
{{<highlight bash>}}
sudo dphys-swapfile swapoff && \
sudo dphys-swapfile uninstall && \
sudo update-rc.d dphys-swapfile remove && \
sudo apt purge dphys-swapfile
{{</highlight>}}

We can check swap disable was a success by the following command returning empty

{{<highlight shell>}}
sudo swapon --summary
{{</highlight>}}

3. Next we edit the /boot/cmdline.txt file. And append the following in the end of the file. This needs to be in the same line as all the other text in the file. Do not create a new file.

{{<highlight shell>}}
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
{{</highlight>}}

4. Reboot with sudo reboot

5. SSH in again. Here we start deploying K3s with the Rancher image.


8. On the master and all the workers run the following command

sudo sysctl net.bridge.bridge-nf-call-iptables=1

Reasoning for this is explained better in this issue
Worker Node Setup

    From each worker node run

sudo kubeadm join --token <token> <master-node-ip>:6443 --discovery-token-ca-cert-hash sha256:<sha256>

2. After a few moment, run

kubectl get nodes

and you should see something like

{{<highlight bash>}}
NAME      STATUS   ROLES    AGE    VERSION

master    Ready    master   84d   v1.17.2+k3s1
worker6   Ready    <none>   84d   v1.17.2+k3s1
worker2   Ready    <none>   84d   v1.17.2+k3s1
worker1   Ready    <none>   84d   v1.17.2+k3s1
worker3   Ready    <none>   84d   v1.17.2+k3s1
worker5   Ready    <none>   84d   v1.17.2+k3s1
worker7   Ready    <none>   84d   v1.17.2+k3s1
worker4   Ready    <none>   84d   v1.17.2+k3s1

{{</highlight>}}

And with this the cluster setup is done.
Additional Steps

We can access our cluster now, which is awesome. But we can only do it while sshed in the master node. We should change that.

We can copy the config from the master to our local machine. Use scp . More info on this on this article.

scp pi@x.x.x.100:.kube/config .

Running the command from you local directory will copy the config file in that directory from master-node.

If you have kubectl installed in you local machine, you can then setup kubeconfig by either overriding the config file in $HOME/.kube/config or you can add the file on top of the existing config (if you have it) by

export KUBECONFIG=<location to config from pi>:$HOME/.kube/config

This way you can keep you kubeconfig and switch to rpi cluster config when needed. You can also look into adding the config in the file instead of overriding it. More info here.

This is a pretty decent cluster that one can use for learning purposes. With some volume mounted you can do some pretty cool stuff. I will try out some stuff in the upcoming days. Will also try to get the dashboard running.

Updates to follow.
