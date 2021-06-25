---
layout: post
title: "docker-compose使用代理"
categories: docker
author: "kekxv"
---

docker-compose使用代理

在构建时，可以制定参数，或者设置环境变量：

1. 通过指定构建参数：

```bash
docker-compose build \
--build-arg http_proxy=http://proxy.exaple.com \
--build-arg https_proxy=http://proxy.exaple.com
```

2. 通过设置环境变量：

```yaml
# docker-compose.yaml
build:
  context: .
args:
  - http_proxy=http://proxy.com
  - https_proxy=http://proxy.com
```

参考：[https://blog.k4nz.com/3a12c2cea6897a697bf55cb28e3caa61/](https://blog.k4nz.com/3a12c2cea6897a697bf55cb28e3caa61/)


原文参考文献
- [Docker Build Proxy: Setup the proxy for Dockerfile building – DEV](https://dev.to/zyfa/setup-the-proxy-for-dockerfile-building--4jc8)
- [Compose file version 3 reference | Docker Documentation](https://docs.docker.com/compose/compose-file/#args)
- [elasticsearch – docker-compose build and http_proxy – Stack Overflow](https://stackoverflow.com/questions/34990458/docker-compose-build-and-http-proxy)
