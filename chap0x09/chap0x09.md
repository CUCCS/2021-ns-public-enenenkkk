# 实验九 入侵检测

### 环境配置

##### snort安装

```bash
# 禁止在apt安装时弹出交互式配置界面
export DEBIAN_FRONTEND=noninteractive
apt install snort
```

### 实验过程

##### 实验一：配置snort为嗅探模式

```bash
# 显示IP/TCP/UDP/ICMP头
snort –v
```

<img src="img\snort-v.png" style="zoom: 80%;" />

```bash
# 显示应用层数据
snort -vd
```

<img src="img\snort-vd.png" style="zoom: 80%;" />

```bash
# 显示数据链路层报文头
snort -vde
```

<img src="img\snort-vde.png" style="zoom: 80%;" />

<img src="img\ping_baidu.png" style="zoom: 80%;" />

```bash
# 使用 CTRL-C 退出嗅探模式
# 嗅探到的数据包会保存在 /var/log/snort/snort.log.<epoch timestamp>
# 其中<epoch timestamp>为抓包开始时间的UNIX Epoch Time格式串
# 可以通过命令 date -d @<epoch timestamp> 转换时间为人类可读格式
# exampel: date -d @1511870195 转换时间为人类可读格式
# 上述命令用tshark等价实现如下：
tshark -i eth1 -f "port not 22" -w 1_tshark.pcap
```

<img src="img\time_change.png" style="zoom: 80%;" />

##### 实验二：配置并启用snort内置规则

```bash
# /etc/snort/snort.conf 中的 HOME_NET 和 EXTERNAL_NET 需要正确定义
# 例如，学习实验目的，可以将上述两个变量值均设置为 any
snort -q -A console -b -i eth1 -c /etc/snort/snort.conf -l /var/log/snort/
```

<img src="img\any.png" style="zoom: 80%;" />

##### 实验三：自定义snort规则

```bash
# 注：这里用课件的代码会导致实际传入的变量为空（不是已经转义了么），因此使用vim
# 新建自定义 snort 规则文件
vim /etc/snort/rules/cnss.rules
# INSERT
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"Access Violation has been
detected on /etc/passwd ";flags: A+; content:"/etc/passwd"; nocase;sid:1000001;
rev:1;)
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"Possible too many
connections toward my http server"; threshold:type threshold, track by_src, count
100, seconds 2; classtype:attempted-dos; sid:1000002; rev:1;)
# 添加配置代码到 /etc/snort/snort.conf
include $RULE_PATH/cnss.rules
# 开启apache2
service apache2 start
# 应用规则开启嗅探
snort -q -A fast -b -i eth1 -c /etc/snort/snort.conf -l /var/log/snort/
# 在attacker上使用ab命令进行压力测试
ab -c 100 -n 10000 http://$dst_ip/hello
```

<img src="img\exp3.png" style="zoom:80%;" />

##### 实验四：和防火墙联动

###### Kali-Victim-1

```bash
# 将Guardian-1.7.tar.gz在主机解压缩后，拖入虚拟机Kali-Victim-1
# 安装 Guardian 的依赖 lib
sudo apt-get update
sudo apt-get install libperl4-corelibs-perl
# 解压缩 Guardian-1.7.tar.gz
tar zxf guardian.tar.gz
# 安装 Guardian 的依赖 lib
apt install libperl4-corelibs-perl
```

###### 在Victim上先后开启 **snort**和 guardian.pl

```bash
# 开启 snort
snort -q -A fast -b -i eth0 -c /etc/snort/snort.conf -l /var/log/snort/
```

###### 编辑 guardian.conf 并保存，并更改参数为

```bash
Interface eth0
HostIpAddr 172.16.111.148
```

<img src="img\exp-2.jpg" style="zoom: 50%;" />

```bash
# 启动 guardian.pl
perl guardian.pl -c guardian.conf
```

###### Kali-Attackter

在Attack上用 nmap 暴力扫描 Victim：

```bash
nmap 172.16.111.148 -A -T4 -n -vv
```

guardian.conf 中默认的来源IP被屏蔽时间是 60 秒（屏蔽期间如果黑名单上的来源IP再次触发snort报警消息，则屏蔽时间会继续累加60秒）

```bash
root@KaliRolling:~/guardian# iptables -L -n
Chain INPUT (policy ACCEPT)
target prot opt source destination
REJECT tcp -- 192.168.56.101 0.0.0.0/0 reject-with tcp-
reset
DROP all -- 192.168.56.101 0.0.0.0/0
Chain FORWARD (policy ACCEPT)
target prot opt source destination
Chain OUTPUT (policy ACCEPT)
target prot opt source destination
# 1分钟后，guardian.pl 会删除刚才添加的2条 iptables 规则
root@KaliRolling:~/guardian# iptables -L -n
Chain INPUT (policy ACCEPT)
target prot opt source destination
Chain FORWARD (policy ACCEPT)
target prot opt source destination
Chain OUTPUT (policy ACCEPT)
target prot opt source destination
```

<img src="img\exp-1.jpg" style="zoom: 67%;" />

##### 实验思考题 

IDS与防火墙的联动防御方式相比IPS方式防御存在哪些缺陷？是否存在相比较而言的优势？

- IDS通过软、硬件，对网络、系统的运行状况进行监视，尽可能发现各种攻击企图、攻击行为或者攻击结果，以保证网络系统资源的机密性、完整性和可用性。

- IPS是一部能够监视网络或网络设备的网络资料传输行为的计算机网络安全设备，能够即时的中断、调整或隔离一些不正常或是具有伤害性的网络资料传输行为。

###  问题

`ERROR: /etc/snort/rules/cnss.rules(2) Unable to process the IP address: $EXTERNAL_NET. Fatal Error, Quitting...`

解决方法：分别修改`$EXTERNAL_NET`、`$HTTP_SERVERS为Attacker`、`Victim`的主机ip地址

```bash
  alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"Access Violation has been detected on /etc/passwd ";flags: A+; content:"/etc/passwd"; nocase;sid:1000001; rev:1;)

  alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"Possible too many connections toward my http server"; threshold:type threshold, track by_src, count 100, seconds 2; classtype:attempted-dos; sid:1000002; rev:1;)
```



### 参考资料

[课本](https://c4pr1c3.gitee.io/cuc-ns/chap0x09/exp.html)

[师哥实验](https://github.com/CUCCS/2020-ns-public-LyuLumos/tree/ch0x09/ch0x09)


