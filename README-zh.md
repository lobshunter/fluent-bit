## 部署
### 问题
fluent-bit 1.8.9 有两个坑, 通过 custom build 解决
1. fluent-bit HTTP server 会意外退出 [issue](https://github.com/monkey/monkey/issues/356)

  解决： `docker build -t ${your_tag}-amd64:v1.8.9 -f dockerfiles/Dockerfile.x86_64-master ..`
        docker build 里有 build 操作，实际是 fixed by [this commit](https://github.com/lobshunter/fluent-bit/commit/4591f4c2bed45edeef9a435692a445eb07cf2f45).

2. ARM 镜像在 CentOS 7 节点部署失败 [issue](https://github.com/fluent/fluent-bit/issues/4270), 解决方式如下
  解决：在 ARM-CentOS 机器上 build(build 方式见下节), 然后以下 Dockerfile 覆写官方镜像的 fluent-bit binary
    `docker build -t ${your_tag}-arm64:v1.8.9 -f $Dockerfile .`
    ```Dockerfile
    FROM fluent/fluent-bit:v1.8.9
    COPY ./build/bin/fluent-bit /fluent-bit/bin/fluent-bit
    ```
    
**最后**, push 镜像并支持 multi-arch
```
docker push ${your_tag}-amd64:v1.8.9
docker push ${your_tag}-arm64:v1.8.9
docker manifest create ${your_tag}:v1.8.9 --amend ${your_tag}-amd64:v1.8.9 --amend ${your_tag}-arm64:v1.8.9
docker manifest push ${your_tag}:v1.8.9
```


## 如何 build
参考[官方文档](https://docs.fluentbit.io/manual/installation/sources/build-and-install)
即执行 `cd build && cmake .. && make`

`cmake` 可以调整 build 参数与官方镜像[对齐](https://github.com/fluent/fluent-bit/blob/9ccc6f89171d6792f9605b77b4efd27bc28f472a/dockerfiles/Dockerfile.x86_64?_pjax=%23js-repo-pjax-container%2C%20div%5Bitemtype%3D%22http%3A%2F%2Fschema.org%2FSoftwareSourceCode%22%5D%20main%2C%20%5Bdata-pjax-container%5D#L40-L49)
