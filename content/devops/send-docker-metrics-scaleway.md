---
title: "Sending Docker Metrics to Scaleway Cockpit Loki with Prometheus"
date: 2023-03-04T14:16:40+01:00
tags: ["DevOps"]
summary: "How to send docker metrics to Scaleway Cockpit Loki with Prometheus"
draft: false
---


Before starting *NB* you need to enable cockpit and create an API Token that has permissions to send logs and metrics.

To enable logging for specific Docker containers and visualize the logs in Grafana using Docker Compose, follow these steps:

1. Create your Docker Compose file and add the service for Grafana Prometheus.

```yaml
version: '3.8'

services:
  nginx-app:
    container_name: nginx-app
    image: nginx
    labels:
      logging: "promtail"
      logging_jobname: "containerlogs"
    ports:
      - 8080:80
    networks:
      - app
  promtail:
    image:  grafana/promtail:latest
    container_name: promtail
    volumes:
      - ./config/promtail.yaml:/etc/promtail/docker-config.yaml
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock
    command: -config.file=/etc/promtail/docker-config.yaml
    networks:
      - app
  agent:
    image: grafana/agent:latest
    restart: always
    volumes:
      - "./prometheus/config.yaml:/etc/agent-config/agent.yaml:ro"
      - "/:/host/root:ro"
      - "/sys:/host/sys:ro"
      - "/proc:/host/proc:ro"
      - "/tmp/agent:/etc/agent"
    entrypoint:
      - /bin/grafana-agent
      - -config.file=/etc/agent-config/agent.yaml
      - -metrics.wal-directory=/tmp/agent/wal
    network_mode: "host"
    pid: "host"
  promtail:
    image:  grafana/promtail:latest
    container_name: promtail
    volumes:
      - ./promtail/promtail.yaml:/etc/promtail/docker-config.yaml
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock
    command: -config.file=/etc/promtail/docker-config.yaml
    networks:
      - app

networks:
  app:
    name: app
```

2. Create your prometheus configuration file in `prometheus/config.yaml`

```yaml
metrics:
  wal_directory: /tmp/agent
  global:
    scrape_interval: 60s
    remote_write:
      - url: https://metrics.cockpit.fr-par.scw.cloud/api/v1/push
        headers:
          "X-Token": xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  configs:
    - name: tm-agent
      scrape_configs:
        - job_name: tendermint
          static_configs:
            - targets: ['127.0.0.1:26660']

integrations:
  node_exporter:
    enabled: true
    rootfs_path: /host/root
    sysfs_path: /host/sys
    procfs_path: /host/proc
```

*Enjoy!*
