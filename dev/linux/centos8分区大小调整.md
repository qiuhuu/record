# centos8减少/home分区大小增加/root空间大小

步骤：

1. 使用df-h查看空间使用情况，备份home
2. 卸载home文件系统
3. 删除/home所在的lv
4. 扩展/root所在的lv
5. 扩展/root文件系统
6. 重新创建home lv并挂载home
7. 查看最终调整结果

## 1、使用df-h查看空间使用情况，备份home

首先登陆ssh，使用df -lh查看空间使用情况，有文件注意备份文件

```shell
df -lh
```

备份/home文件

```shell
#备份home文件到/tmp目录，不需要备份则运行命令
#tar cvf /tmp/home.tar /home
#zip -r /tmp/home.zip /home
```

## 2、卸载home文件系统

​	卸载home文件系统，解除home目录的占用，卸载home目录

```shell
#接触目录占用
fuser -km /home/
#卸载home
umount /home
```

## 3、删除/home所在的lv

​	删除/home所在的lv，这一步centos8有很大不同，因为centos7中目录是/dev/mapper/centos-home,而在centos8中为 /dev/mapper/cl-home，因此注意卸载设备名称

```shell
lvremove /dev/mapper/cl-home
```

## 4、扩展/root所在的lv

​	扩展/root所在的空间lv，增加50G，注意cl-root的名称

```shell
lvextend -L +50G /dev/mapper/cl-root
```

## 5、扩展/root文件系统

​	扩展/root文件系统，这一步是真正增加root空间，centos7和centos8具有非常大的差别，centos7中是使用xfs_growfs /dev/mapper/centos-root，按逻辑centos8就应该是 xfs_growfs /dev/mapper/cl-root，但是结果就是

```shell
xfs_growfs /dev/mapper/cl-root 

xfs_growfs / 
```

## 6、重新创建home lv并挂载home

<font color=red>注意：注意cl 在不同机器上可能有不同的写法<font>

重新创建home lv并挂载home，创建20g空间的home，

```shell
lvcreate -L 20G -n home cl
```


lvcreate_1g_home 文件系统类型设置

```shell
mkfs.xfs /dev/cl/home
```


挂载到home目录

```shell
mount /dev/cl/home /home
```

恢复home目录下文件

```shell
mv /tmp/home.tar /home

cd /home

tar xvf  home.tar

mv home/* .

#rm -rf home*
```

## 7、查看最终调整结果

查看最终调整结果，查看各分区大小

```shell
df -lh
```

