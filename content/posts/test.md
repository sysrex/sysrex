---
title: "Auto generate"
date: 2016-03-04T14:16:40+01:00
tags: ["AWS"]
summary: "How to add swap to an EC2 instance"
draft: false
---


### Run the following commands

```bash

sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
sudo /sbin/mkswap /var/swap.1
sudo chmod 600 /var/swap.1
sudo /sbin/swapon /var/swap.1

```


### And then add the line in your /etc/fstab

```bash

/var/swap.1   swap    swap    defaults        0   0

```


and that's all folks, now you should run *free -m* and see the results.