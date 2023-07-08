---
title: "Jenkins on debian 10"
date: 2020-06-11T09:16:40+01:00
tags: ["AWS"]
summary: "Install Jenkins on debian 10 buster"
draft: false
---

### Update packages and then install the default jdk 

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install default-jdk
```


### We're here using wget to pull the key for the debian repository. This command will return OK.

```bash
wget -qO - https://pkg.jenkins.io/debian-stable/jenkins.io.key | apt-key add -
```


### Add the app debian repository

```bash
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```

### Update packages cache and then proceed with the installation.

```bash
sudo apt update
sudo apt install jenkins
```


### Enable jenkins to start after reboot

```bash
sudo systemctl enable --now jenkins
```