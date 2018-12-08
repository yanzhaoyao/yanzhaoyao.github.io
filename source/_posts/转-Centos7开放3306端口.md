---
title: [转]Centos7开放3306端口
typora-root-url: 转-Centos7开放3306端口
typora-copy-images-to: 转-Centos7开放3306端口
date: 2018-12-07 22:24:27
tags: 
- Centos7
- Linux
categories: Linux
---

在 Centos 7 或 RHEL 7 或 Fedora 中防火墙由 firewalld 来管理，而不是 iptables。

------

## 一、firewalld 防火墙

语法命令如下：启用区域端口和协议组合

```
firewall-cmd [--zone=<zone>] --add-port=<port>[-<port>]/<protocol> [--timeout=<seconds>]
```

此举将启用端口和协议的组合。 
端口可以是一个单独的端口 `<port>` 或者是一个端口范围 `<port>-<port>`。 
协议可以是 tcp 或 udp。

**查看 firewalld 状态**

```
systemctl status firewalld
```

![20180313152027607](/20180313152027607.png)

**开启 firewalld**

```
systemctl start firewalld
```

![这里写图片描述](/2018031315252841.png)
**开放端口**

```
// --permanent 永久生效,没有此参数重启后失效
firewall-cmd --zone=public --add-port=80/tcp --permanent 

firewall-cmd --zone=public --add-port=1000-2000/tcp --permanent
```

![这里写图片描述](/20180313150650743.png)
**重新载入**

```
firewall-cmd --reload
```

![这里写图片描述](/20180313150909843.png)
**查看**

```
firewall-cmd --zone=public --query-port=80/tcp
```

![这里写图片描述](/20180313150931669.png)
**删除**

```
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

## 二、iptables 防火墙

也可以还原传统的管理方式使用 iptables

```
systemctl stop firewalld  
systemctl mask firewalld
```

**安装 iptables-services**

```
yum install iptables-services  
```

**设置开机启动**

```
systemctl enable iptables
```

**操作命令**

```
systemctl stop iptables  
systemctl start iptables  
systemctl restart iptables  
systemctl reload iptables 
```

**保存设置**

```
service iptables save
```

**开放某个端口 在 /etc/sysconfig/iptables 里添加**

```
-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
```

作者：空山冥卫 
来源：CSDN 
原文：https://blog.csdn.net/weiyangdong/article/details/79540217 
版权声明：本文为博主原创文章，转载请附上博文链接！