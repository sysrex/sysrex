---
title: "Jenkins on debian 10"
date: 2020-06-11T09:16:40+01:00
tags: ["AWS"]
summary: "Install Jenkins on debian 10 buster"
draft: false
---

### Update packages and then install the default jdk 

{{<highlight bash>}}

sudo apt update && sudo apt upgrade -y
sudo apt install default-jdk

{{</highlight>}}



### We're here using wget to pull the key for the debian repository. This command will return OK.

{{<highlight bash>}}

wget -qO - https://pkg.jenkins.io/debian-stable/jenkins.io.key | apt-key add -

{{</highlight >}}


### Add the app debian repository

{{<highlight bash>}}

sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

{{</highlight >}}


### Update packages cache and then proceed with the installation.

{{<highlight bash>}}

sudo apt update
sudo apt install jenkins

{{</highlight >}}


### Enable jenkins to start after reboot

{{<highlight bash>}}

sudo systemctl enable --now jenkins

{{</highlight >}}