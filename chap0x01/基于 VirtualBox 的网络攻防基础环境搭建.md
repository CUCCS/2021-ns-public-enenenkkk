# 基于 VirtualBox 的网络攻防基础环境搭建
## 实验目的
- 掌握 VirtualBox 虚拟机的安装与使用；
- 掌握 VirtualBox 的虚拟网络类型和按需配置；
- 掌握 VirtualBox 的虚拟硬盘多重加载；

## 实验环境
以下是本次实验需要使用的网络节点说明和主要软件举例：
- VirtualBox 虚拟机
- 攻击者主机（Attacker）：Kali Rolling 2021.2
- 网关（Gateway, GW）：Debian 10
- 靶机（Victim）：Debian 10 / xp-sp3 / Kali


## 实验要求
- [虚拟硬盘配置成多重加载，效果如下图所示；](#a)
![](img/vb-multi-attach.png)
- [搭建满足如下拓扑图所示的虚拟机网络拓扑；](#b)
![](img/vb-exp-layout.png)
- 完成以下网络连通性测试；

- [x] [靶机可以直接访问攻击者主机](#1)
- [x] [攻击者主机无法直接访问靶机](#2)
- [x] [网关可以直接访问攻击者主机和靶机](#3)
- [x] [靶机的所有对外上下行流量必须经过网关](#4)
- [x] [所有节点均可以访问互联网](#5)
## 实验步骤
### 一、<span id="a">虚拟硬盘配置成多重加载</span>
    1. 通过老师提供的链接完整下载好三个文件 -> 打开virtualbox导入ova文件安装好Kali，新建两个虚拟机分别导入Debian 10 和 Windows XP SP3；
    2. 管理 -> 虚拟介质管理 -> 选中盘片 -> 释放，选择多重加载

![](img/multi-attach.png)
### <span id="b">二、搭建实验要求的虚拟机网络拓扑；</span>
#### 根据拓扑图新建六个虚拟机：
![](img/six-computer.png)

    1. 网关（Debian-Gateway）设置四块网卡：
![](img/gw-cards.jpg)

    开启Debian-Gateway虚拟机，登录并输入`ip a`查看各网卡IP地址：
![](img/gw-ip-a.jpg)
   
    2.Kali-Attacker设置：
![](img/kaliattacker-set.png)
    
    开启Kali-Attacker虚拟机，登陆并查看IP地址：
![](img/kaliattacker-ip.png)

    3.Victim组设置：
    victim1组：Kali-Victim-1与XP-Victim-1都设置[内部网络-intnet1]（注：xp虚拟机中 网卡 -> 高级-> 芯片控制 -> 选择 PCnet-FAST III）;
![](img/kali-victim-1-card.png)
![](img/xp-victim-1-card.png)

    开启虚拟机并查看IP地址：
![](img/kali-victim-1-ip.png)
![](img/xp-victim-1-ip.png)
  
    victim2组：Debian-Victim-2与XP-Victim都设置[内部网络-intnet2]。
![](img/debian-victim-2-card.png)
![](img/xp-victim-2-card.png)

    开启虚拟机并查看IP地址：
![](img/debian-victim-2-ip.png)
![](img/xp-victim-2-ip.png)

### 三、连通性测试

- [x] <span id="1">**靶机可以直接访问攻击者主机**</span>  

    攻击者：Kali-Attacker，ip地址：10.0.2.15;

    四台靶机：victim1组&victim2组
`(xp虚拟机需关闭防火墙)`

*victim1组* --均ping通
![](img/intnet1-ping.png)
*victim2组* --均ping通
![](img/intnet2-ping.png)

- [x] <span id="2">**攻击者主机无法直接访问靶机**</span>
  
    Kali-Attacker对两组靶机都无法ping通。
![](img/ping-intnet1.png)
![](img/ping-intnet2.png)


- [x] <span id="3">**网关可以直接访问攻击者主机和靶机**</span>

在Debian-Gateway界面依次ping攻击者虚拟机和两台不同局域网靶机，均可ping通
![](img/gw-ping-attacker.png)
![](img/gw-ping-victim.png)
- [x] <span id="4">**靶机的所有对外上下行流量必须经过网关**</span>
  
    以访问Debian-Victim-2端口为例，使用命令

    `sudo tcpdump -i enp0s3`

    在Debian-Vicitim-2端 `ping www.baidu.com`

    于网关界面监测动态:
![](img/tcp-thru-gw.png)
- [x] <span id="5">**所有节点均可以访问互联网**</span>
  
    使用六台虚拟机分别访问baidu,均可ping通。

*Debian-Gateway*
![](img/gw-ping-baidu.png)

*Kali-Attacker*
![](img/attacker-ping-baidu.png)

*victim1组*
![](img/v1-ping.png)

*victim2组*
![](img/v2-ping.png)

### 四、实验遇到的问题
1. 在debian、kali用ping抓包后需`Ctrl+c`停止；

### 五、参考资料
[实验教学视频](https://www.bilibili.com/video/BV1CL41147vX?p=12)
