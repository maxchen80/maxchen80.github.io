---
layout: post
title: 虚拟机安装CentOS后的网络配置
---
win平台下的虚拟机软件，能够选择的基本就是vmware和VirtualBox。两款软件最显著的区别就是vmware是商业软件，VirtualBox是开源软件。网上有比较多的这两款软件的硬件性能利用率对比，有的还包括了kvm和xem，实际性能差异和稳定性基本都是微乎其微。作为日常学习，而非生产环境的使用，对于性能和稳定性不用过于纠结，因为都是非常牛逼的软件呵呵。那么win平台因为免费一般情况下推荐VirtualBox，如果实在喜欢vmware的话网上破解码也是一搜一大把。
使用vmware和VirtualBox，首先需要解决的问题就是系统安装完毕之后的网络相关设置，在此记录下来避免每次都花费不必要的时间。

### VirtualBox网络配置
关于NAT、Bridge、HostOnly的区别和讨论，不在本文讨论范围，谷歌和百度可以搜索到大量的结果。
##### NAT模式
VirtualBox安装CentOS-6.5之后，已经默认设置了一个网络地址转换（NAT）的eth0网卡，但是没有开启，可以使用`ifup eth0`启用。试试`ping qq.com`，可以正常链接。
如果需要SSH连接上虚拟机，我们还需要设置一个仅主机网卡，或者一个桥接网卡。
##### HostOnly模式
在VirtualBox里选择虚拟电脑，进入设置界面，创建一张新的网卡，链接方式选择仅主机(Host-Only)适配器：
![Alt text](/assets/images/2016-01-10-vm-network-config-001.png)
进入宿主机的控制面板\网络和 Internet\网络和共享中心\更改适配器设置，查看VirtualBox Host-Only Ethernet Adapter的状态信息：
![Alt text](/assets/images/2016-01-10-vm-network-config-002.png)
在虚拟机里新增一个网卡：
	# cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth1
	# vi /etc/sysconfig/network-scripts/ifcfg-eth1
	DEVICE=eth1
	HWADDR=08:00:27:1E:86:74
	TYPE=Ethernet
	ONBOOT=yes
	BOOTPROTO=static
	IPADDR=192.168.56.2
	NETMASK=255.255.255.0
	GATEWAY=192.168.56.0
	# service network restart
##### Bridge模式
在VirtualBox里选择虚拟电脑，进入设置界面，创建一张新的网卡，链接方式选择桥接网卡：
![Alt text](/assets/images/2016-01-10-vm-network-config-003.png)
进入宿主机的控制面板\网络和 Internet\网络和共享中心\更改适配器设置，查看VirtualBox Host-Only Ethernet Adapter的状态信息：
![Alt text](/assets/images/2016-01-10-vm-network-config-004.png)
在虚拟机里新增一个网卡：
	# cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth2
	# vi /etc/sysconfig/network-scripts/ifcfg-eth2
	DEVICE=eth2
	HWADDR=08:00:27:97:BB:26
	TYPE=Ethernet
	ONBOOT=yes
	BOOTPROTO=static
	IPADDR=10.161.162.255
	NETMASK=255.255.254.0
	GATEWAY=10.161.162.1
	# service network restart

### vmware网络配置
未完待续



