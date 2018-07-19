---
layout: post
title: How to Setup a Shadowsocks Server on Vultr Host (CentOS 7)
key: 20180413
tags: Shadowsocks Vultr
---
# 用 Vultr 的 CentOS 7 主机搭建 Shadowsocks 服务器

## 1. Install CentOS 7 on the Vultr host

## 2. 一键安装 Shadowsocks (SS) 并随安装配置。

　　参考：

- [Shadowsocks Python版一键安装脚本](https://teddysun.com/342.html)

### 2.1 终端执行一键安装脚本：

```shell
wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

<!--more-->

### 2.2 在 /etc/shadowsocks.json 中可改配置。多用户改法：

```json
{
    "server":"0.0.0.0",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "8989":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "timeout":300,
    "method":"your_encryption_method",
    "fast_open": false
}
```

### 2.3 服务命令：

```shell
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
```

## 3、4 略。

## 5. 用 BBR 代替“锐速”加速

　　7 月 5 日发现主机 CPU 使用率极高，接近 100%，查了一下可能和锐速有关。于是参考网上建议，改用 Google BBR 进行 TCP 加速。

　　参考：

- [Vultr Centos安装shadowsocks服务端并开启BBR加速](https://rootrl.github.io/2017/10/11/Vultr-Centos%E5%AE%89%E8%A3%85shadowsocks%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%B9%B6%E5%BC%80%E5%90%AFBBR%E5%8A%A0%E9%80%9F/)
- [一键安装最新内核并开启 BBR 脚本](https://teddysun.com/489.html)

### 5.1 一键安装 BBR

　　一键安装：

```shell
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```

安装完成后根据提示重启。

### 5.2 检查是否启动

　　重启后输入：

```shell
uname -r
```

如看到版本号说明 BBR 安装成功。

　　检查是否开启：

```shell
sysctl net.ipv4.tcp_available_congestion_control
# 返回值一般为：net.ipv4.tcp_available_congestion_control = bbr cubic reno

sysctl net.ipv4.tcp_congestion_control
# 返回值一般为：net.ipv4.tcp_congestion_control = bbr

sysctl net.core.default_qdisc
# 返回值一般为：net.core.default_qdisc = fq

lsmod | grep bbr
# 返回值有tcp_bbr则说明已经启动
```

To be continued...