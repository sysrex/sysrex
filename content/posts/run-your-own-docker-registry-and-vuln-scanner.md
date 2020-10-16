---
title: "Running your own docker registry"
date: 2020-10-16T11:57:40+01:00
tags: ["docker", "scanner"]
summary: "How to run your own docker registry with clair vulnerability scanner"
draft: false
---


For a while now I wanted to run my own docker registry so when I stumbled across  [Jessie Frazelle's](https://r.j3ss.co)...
I knew that's what I wanted to build.


So here's my unpolished docker-compose file (I still have to review it for security and probably add nginx as container)


```yaml

version: '3'
services:

  postgres:
    container_name: clair_postgres
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_PASSWORD: "Password!"
      POSTGRES_USER: "clair"
      POSTGRES_DB: "clair"
    volumes:
      - /mnt/HC_Volume_7546280/data/postgres/:/var/lib/postgresql/data:rw

  clair:
    container_name: clair
    image: quay.io/coreos/clair:latest
    ports:
    - "6060:6060"
    restart: always
    depends_on:
      - postgres
    volumes:
      - /tmp:/tmp
      - /mnt/HC_Volume_7546280/data/clair_config/:/config/:ro
    command: [--log-level=debug, --config, /config/config.yaml]

  registry:
    restart: always
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - /mnt/HC_Volume_7546280/data:/data

  reg:
    restart: always
    depends_on:
      - registry
    image: r.viaops.com/reg:lastest
    ports:
    - "8080:8080"
    command: "server --clair http://r.viaops.com:6060 -r r.viaops.com"
```

