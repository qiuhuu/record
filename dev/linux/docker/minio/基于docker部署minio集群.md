# 基于docker部署minio集群

1. 首先准备 2-4台服务器（官网建议集群4台），我这里仅测试部署，所以准备了两台服务器

   | ip              | name    |
   | --------------- | ------- |
   | 192.168.178.132 | minio-1 |
   | 192.168.178.133 | minio-2 |
   |                 |         |

2. 修改 `/etc/hosts`

   `vi /etc/hosts`

   `192.168.178.132  minio-1`

   `192.168.178.133  minio-2`

3. 同步服务器时间，关闭服务器防火墙，关闭selinux

   关闭防火墙：（如果不关闭，那么请自行检查节点之间网络，网络不通会引起大量问题）

   ```shell
   systemctl stop firewalld
   systemctl disable firewalld
   ```

   关闭selinux：

   ```shell
   vi /etc/sysconfig/selinux
   #将SELINUX=enforcing指令更改为SELINUX=disabled
   ```

   保存并重启

   `sudo shutdown -r now`

   重启后查看selinux状态

   `sestatus`

4. 服务器都部署docker，并pull minio

   `docker pull minio/minio`

5. 每个节点执行命令，运行Minio

   ```shell
   docker run -d --network=host --name minio \
           --restart=always \
           --log-opt max-size=10m \
           -v /etc/timezone:/etc/timezone \
           -v /etc/localtime:/etc/localtime \
           -v /data/minio/data1:/data1 \
           -v /data/minio/data2:/data2 \
           -e "MINIO_ROOT_USER=admin" \
           -e "MINIO_ROOT_PASSWORD=admin123" \
           -e "MINIO_ACCESS_KEY=minio" \
           -e "MINIO_SECRET_KEY=minio123" \
           minio/minio server http://minio-{1...2}/data{1...2}
   ```

   一个两个节点，每个节点挂载2个目录

   注意： `network`一定要用 `host` 模式，密码长度最小8位

   

6. 启动完成运行，查看minio日志是否运行成功。

   1. 出现以下日志表示启动成功：

      ```log
      Formatting 1st pool, 1 set(s), 4 drives per set.
      Waiting for all MinIO sub-systems to be initialized.. lock acquired
      Automatically configured API requests per node based on available memory on the system: 8
      All MinIO sub-systems initialized successfully in 49.902533ms
      Waiting for all MinIO IAM sub-system to be initialized.. lock acquired
      Finished loading IAM sub-system (took 0.0s of 0.0s to load data).
      Status:         4 Online, 0 Offline. 
      API: http://192.168.178.132:9000  http://172.17.0.1:9000  http://127.0.0.1:9000         
      
      Console: http://192.168.178.132:41044 http://172.17.0.1:41044 http://127.0.0.1:41044     
      
      Documentation: https://docs.min.io
      
      WARNING: Console endpoint is listening on a dynamic port (41044), please use --console-address ":PORT" to choose a static port.
      ```

   2. 错误：检查其他节点未运行，或网络不通，请检查防火墙是否关闭或开启防火墙状态下端口是否开放

      ```log
      API: SYSTEM()
      Time: 13:55:39 UTC 05/26/2022
      Error: Marking http://minio-2:9000/minio/storage/data1/v45 temporary offline; caused by Post "http://minio-2:9000/minio/storage/data1/v45/readall?disk-id=&file-path=format.json&volume=.minio.sys": dial tcp 192.168.178.133:9000: connect: no route to host (*fmt.wrapError)
             7: internal/logger/logger.go:278:logger.LogIf()
             6: internal/rest/client.go:151:rest.(*Client).Call()
             5: cmd/storage-rest-client.go:153:cmd.(*storageRESTClient).call()
             4: cmd/storage-rest-client.go:542:cmd.(*storageRESTClient).ReadAll()
             3: cmd/format-erasure.go:387:cmd.loadFormatErasure()
             2: cmd/format-erasure.go:326:cmd.loadFormatErasureAll.func1()
             1: internal/sync/errgroup/errgroup.go:123:errgroup.(*Group).Go.func1()
      Client http://minio-2:9000/minio/storage/data1/v45 online
      Waiting for all other servers to be online to format the disks (elapses 2m44s)
      ```

   3. 错误：检查目录是否都挂载出来，检查其他节点是否未运行，或网络不通，请检查防火墙是否关闭或开启防火墙状态下端口是否开放，请确认挂载目录是否有权限

      ```log
      API: SYSTEM()
      Time: 13:55:35 UTC 05/26/2022
      Error: Read failed. Insufficient number of disks online (*errors.errorString)
             6: internal/logger/logger.go:278:logger.LogIf()
             5: cmd/prepare-storage.go:242:cmd.connectLoadInitFormats()
             4: cmd/prepare-storage.go:302:cmd.waitForFormatErasure()
             3: cmd/erasure-server-pool.go:93:cmd.newErasureServerPools()
             2: cmd/server-main.go:654:cmd.newObjectLayer()
             1: cmd/server-main.go:505:cmd.serverMain()
      Waiting for a minimum of 2 disks to come online (elapsed 2m39s)
      
      ```

      

7. 访问成功日志的`Console`地址

