# 进程管理

## 进程的概念与进程查看

### 进程的查看命令

- 查看命令
  - ps
   -elf
  - pstree
  - top

- 结论
  - 进程也是树形结构
  - 进程和权限有着密不可分的关系

Linux下有3个特殊的进程，idle进程(PID = 0), init进程(PID = 1)和kthreadd(PID = 2)

idle进程由系统自动创建, 运行在内核态

idle进程其pid=0，其前身是系统创建的第一个进程，也是唯一一个没有通过fork或者kernel_thread产生的进程。完成加载系统后，演变为进程调度、交换

init进程由idle通过kernel_thread创建，在内核空间完成初始化后, 加载init程序, 并最终用户空间

由0进程创建，完成系统的初始化. 是系统中所有其它用户进程的祖先进程
Linux中的所有进程都是有init进程创建并运行的。首先Linux内核启动，然后在用户空间中启动init进程，再启动其他系统进程。在系统启动完成完成后，init将变为守护进程监视系统其他进程。

## 进程的控制命令

- 调整优先级
  - nice 范围从-20到19，值越小优先级越高，抢占资源就越多
  - renice 重新设置优先级
- 进程的作业控制
  - jobs
  - &符号
    - ./a.sh & 执行后台任务
    - jobs: 查看后台任务获取编号
    - fg 编号:调到前台
    - bg 编号:调到后台

## 进程的通信方式——信号

- kill -l 查看所有信号
  - SIGINT 通知前台进程组终止进程 ctrl + c
  - SIGKILL 立即结束程序，不能被阻塞和处理 kill -9 pid

## 守护进程和系统日志

- 使用 nohup 与 & 符号配合命令运行一个命令
  - nohup 命令使进程忽略 hangup（挂起）信号
- 守护进程（daemon）和一般进程有什么差别呢？
- 使用 screen 命令
  - screen 进入 screen 环境
  - ctrl +a d 退出（detached）screen环境
  - screen -ls 查看 screen 的会话
  - screen -r sessionid 恢复会话

## 服务管理工具 systemctl

- 服务（提供常见功能的守护进程）集中管理工具
  - service 3.6以前
  - systemctl
    - systemctl start | stop | restart | reload | enable | disable 服务名称
    - 软件包安装的服务单元 /usr/lib/system/system/

## SELinux简介

- MAC（强制访问控制）与DAC（自主访问控制）
- 查看 SELinux 的命令
  - getenforce
  - /usr/sbin/sestatus
  - ps -Z and ls -Z and id -Z
- 关闭 SELinux
  - setenforce 0
  - /etc/selinux/sysconfig