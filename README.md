

- ## Basic setups for different platforms (not production ready - useful for personal use)

- [`Prometheus / Grafana / alertmanager / prometheus-webhook-dingtalk`](https://github.com/xiaozhenkai/awesome-compose/tree/main/prometheus-grafana-alertmanager-dingtalk)



## Getting started

These instructions will get you through the bootstrap phase of creating and deploying samples of containerized applications with Docker Compose.

### Prerequisites

- Make sure that you have Docker and Docker Compose installed
  - Windows or macOS: [Install Docker Desktop](https://www.docker.com/get-started)
  - Linux: [Install Docker](https://www.docker.com/get-started) and then [Docker Compose](https://github.com/docker/compose)
- Download some or all of the samples from this repository.

### Running a sample

The root directory of each sample contains the `docker-compose.yaml` which describes the configuration of service components. All samples can be run in a local environment by going into the root directory of each one and executing:

```
docker-compose up -d
```

Check the `README.md` of each sample to get more details on the structure and what is the expected output. To stop and remove all containers of the sample application run:

```
docker-compose down
```