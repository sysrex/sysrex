---
title: "Adding swap to ec2"
date: 2016-03-04T14:16:40+01:00
tags: ["AWS"]
summary: "How to add swap to an EC2 instance"
draft: false
---


### Run the following commands 

{{<highlight bash>}}

sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
sudo /sbin/mkswap /var/swap.1
sudo chmod 600 /var/swap.1
sudo /sbin/swapon /var/swap.1

{{</highlight>}}



### And then add the line in your /etc/fstab

{{<highlight bash>}}

/var/swap.1   swap    swap    defaults        0   0

{{</highlight >}}


and that's all folks, now you should run *free -m* and see the results.
