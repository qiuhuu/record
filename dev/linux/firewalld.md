# 智障解法: 关闭防火墙

1. 操作

	```shell
	# 1. 验证 防火墙 是否打开
	> systemctl status firewalld
	# 2. 关闭 防火墙
	> systemctl stop firewalld
	# 3. 验证 防火墙 是否打开
	> systemctl status firewalld
	```

2. 结果

	1. 防火墙关闭成功
	2. 宿主机 访问 虚拟机 80 端口成功

# 常规解法: 让 firewalld 开放 80端口

## 1. 确认 firewalld 是否打开

## 2. 查看 firewalld 的 开放端口列表

1. 概述

	1. 查看开放端口

2. 命令

	```shell
	> firewall-cmd --list-ports
	```

## 3. 将 80/tcp 添加到 开放端口列表中

1. 命令

	```shell
	# 1. 添加端口 --permanent永久生效
	> firewall-cmd --zone=public --add-port=80/tcp --permanent
	
	# 2. 重启 firewalld
	> firewall-cmd --reload
	
	# 3. 查看是否生效
	# 80/tcp 加入了列表
	> firewall-cmd --list-ports
	```