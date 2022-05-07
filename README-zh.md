# 部署

## 问题

fluent-bit 默认使用 jemalloc, ARM 镜像因为 CentOS 7 使用 64KB page size 会启动失败 [issue](https://github.com/fluent/fluent-bit/issues/4270).
解决方式是自己 build arm 镜像，步骤如下:

- git clone <https://github.com/fluent/fluent-bit> && cd fluent-bit && git checkout v1.9.3
- [set jemalloc=Off](https://github.com/fluent/fluent-bit/blob/v1.9.3/dockerfiles/Dockerfile#L62)
- cp dockerfiles/Dockerfile .
- docker buildx build --platform "linux/arm64" -t ${your_tag}-arm64:v1.9.3 --target=production . --push

然后推送 multi-arch 镜像到自己的 registry

```sh
docker pull --platform linux/amd64 fluent/fluent-bit:1.9.3 && docker tag fluent/fluent-bit:1.9.3 ${your_tag}-amd64:v1.9.3
docker push ${your_tag}-amd64:v1.9.3
docker push ${your_tag}-arm64:v1.9.3
docker manifest create ${your_tag}:v1.9.3 --amend ${your_tag}-amd64:v1.9.3 --amend ${your_tag}-arm64:v1.9.3
docker manifest push ${your_tag}:v1.9.3
```
