---
title: "Sending Docker Logs to Scaleway Cockpit Loki with Promtail"
date: 2022-03-04T14:16:40+01:00
tags: ["DevOps"]
summary: "How to send docker logs to Scaleway Cockpit Loki with Promtail"
draft: false
---


Before starting *NB* you need to enable cockpit and create an API Token that has permissions to send logs and metrics.

To enable logging for specific Docker containers and visualize the logs in Grafana using Docker Compose, follow these steps:

1. Create your Docker Compose file and add the service for Grafana Promtail.

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

networks:
  app:
    name: app
```

2. Create your promtail configuration file in `config/promtail.yaml`

```yaml
# https://grafana.com/docs/loki/latest/clients/promtail/configuration/
# https://docs.docker.com/engine/api/v1.41/#operation/ContainerList
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://api_key:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@https://logs.cockpit.fr-par.scw.cloud/loki/api/v1/push/loki/api/v1/push

scrape_configs:
  - job_name: flog_scrape
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
        filters:
          - name: label
            values: ["logging=promtail"]
    relabel_configs:
      - source_labels: ['__meta_docker_container_name']
        regex: '/(.*)'
        target_label: 'container'
      - source_labels: ['__meta_docker_container_log_stream']
        target_label: 'logstream'
      - source_labels: ['__meta_docker_container_label_logging_jobname']
        target_label: 'job'
```

3. Add the logging=promtail label to containers that need to be enabled for logging. Configure Grafana Promtail to scrape logs from containers with the logging=promtail label.


*Enjoy!*