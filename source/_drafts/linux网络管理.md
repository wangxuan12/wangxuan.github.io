# 网络管理

## 网络状态查看

### net-tools

- ifconfig
- route
- netstat

### iproute2

- ip
- ss

### 查看网卡物理链接情况

mii-tool eth0

### 查看网关

route -n
使用-n参数不解析主机名

## 网络配置

- ifconfig<接口><IP地址>[netmask 子网掩码]
- ifup<接口>
- ifdown<接口>

### 添加网关

- route add default gw<网关ip>
- route add -host<指定ip> gw<网关ip>
- route add -net<指定网段>netmask<子网掩码> gw<网关ip>

## 路由命令

## 网络故障排除

- ping
- traceroute
- mtr
- nslookup
- telnet
- tcpdump
- netstat
- ss

## 网络服务管理

## 常用网络配置文件

源代码编译安装 举例
wget https://..../openresty-1.15.8.1.tar.gz
tar -zxf openresty-VERSION.tar.gz
cd openresty-VERSION
./configure --prefix=/usr/local/openresty
make -j2
make install
