---
title: GPU configuration
keywords: HOBBIT Documentation
sidebar: main_sidebar
permalink: gpu.html
folder: docs
---

[Install NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#setting-up-nvidia-container-toolkit).

Identify UUIDs of GPUs by running `nvidia-smi -a`.

Configure [`/etc/docker/daemon.json`](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file):

```
{
    "default-runtime": "nvidia",
    "node-generic-resources": ["NVIDIA-GPU=UUID1", "NVIDIA-GPU=UUID2"],
```

Restart the Docker: `sudo systemctl restart docker`.

Test it with a Docker container: `sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi`.

Test it with a Docker service: `sudo docker service create --restart-condition none --name test nvidia/cuda:11.0-base nvidia-smi`, then `docker service logs test`.
