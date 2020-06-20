# 内存与磁盘管理


## 内存和磁盘使用率查看

- 内存使用率查看常用命令
  - free
    - free -m 以mb单位显示
    - free -g 以gb单位显示
  - top
- 磁盘使用率查看常用命令
  - fdisk -l fdisk命令不要轻易使用
  - df
  - du 实际大小 不含空洞 du查看的是块数据（大小4k）
  - du 与 ls 的区别 统计大小不同 ls包含空洞 ls查看的是i节点个数

## ext4(old) fxs(new) 文件系统

- 超级块
- 超级块副本
- i 节点（inode）
  - rm 删除文件 其实就是断开文件与i节点之间的链接
  - ls -li 可以查看到文件对应的i节点
- 数据块（datablock）
- ln afile bfile 把bfile链接到afile链接的i节点
- ln -s 软链接(符号链接) 
  - b 和 a是不同的i节点，b指向a的路径，对b的权限修改都会反应到a上，
  - 可以跨越文件系统
- facl 文件访问控制列表
  - getfacl 查看文件访问控制列表
  - setfacl -m u:user1:r afile
    - -m 赋予
    - -x 收回

## 磁盘配额的使用

## 磁盘的分区与挂载

- 常用命令
  - fdisk fdisk命令不要轻易使用
  - mkfs
  - parted
  - mount
- 常见配置文件
  - /etc/fstab

## 文件分区（虚拟内存）的查看与创建

- 增加交换分区的大小
  - mkswap
  - swapon
- 使用文件制作交换分区
  - dd if = /def/zero bs=4M count=1024 of=/swapfile

## 软件 RAID 的使用

- RAID 的常见级别及含义
  - RAID 0 striping 条带方式，提高单盘吞吐率
  - RAID 1 mirroring 镜像方式，提高可靠性
  - RAID 5 有奇偶校验
  - RAID 10 是RAID 1 与 RAID 0 的结合
- 软件 RAID 的使用

## 逻辑卷管理

- 逻辑卷和文件系统的关系
- 为Linux创建逻辑卷
- 动态扩容逻辑卷

## 系统综合状态查看

- 使用 sar 命令查看系统综合状态
  - u 1 10  查看cpu状态 每秒采样一次 总计10次
  - r 查看内存
  - b 查看io
  - d 查看磁盘
  - q 查看进程
- 使用第三方命令查看网络流量
  - yum install epel-release
  - yum install iftop
  - iftop -P 查看网络