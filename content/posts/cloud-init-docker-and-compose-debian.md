---
title: "Cloud init script to install docker and docker compose debian 10"
date: 2020-10-13T11:42:40+01:00
tags: ["configs"]
summary: "Cloud init script to install docker and docker compose on debian 10"
draft: false
---

```yaml
#cloud-config
groups:
  - docker
users:
  - default
  # the docker service account
  - name: docker-service
    groups: docker
package_upgrade: true
packages:
  - apt-transport-https
  - ca-certificates
  - curl
  - gnupg-agent
  - software-properties-common
runcmd:
  # install docker following the guide: https://docs.docker.com/install/linux/docker-ce/debian/
  - curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
  - sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
  - sudo apt-get -y update
  - sudo apt-get -y install docker-ce docker-ce-cli containerd.io
  - sudo systemctl enable docker
  # install docker-compose following the guide: https://docs.docker.com/compose/install/
  - sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  - sudo chmod +x /usr/local/bin/docker-compose
power_state:
  mode: reboot
  message: Restarting after installing docker & docker-compose
```