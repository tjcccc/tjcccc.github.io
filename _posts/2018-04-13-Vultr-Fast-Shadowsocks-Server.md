---
layout: post
title: How to Setup a Shadowsocks Server on Vultr Host (CentOS 7)
key: 20180413
tags: Shadowsocks Vultr
---
# 用 Vultr 的 CentOS 7 主机搭建 Shadowsocks 服务器

## 1. Install CentOS 7 on the Vultr host

## 2. 一键安装 Shadowsocks (SS) 并随安装配置。参考：https://teddysun.com/342.html

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

## 3. 更换 CentOS 7 内核以安装“锐速”——对 SS 进行 TCP 加速。参考：https://www.zhangfangzhou.cn/lotserver.html

### 3.1 更换内核：

```shell
rpm -ivh http://file.asuhu.com/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force
```

### 3.2 设置启动的内核：

```shell
grub2-set-default `awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg | grep '(3.10.0-229.1.2.el7.x86_64) 7 (Core)'|awk '{print $1}'`
```

### 3.3 重启（reboot）后一键安装“锐速”。全部选默认即可。

```shell
wget --no-check-certificate -O appex.sh https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh && chmod +x appex.sh && bash appex.sh install
```

To be continued...