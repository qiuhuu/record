# 同物理机的docker容器之间可以ping通但是无法访问端口

#解决方案，把docker0网卡添加到trusted域

```sh
firewall-cmd --permanent --zone=trusted --change-interface=docker0
firewall-cmd --reload
```

然后关闭所有容器

```sh
docker stop $(docker ps -aq)
```

重启docker服务

```sh
systemctl restart docker.service
```


