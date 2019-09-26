## Docker磁盘空间使用分析与清理

用了一段时间Docker后，会发现它占用了不少硬盘空间。还好Docker 1.13引入了解决方法，它提供了简单的命令System来查看/清理Docker使用的磁盘空间。

## 前情提要
```shell
#MyVersion
[root@dockercon ~]# docker version
Client:
 Version:   17.12.0-ce
 API version:   1.35
 Go version:    go1.9.2
 Git commit:    c97c6d6
 Built: Wed Dec 27 20:10:14 2017
 OS/Arch:   linux/amd64
Server:
 Engine:
  Version:  17.12.0-ce
  API version:  1.35 (minimum version 1.12)
  Go version:   go1.9.2
  Git commit:   c97c6d6
  Built:    Wed Dec 27 20:12:46 2017
  OS/Arch:  linux/amd64
  Experimental: false
  ```
Docker 的内置 CLI 指令docker system df，可用于查询镜像（Images）、容器（Containers）和本地卷（Local Volumes）等空间使用大户的空间占用情况。
```shell
[root@dockercon ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
kalilinux/kali-linux-docker   latest              c927a54ec8a4        8 days ago          1.88GB
nginx                         latest              3f8a4339aadd        9 days ago          108MB
busybox                       latest              6ad733544a63        2 months ago        1.13MB
[root@dockercon ~]# docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              3                   0                   1.994GB             1.994GB (100%)
Containers          0                   0                   0B                  0B
Local Volumes       0                   0                   0B                  0B
Build Cache                                                 0B                  0B
```
可以进一步通过-v参数查看空间占用细节
```shell
[root@dockercon ~]# docker system df -v
#镜像空间使用情况
Images space usage:

REPOSITORY                    TAG                 IMAGE ID            CREATED ago         SIZE                SHARED SIZE         UNIQUE SiZE         CONTAINERS
kalilinux/kali-linux-docker   latest              c927a54ec8a4        8 days ago ago      1.884GB             0B                  1.884GB             0
nginx                         latest              3f8a4339aadd        9 days ago ago      108.5MB             0B                  108.5MB             0
busybox                       latest              6ad733544a63        2 months ago ago    1.129MB             0B                  1.129MB             0

#容器空间使用情况
Containers space usage:

CONTAINER ID        IMAGE               COMMAND             LOCAL VOLUMES       SIZE                CREATED ago         STATUS              NAMES

#本地卷使用情况
Local Volumes space usage:

VOLUME NAME         LINKS               SIZE

Build cache usage: 0B
```
## 空间清理
|不同状态 |	已使用镜像（used image）|	未引用镜像（unreferenced image）|	悬空镜像（dangling image）
:---:|:---:|:---:|:---:|:---
镜像含义|	指所有已被容器（包括已停止的）关联的镜像。|	没有被分配或使用在容器中的镜像	|未配置任何 Tag （也就无法被引用）的镜像

### Docker内置自动清理：

通过 Docker 内置的 CLI 指令docker system prune来进行自动空间清理。
```shell
[root@dockercon ~]# docker system prune --help

Usage:  docker system prune [OPTIONS]

Remove unused data

Options:
  -a, --all             Remove all unused images not just dangling ones
      --filter filter   Provide filter values (e.g. 'label=<key>=<value>')
  -f, --force           Do not prompt for confirmation
      --volumes         Prune volumes
```
docker system prune 自动清理说明：

* 该指令默认会清除所有如下资源：

    + 已停止的容器（container）
    + 未被任何容器所使用的卷（volume）
    + 未被任何容器所关联的网络（network）
    + 所有悬空镜像（image）。
* 该指令默认只会清除悬空镜像，未被使用的镜像不会被删除。添加-a 或 --all参数后，可以一并清除所有未使用的镜像和悬空镜像。

