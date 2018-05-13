# 用 Vultr 的 CentOS 7 主机搭建 Shadowsocks 服务器

## 1. Install CentOS 7 as OS on the Vultr host.

## 2. 一键安装 Shadowsocks （SS）并随安装配置。参考：https://teddysun.com/342.html

### 2.1 终端执行一键安装脚本：
```
wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```
### 2.2 在 /etc/shadowsocks.json 中可改配置。多用户改法：
```
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
### 2.3 服务命令（2015 年 08 月 28 日修正）：
```
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
```
## 3. 更换 CentOS 7 内核以安装“锐速”——对 SS 进行 TCP 加速。参考：https://www.zhangfangzhou.cn/lotserver.html

### 3.1 更换内核：
```
rpm -ivh http://file.asuhu.com/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force
```
### 3.2 设置启动的内核：
```
grub2-set-default `awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg | grep '(3.10.0-229.1.2.el7.x86_64) 7 (Core)'|awk '{print $1}'`
```
### 3.3 重启（reboot）后一键安装“锐速”。全部选默认即可。
```
wget --no-check-certificate -O appex.sh https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh && chmod +x appex.sh && bash appex.sh install
```

---

以下内容未完成。

---

## 4. 安装 Kcptun 通过“将 TCP 流转换为 KCP+UDP 流”对 SS 进行加速。参考：https://www.gblm.net/209.html

### 4.1 执行一键安装：
```
wget https://raw.githubusercontent.com/kuoruan/kcptun_installer/master/kcptun.sh
chmod +x ./kcptun.sh
./kcptun.sh
```
### 4.2 相关操作命令：
```
如需更新：
./kcptun.sh update
注：全面支持脚本、Kcptun和配置的更新！
如需重新配置：
./kcptun.sh reconfig
卸载：
./kcptun.sh uninstall
```

### 4.3 根据 Kcptun 安装后提示的客户端下载地址下载对应系统的 Kcptun 客户端。本文以 kcptun-lunux-amd64 为例。

### 4.4 配置开机启动脚本

下载后解压到 ~/Kcptun 下。编写启动脚本：
```
touch /etc/init.d/kcptun
chmod +x /etc/init.d/kcptun
vim /etc/init.d/kcptun
```
脚本内容：
```
#!/bin/bash

#
# KCPTun       Startup script for the KCPTun Server
#
# chkconfig: - 90 10
# 
# description: The KCPTun is a secure tunnel based On KCP with N:M Multiplexing. \
#              (https://github.com/xtaci/kcptun)
# processname: kcptun
# config: /etc/sysconfig/kcptun
# pidfile: /var/run/kcptun.pid
# 

### BEGIN INIT INFO
# Provides: KCPTun
# Required-Start: $network $syslog $local_fs $remote_fs $named
# Required-Stop: $network $local_fs $remote_fs 
# Should-Start: 
# Should-Stop:        
# Default-Start:      
# Default-Stop:  
# Short-Description: Start and stop KCPTun Server
# Description: The KCPTun is a secure tunnel based On KCP with N:M Multiplexing.
### END INIT INFO


# Author: farawayzheng <http://blog.csdn.net/farawayzheng_necas>
# 
# To install:
#   Copy this file to /etc/rc.d/init.d/kcptun
#   $ chkconfig --add kcptun
#
#
# To uninstall:
#   $ chkconfig --del kcptun
#   

#export PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin:/opt/bin:

BASE=$(basename $0)

# modify these in /etc/sysconfig/$BASE (/etc/sysconfig/kcptun)
KCPTUN=/foo/kcptun/server_linux_amd64

KCPTUN_PIDFILE=/var/run/$BASE.pid
KCPTUN_LOGFILE=/var/log/$BASE.log
KCPTUN_LOCKFILE=/var/lock/subsys/$BASE
KCPTUN_OPTS="-t 127.0.0.1:9000 -l :29900 --key mykey --crypt aes-128 --mode fast2"
KCPTUN_DESC="KCPTUN"

# Source function library.
. /etc/rc.d/init.d/functions

if [ -f /etc/sysconfig/$BASE ]; then
    . /etc/sysconfig/$BASE
fi

# Check kcptun server is present
if [ ! -x $KCPTUN ]; then
    echo "$KCPTUN not present or not executable!"
    exit 1
fi

RETVAL=0
STOP_TIMEOUT=${STOP_TIMEOUT-10}

start() {

    if [ -f ${KCPTUN_LOCKFILE} ]; then

        if [ -s ${KCPTUN_PIDFILE} ]; then
            echo "$BASE might be still running, stop it first!"
            killproc -p ${KCPTUN_PIDFILE} -d ${STOP_TIMEOUT} $KCPTUN
        else
            echo "$BASE was not shut down correctly!"
        fi

        rm -f ${KCPTUN_PIDFILE} ${KCPTUN_LOCKFILE}
        sleep 2
    fi

    echo -n $"Starting $BASE: "
    $KCPTUN --log ${KCPTUN_LOGFILE} $KCPTUN_OPTS &
    RETVAL=$?

    if [ "$RETVAL" = "0" ]; then
        success
        sleep 2
        ps -A o pid,cmd | grep "$KCPTUN --log ${KCPTUN_LOGFILE} $KCPTUN_OPTS" | awk '{print $1}' | head -n 1 > ${KCPTUN_PIDFILE} 
    else
        failure
    fi
    echo

    [ $RETVAL = 0 ] && touch ${KCPTUN_LOCKFILE}
    return $RETVAL
}

stop() {
    echo -n $"Stopping $BASE: "
    killproc -p ${KCPTUN_PIDFILE} -d ${STOP_TIMEOUT} $KCPTUN
    RETVAL=$?
    echo
    [ $RETVAL = 0 ] && rm -f ${KCPTUN_PIDFILE} ${KCPTUN_LOCKFILE}
    return $RETVAL
}


case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  status)
        status -p ${KCPTUN_PIDFILE} $KCPTUN
        RETVAL=$?
        ;;
  restart)
        stop
        start
        ;;
  *)
        echo $"Usage: $BASE { start | stop | restart | status }"
        RETVAL=2
        ;;
esac

exit $RETVAL
```
以上代码为示例，需根据之前安装时配置的参数调整：
```
脚本中的 KCPTUN_OPTS=”-t 127.0.0.1:9000 -l :29900 –key mykey –crypt aes-128 –mode fast2” 可以根据具体情况来进行修改
```

### 4.5 配置开机启动：
激活开机启动：
```
chkconfig --add kcptun
chkconfig kcptun on
```
启动kcptun：
```
service kcptun start
```