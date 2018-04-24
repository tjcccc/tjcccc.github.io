## SSH Brute Force Attacks Prevention

　　昨日经朋友提醒，注意到登录 SSH 后，有很多其他 IP 错误登录的提示——原来是遭受了所谓 SSH 暴力登录尝试攻击。他们有些来自黑客，有些可能来自……

　　这个问题并不新鲜，但我是第一次架设 Linux 服务器使用，全无经验，有点茫然，不得不谷歌百度，赶紧防治一下——作为我的运维新手任务。

## 东北大学 IP 黑名单方法　　

　　通过搜索尝试攻击的 IP 地址，搜到了[东北大学的黑名单](http://antivirus.neu.edu.cn/scan/ssh.php)和应对方法：

```shell
ldd `which sshd` | grep libwrap # 确认sshd是否支持TCP Wrapper，输出类似:libwrap.so.0 => /lib/libwrap.so.0 (0x00bd1000)
cd /usr/local/bin/
wget antivirus.neu.edu.cn/ssh/soft/fetch_neusshbl.sh
chmod +x fetch_neusshbl.sh
cd /etc/cron.hourly/
ln -s /usr/local/bin/fetch_neusshbl.sh .
./fetch_neusshbl.sh
```

如不支持 TCP Wrapper, 需先安装：

```shell
yum install tcp_wrappers
```

这个方法的问题是，如果有新 IP 来袭，就防不了了。于是我又采用了 iptables 方法。

## 用 iptables 防止 SSH 登录攻击

　　未登陆过的用户如登录失败超过三次，使其一分钟内禁止登录：

```shell
/usr/sbin/iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent --set
/usr/sbin/iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent  --update --seconds 60 --hitcount 4 -j DROP
``` 

via: [Block SSH Brute Force Attacks with IPTables](https://www.rackaid.com/blog/how-to-block-ssh-brute-force-attacks/)

（To be continued)
1804240835