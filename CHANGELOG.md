# ceph-container
该项目从 https://github.com/ceph/ceph-container fork 而来，因为 ceph-container 默认的基础镜像均为 CentOS，在部分项目中，会因为海光的 CPU 不兼容而导致服务启动失败，故而在该项目的基础上增加 RockyLinux 的支持

> 海光的 CPU 会导致如下错误：
> segfault at 55db25b34320 ip 00007fe8640fcead sp 00007ffe8b235c68 error 4 in libc-2.28.so[7fe8640d8000+1cd000] likely on CPU 61 (core 29, socket 1)
> 重现方式：`docker run -ti --rm --entrypoint bash docker.io/rook/ceph:v1.13.7`，同时查看系统日志 `journalctl -f | grep kernel:`

- 复制 ./ceph-releases/ALL/centos 到 ./ceph-releases/ALL/rockylinux，以此为基础进行修改
- 修改 ./ceph-releases/ALL/rockylinux/9/daemon-base/__DOCKERFILR__INSTALL__，将部分 repo 地址替换为内部 Nexus 代理地址
- 修改 ./ceph-releases/ALL/rockylinux/9/daemon-base/__DOCKERFILR__INSTALL__，增加 entos-stream 仓库，避免 selinux-policy 因版本需求较高而下载失败
- 修改 Makefile，增加 http proxy 参数

> 构建脚本 `make build FLAVORS=reef,rockylinux,9`
> 构建结束后，会生成 ceph/demo、ceph/daemon 和 ceph/daemon-base 三个镜像文件，镜像文件之间的关系参考：https://docs.ceph.com/en/latest/install/containers/

构建结束后，将镜像文件 reTag 并推送到 harbor 仓库，命令如下：
```shell
docker tag ceph/daemon-base:main-reef-rockylinux-9-aarch64 harbor.dameng.io/ceph/daemon-base:main-reef-rockylinux-9-aarch64
docker push harbor.dameng.io/ceph/daemon-base:main-reef-rockylinux-9-aarch64

docker tag ceph/daemon:main-reef-rockylinux-9-aarch64 harbor.dameng.io/ceph/daemon:main-reef-rockylinux-9-aarch64
docker push harbor.dameng.io/ceph/daemon:main-reef-rockylinux-9-aarch64
```