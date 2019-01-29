---
layout: post
title: Deploying Shadowsocks with Docker in Debian 9
key: 20190129
tags: Shadowsocks Debian Docker
---
# 在 Debian 9 上用 Docker 搭建 Shadowsocks 服务器

## 从 Vultr 到 BandwagonHost，从 CentOS 到 Debian

昨天买了传说中的[搬瓦工](https://bandwagonhost.com)服务器，普通的 CN2 线路。如果实测下来速度真的比 Vultr 的服务器快，以后就准备换这个了。换服务器的想法来自这篇[「科学上网」](https://github.com/haoel/haoel.github.io/blob/master/README.md)。

之前服务器上装的系统是 CentOS 7 ——鸟哥学徒的不二选择，这次想试试 Debian 了。CentOS 虽然稳定，但是软件版本比较老，有时装某些软件会比较麻烦，科学上网要跟得上时代发展，还是不要太过保守。

第一步，管理后台安装了 Debian 9 x86_64 系统。系统装完后搬瓦工默认的 ssh 端口是随机的，这点就比 Vultr 好，想起当初不懂，装完系统美滋滋，结果 22 端口被暴力登录疯狂攻击……

<!--more-->

顺便讲一下修改 ssh 端口的方法。编辑 `/etc/ssh/sshd_config` 文件，在其中修改或添加端口：

```shell
Port 1234
Port 4567
```

## sudo and sbin

不习惯直接用 root 账号，新建了自己的号，然后发现没有 sudo 命令。需要自己安装：

```shell
apt-get update
apt-get install sudo
```

想用某些命令，发现 `not found` ，一查原来 sbin 没有放到 PATH 中。需要输入：

```shell
export PATH=$PATH:/sbin
```

## Enable BBR

Debian 9 内置 BBR (Bottleneck Bandwidth and RTT) 模块，只需要命令开启就可以。参考[「Debian 9开启Google BBR」](https://zocodev.com/debian-9-enable-google-bbr.html)：

```shell
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

按文章说要重启，我没重启好像也生效了，因为输入：

```shell
lsmod | grep bbr
```

已经有内容出现了，应该算成功开启了吧。

## Install Docker

自从接触了 Docker 之后就爱不释手。以前直接把 Shadowsocks 装到系统上，这次准备装到 Docker 容器里，方便以后再换服务器时可以一键部署。

在 Debian 里安装 Docker 可参考官方文档[「Get Docker CE for Debian」](https://docs.docker.com/install/linux/docker-ce/debian/)。

## Use Dockerfile

花了半天，[学习用法](https://docs.docker.com/engine/reference/#environment-replacement)，参考他人，加上自己调试，终于 Build 成功加顺利运行。代码详见 [tjcccc/myDockers/shadowsocks](https://github.com/tjcccc/myDockers/blob/master/shadowsocks/)，也可访问 Docker Hub 的 [tjcccc/shadowsocks-server](https://cloud.docker.com/repository/docker/tjcccc/shadowsocks-server)。

### Docokerfile

```Dockerfile
FROM debian:stable
COPY ./entrypoint.sh /entrypoint.sh

RUN apt-get update -y && apt-get install -y git python-pip && apt-get clean
RUN pip install git+https://github.com/shadowsocks/shadowsocks.git@master
RUN chmod +x /entrypoint.sh

ENV SS_SERVER 0.0.0.0
ENV SS_SERVER_PORT 12345
ENV SS_PASSWORD password
ENV SS_METHOD aes-256-cfb
ENV SS_TIMEOUT 300

EXPOSE ${SS_SERVER_PORT}

ENTRYPOINT ["/entrypoint.sh"]
```

简单说明一下：

- `FROM`: 是容器用到的 Linux 系统。暂时用 Debian 的最新稳定版，以后再试一下精简版。
- `COPY`: 由于最后的 `ENTRYPOINT` 只能使用一发「命令」而不能使用「命名 + 选项」，所以把复杂的命令行操作都放到了一个 sh 文件里，然后把这个文件从代码库目录拷贝给容器，最后由 `ENTRYPOINT` 执行。
- `RUN`: 执行命令行操作。安装 Shadowsocks 及相关的软件，再给予 entrypoint.sh 文件执行能力。
- `ENV`: 设置环境变量以及默认参数。默认端口我改成了 12345，方便自己使用。我自己用的话，只要设定密码就可以了。
- `EXPOSE` 暴露端口方便外部使用和映射。
- `ENTRYPOINT` 容器的入口点。注意不能使其一发运行完毕，必须留在进程里，否则容器一跑完就自动退出了。我一开始自作聪明给 `ssserver` 加了后台运行的 `-d` 选项，结果 `run` 完就 `Exited(1)` 了……

### entrypoint.sh

```sh
#!/bin/bash

/usr/local/bin/ssserver --fast-open -s ${SS_SERVER} -p ${SS_SERVER_PORT} -k ${SS_PASSWORD} -m ${SS_METHOD} -t ${SS_TIMEOUT} "$@"
```

简单说明一下：

- `--fast-open`：这个默认开启。
- 各种环境变量都加进去了，所以 `ssserver` 后面不能再加 `-s -p -k -m -t` 这五种选项。但是还可以加其他的，`"$@"` 就是表示在 `entrypoint.sh` 后面接的输入内容都会加在这条命令后面。即方便使用必要配置，又允许自定义配置，两开花。

## 运行感觉

好像打开页面是比 Vultr 快一点，不错。

还有一点疑问，在主机上开启了 BBR 是否会作用到容器中，是否应该在容器里开启？Github 上其他人的 Dockerfile 没看到有写，大概是不需要的。