* 可以添加-f 或 --force参数用以忽略相关告警确认信息。
```shell
[root@dockercon ~]# docker system prune --help

Usage:  docker system prune [OPTIONS]

Remove unused data

Options:
  -a, --all             Remove all unused images not just dangling ones
      --filter filter   Provide filter values (e.g. 'label=<key>=<value>')
  -f, --force           Do not prompt for confirmation
      --volumes         Prune volumes
[root@dockercon ~]# docker system prune --all
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all images without at least one container associated to them
        - all build cache
Are you sure you want to continue? [y/N] y
Deleted Containers:
f095899e7343e160d5b32d0688a6561a1a7f6af91c42ffe966649240b58ca23f

Deleted Images:
untagged: busybox:latest
untagged: busybox@sha256:e3789c406237e25d6139035a17981be5f1ccdae9c392d1623a02d31621a12bcc
deleted: sha256:6ad733544a6317992a6fac4eb19fe1df577d4dec7529efec28a5bd0edad0fd30
deleted: sha256:0271b8eebde3fa9a6126b1f2335e170f902731ab4942f9f1914e77016540c7bb
untagged: kalilinux/kali-linux-docker:latest
untagged: kalilinux/kali-linux-docker@sha256:28ff9e4bf40f7399e0570394a2d3d388a7b60c748be1b0a180c14c87afad1968
deleted: sha256:c927a54ec8a46164d7046b2a6dc09b2fce52b3066317d50cf73d14fa9778ca48
deleted: sha256:244c1920ef0442167cdbd095e5d29813cb5be0b70cc116faf8d7e50074f6c446
deleted: sha256:7748477cf079d6b0c13925ca90a5a1c7e93b8b508853f0cdff506c18caee14bd
deleted: sha256:dd9acc2ebbb7901b407d4270d4fd065d9bee10d11f2df13a256d892cc6e892f9
deleted: sha256:46c7843e50429fcafe2d3b6c676ac1a25e00851420ba2b1d52c69307f68ab3e5
deleted: sha256:f0944ddbb9bb11fb68f7edbde8e849233f7562d8087248c944e8c2fc7fe9fc0b
deleted: sha256:146e723c1713625c00cc736d74c9f6a16bd24464c42b33a8a234ec6e4c8b61ef
deleted: sha256:bca8a24862472a44c7ab1e3bdf2d5e4008e35d6c50b94f2547d3d595d86abef1
deleted: sha256:749be9d8a5ebb09cbc58d50c4b7244a10accdedc2a01c1d65d07d25322caacad
deleted: sha256:2d9e7ebb987a4cfb3142ce1612640248085d05b264012cb0885b3062105dfcb4
deleted: sha256:0655dca90e7c9c62d48128343ce89e016ae9f9df75c9dd6ad66c281e04e2b431
deleted: sha256:e78aa5d90040550584961eaccec1d047b755e97148fe753186e221c5ac40e330
deleted: sha256:598719dc4ba2de8d1be6564ca1f43846497608188cd20476712f7449755fea21
deleted: sha256:b084b4800972b561c21d804fab08c1fff0b9a9bcbf95a5394c0d4292c145c6d0
deleted: sha256:2e1b87f8f95e635c8ff4cbde28be38df39e8f3614576e09d7fb69c20421d1727
deleted: sha256:4a4a13e39112faa3b7ef0cb307bbf926fd1e46f3fbb9bc803cb9f4ab2f7694b0
untagged: alpine:latest
untagged: alpine@sha256:ccba511b1d6b5f1d83825a94f9d5b05528db456d9cf14a1ea1db892c939cda64
untagged: alpine-io:latest
deleted: sha256:3a043b0342a4907a1dfc95e2ea5e4df6a8e92d29dfe5d5910282bdfff27045d4
deleted: sha256:ddfb1d0e7629fd459b04f6efa89109ea0f7458aec76760e31888464d3074ae56
deleted: sha256:b6a7ea2197b744efab03320eda59d036ac3458ab7a0c5ada355faff0dd936af0
deleted: sha256:c96ab19b9ede349cb84e510a76a93d2b155aad54416f1591d7128cdeef228efc
deleted: sha256:43e7d32baaf31ab6bd4210ff3df54d1dec57cc761eab88c5eaef2973d6bed770
deleted: sha256:11a9226e2c0aeaa12408501b274575c8ee471a785b332af3c776e23dfd2eb629
deleted: sha256:bd9f490e64a2ceccdeb936f43047c0757635b4bc88159ba5b191285ef41f535c
deleted: sha256:e21c333399e0aeedfd70e8827c9fba3f8e9b170ef8a48a29945eb7702bf6aa5f
deleted: sha256:04a094fe844e055828cb2d64ead6bd3eb4257e7c7b5d1e2af0da89fa20472cf4
untagged: nginx:latest
untagged: nginx@sha256:cf8d5726fc897486a4f628d3b93483e3f391a76ea4897de0500ef1f9abcd69a1
deleted: sha256:3f8a4339aadda5897b744682f5f774dc69991a81af8d715d37a616bb4c99edf5
deleted: sha256:bb528503f6f01b70cd8de94372e1e3196fad3b28da2f69b105e95934263b0487
deleted: sha256:410204d28a96d436e31842a740ad0c827f845d22e06f3b1ff19c3b22706c3ed4
deleted: sha256:2ec5c0a4cb57c0af7c16ceda0b0a87a54f01f027ed33836a5669ca266cafe97a

Total reclaimed space: 5.219GB
```