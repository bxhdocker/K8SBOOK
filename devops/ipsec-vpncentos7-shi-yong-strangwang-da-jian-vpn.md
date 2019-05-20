# Ipsec VPN-centos7使用strangwang搭建vpn

 本教学介绍了如何使用 Strongswan 5.x.x 在 centos7+ 服务器上架设支持 ikev1/ikev2 的 Ipsec VPN。适用于 openSUSE、iOS、Android、Windows 和其它 Linux。

## 简介：

### 什么是 IPsec？

IPsec 是 [虚拟私密网络（VPN）](https://zh.opensuse.org/SDB:VPN_%E9%85%8D%E7%BD%AE) 的一种，用于在服务器和客户端之间建立加密隧道并传输敏感数据之用。它由两个阶段组成，第一阶段（Phrase 1, ph1），交换金钥建立连接，使用互联网金钥交换（ike）协议; 第二阶段（Phrase 2, ph2），连接建立后对数据进行加密传输，使用封装安全载荷（esp）协议。参考：[维基百科 IPsec 词条](http://zh.wikipedia.org/wiki/IPsec)。

其中，第一阶段和第二阶段可以使用不同的加密方法（cipher suites）。甚至，第一阶段 ike 协议的第一版（ikev1）有两种模式，主力模式（main mode）和积极模式（aggressive mode），主力模式进行六次加密握手，而积极模式并不加密，以实现快速建立连接的目的。

第一阶段的 ike 协议有两个版本（ikev1/ikev2），不同的开源/闭源软件实现的版本均不同，不同的设备实现的版本也不同。再联系到第一阶段/第二阶段使用的各种不同加密方法，使得 IPsec 的配置有点黑魔法的性质，要么完全懂，通吃; 要么完全不懂，照抄

本文以公司内网192.168.0.0/21和腾讯vpc内网10.125.0.0/16做ike vpn，互联网络通信

网络图示：

![](../.gitbook/assets/image%20%2896%29.png)

## 介绍：

系统：centos 7

软件：strongswan 5.7

**补充说明**

 StrongSwan 的发行版已包含在 EPEL 源中, 但是CentOS源的包比较旧，所以我们手动在官网[https://www.strongswan.org/download.html](https://www.strongswan.org/download.html)下载安装包

 strongSwan的发行版已包含在EPEL源中, 但源中的包版本5.3.2比较低，目前官网上5.4.0已经发布了。

```text
安装EPEL源：
yum -y install epel-release
```

 查看SELinux状态

```text
[root@slhk ~]# sestatus
  SELinux status: disabled
```

如果SELinux没有被禁用，需禁用之，并重启服务器

查看防火墙状态

```text
systemctl status firewalld
```

## **一、安装**

使用epel仓库里面的yum安装

\# yum install strongswan 

yum安装说明：

官方安装说明：[https://centos.pkgs.org/7/epel-x86\_64/strongswan-5.7.1-1.el7.x86\_64.rpm.html](https://centos.pkgs.org/7/epel-x86_64/strongswan-5.7.1-1.el7.x86_64.rpm.html)

centos6 安装5.4版本说明 [https://centos.pkgs.org/6/epel-x86\_64/strongswan-5.4.0-2.el6.x86\_64.rpm.html](https://centos.pkgs.org/6/epel-x86_64/strongswan-5.4.0-2.el6.x86_64.rpm.html)

二进制安装说明：

官方安装说明：[https://wiki.strongswan.org/projects/strongswan/wiki/InstallationDocumentation](https://wiki.strongswan.org/projects/strongswan/wiki/InstallationDocumentation)

官方页面: [https://www.strongswan.org/download.html](https://www.strongswan.org/download.html)

## **二、配置strongswan**

strongSwan包含3个配置文件，在目录/etc/strongswan/下。  
strongswan.conf            strongSwan各组件的通用配置  
ipsec.conf                     IPsec相关的配置，定义IKE版本、验证方式、加密方式、连接属性等等  
ipsec.secrets                 定义各类密钥，例如：私钥、预共享密钥、用户账户和密码

### 1.公司内网192.168.1.228上配置说明

#### vim /etc/strongswan/ipsec.conf

```text
config setup
       uniqueids=never
     #  charondebug="ike 4, knl 4, net 4, cfg 4"
conn %default
     authby=psk
     type=tunnel
     #compress = yes
     #keyexchange=ike
     #ike=aes256-sha256-sha1-modp2048-modp1024,3des-sha256-sha1-modp2048-modp1024!
     #esp=aes256-sha256-sha1,3des-sha256-sha1!
conn marketing1
     keyexchange=ikev2
     left=192.168.1.228
     leftid=218.17.157.34
     leftsubnet=192.168.0.0/21
     right=129.204.65.204
     rightid=129.204.65.204
     rightsubnet=10.125.0.0/16
     auto=route
     ike=aes-sha1-modp1024
     ikelifetime=86400s
     esp=aes-sha1-modp1024
     lifetime=86400s
     type=tunnel
```

其中 config setup 只能出现一次，而 conn &lt;连接名称&gt; 可以有很多个。这里的名称不是预定义的，可以随意写，只要你能识别就行，主要用来定义一种连接，就是为了让你在日志里好找。

新版 strongswan 里 config setup 的内容不如旧版的多，许多旧版必须有的选项都被作废了。比如：

* plutostart 新版所有的 ike 协议都由原来 ikev2 的 daemon：charon 接管。根本就没有 pluto 了。
* nat\_traversal 新版所有的 ike 协议都是可以穿越路由器（NAT）的。
* virtual\_private 定义服务器的局域网 IP 地址。现在被魔术字 0.0.0.0/0 取代了。
* pfs 完美向前保密，用于 rekey 时。意思是你现在的金钥互换过程如果被攻破了，会不会对已经互换过的金钥产生影响。以前一般设置成 no 来适用于 iOS 这种客户端会以积极模式发出非加密的 rekey 请求的情况。现在这个选项完全没作用了。第一阶段永远是完美向前保密的，第二阶段（esp）如果指定了 DH 组那么也是完美向前保密的，但是默认加密方法就已经指定了 DH 组。所以该选项永远为 yes。

**更多说明查考** [https://zh.opensuse.org/index.php?title=SDB:Setup\_Ipsec\_VPN\_with\_Strongswan&variant=zh](https://zh.opensuse.org/index.php?title=SDB:Setup_Ipsec_VPN_with_Strongswan&variant=zh)

密码文件ipsec.secrets文件默认不存在，需要自己生成。  
vi /etc/strongswan/ipsec.secrets

```text
# ipsec.secrets - strongSwan IPsec secrets file
218.17.157.34 10.125.0.19 : PSK 68p63kt6
192.168.1.228 10.125.0.19 : PSK 68p63kt6
```

#### CentOS配置转发

```text
vim /etc/sysctl.conf

net.ipv4.ip_forward = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

sysctl -p 
```

CentOS配置iptables防火墙，允许UDP500，UDP4500，并配置转发和NAT规则

若是开启访问，请参考配置 Iptables 转发

### 参考：配置 Iptables 转发

```text
iptables -A INPUT -p udp --dport 500 -j ACCEPT
iptables -A INPUT -p udp --dport 4500 -j ACCEPT
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 192.168.0.0/21 -o eth0 -j MASQUERADE  #地址与上面地址池对应
iptables -A FORWARD -s 192.168.0.0/21 -j ACCEPT     #同上
#为避免VPS重启后NAT功能失效，可以把如上5行命令添加到 /etc/rc.local 文件中，添加在exit那一行之前即可。
```

#### 公司内网设置路由

把去往10.125.0.0/16段路由代理到192.168.1.228

![](../.gitbook/assets/image%20%2817%29.png)

![](http://doc.maizuo.com/api/file/getImage?fileId=5c10c046915399000d0001aa)

#### 启动

```text
systemctl enable strongswan.service
systemctl start strongswan.service
```

### 

### 2.公司内网10.125.0.19上配置说明

#### vim /etc/strongswan/ipsec.conf

```text
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration
# Add connections here.

# Sample VPN connections

config setup
       uniqueids = never
       charondebug="cfg 2, dmn 2, ike 2, net 2"
conn %default
     authby=psk
     type=tunnel
conn marketing
    keyexchange=ikev2
    left=10.125.0.19
    leftid=129.204.65.204
    leftsubnet=10.125.0.0/16
    right=218.17.157.34
    rightid=218.17.157.34
    rightsubnet=192.168.0.0/21
    ike=aes-sha1-modp1024
    auto=route
    ikelifetime=86400s
    esp=aes-sha1-modp1024
    lifetime=86400s
    type=tunnel
```

密码文件ipsec.secrets文件默认不存在，需要自己生成。

####  vi /etc/strongswan/ipsec.secrets

```text
# ipsec.secrets - strongSwan IPsec secrets file
10.125.0.19 218.17.157.34 : PSK 68p63kt6
129.204.65.204 218.17.157.34 : PSK 68p63kt6
```

#### CentOS配置转发

```text
vim /etc/sysctl.conf

net.ipv4.ip_forward = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

sysctl -p 
```

CentOS配置iptables防火墙，允许UDP500，UDP4500，并配置转发和NAT规则

参考

```text
# 停用firewalld，启用iptables
systemctl stop firewalld
systemctl disable firewalld
yum install -y iptables-services iptables-devel
systemctl enable iptables
systemctl start iptables
systemctl status iptables

# 启用转发
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
# 添加nat
iptables -t nat -A POSTROUTING -j MASQUERADE

# 设置允许转发。默认是拒绝所有转发，-I 插入一下规则
iptables -I FORWARD -s 192.168.0.0/16 -j ACCEPT
iptables -I FORWARD -d 192.168.0.0/16 -j ACCEPT
service iptables save
```

#### 腾讯内网安全组放开4500和500

#### 启动

```text
systemctl enable strongswan.service
systemctl start strongswan.service
```

## 三、检查

自此vpn配置完毕，可以互ping进行检查

排查故障

tcpdump -i enp2s0 -nNX -s0 port 4500 or 500  查看有没有发包进行连接

## 查考文档

扩展GRE VPN: [https://blog.csdn.net/zhydream77/article/details/81051429](https://blog.csdn.net/zhydream77/article/details/81051429)

[https://gist.github.com/losisli/11081793](https://gist.github.com/losisli/11081793)

[https://zh.opensuse.org/index.php?title=SDB:Setup\_Ipsec\_VPN\_with\_Strongswan&variant=zh](https://zh.opensuse.org/index.php?title=SDB:Setup_Ipsec_VPN_with_Strongswan&variant=zh)

{% embed url="https://blog.csdn.net/tty521/article/details/79350412" %}

[https://zh.opensuse.org/index.php?title=SDB:Setup\_Ipsec\_VPN\_with\_Strongswan&variant=zh](https://zh.opensuse.org/index.php?title=SDB:Setup_Ipsec_VPN_with_Strongswan&variant=zh)

