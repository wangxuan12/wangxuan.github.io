# 服务管理



## 防火墙



### 防火墙分类

- iptables 的表和链
- iptables 的filter表
- iptables 的nat表
- iptables 配置文件
- firewallD 服务



#### 软件防火墙

- 包过滤防火墙和应用层防火墙
  - CentOS 6 默认是iptables
  - CentOS 7 默认是firewallD（底层使用netfilter）



#### 硬件防火墙

- 主要防一些如DOS类的流量攻击，也兼顾一些包过滤的功能



## iptables 的表和链

- 规则表
  - filter nat mangle raw
- 规则链
  - INPUT OUTPUT FORWARD
  - PREROUTING POSTROUTING



### iptables的filter表

- iptables -t filter 命令 规则链 规则
  - 命令
    - -L 查看规则
    - -A -I  添加规则 一个在后追加，一个插到最前
    - -D -F -P
      - -D 删除规则
      - -F 清除所有规则，不影响默认规则
      - -P 修改默认规则也就是policy规则
    - -N -X -E 这三条命令分别是 增加自定义规则，删除自定义规则和重命名自定义规则
  - iptables -vnL 查看所有的规则信息
  - iptables -A INPUT -i eth0 -s 10.0.0.2/24 -p tcp --dport 80 -j ACCEPT



### iptables的nat表

- iptables -t nat 命令 规则链 规则
  - PREROUTING 目的地址转换
  - POSTROUTING 源地址转换



### firewallD服务

- firewallD特点
  - 支持区域 “zone” 概念
  - firewall-cmd
- systemctl start | stop | enable | disable  firewall.service



### SSH

- 常用命令
  - ssh-keygen -t rsa 生成密钥
  - ssh-copy-id 拷贝公钥到服务器
    - ssh-copy-id -i /root/.ssh/id_rsa.pub root@19.0.0.1
  - 远程拷贝
    - scp a.txt root@10.0.0.1:/tmp/
    - scp root@10.0.0.1:/tmp/a.txt /tmp/k.txt



### FTP服务



### Samba和NFS

- 常见共享服务的区别
- Samba服务的安装
- Samba服务的配置文件
- Samba用户的设置
- Samba服务的服务和停止
- NFS服务的配置
- NFS服务的启动和停止



### Nginx

- Nginx和Web服务介绍
  - Nginx是一个高性能的Web和反向代理服务器
  - 支持Http、Https和电子邮件代理协议
  - OpenResty是基于Nginx和Lua实现的Web应用网关，集成了大量的第三方模块
- OpenResty软件下载和安装
- OpenResty的配置文件
- 使用OpenResty配置域名虚拟主机



### LNMP

- Linux + Nginx + MySQL + PHP



### DNS服务

- DNS服务介绍
  - DNS（Domain Name System）域名系统
  - FQDN（Full Qualified Domain Name） 完全限定域名
  - 域分类：根域、顶级域（TLD）
  - 查询方式：递归、迭代
  - 解析方式：正向解析、反向解析
  - DNS服务器的类型：缓存域名服务器、主域名服务器、从域名服务器
- BIND软件的安装
- BIND的配置文件
- 使用 dig 和 nslookup 命令测试 DNS
- 从域名服务器的配置



### NAS

- NAS（Network Attached Storage）网络附属存储
- NAS 支持的协议 NFS、CIFS、FTP
- 保证数据安全方式 磁盘阵列

