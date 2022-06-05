1. 上传jar到服务器的指定目录

2. 在该目录下创建Dockerfile 文件

    ```shell
      vi Dockerfile
    ```

    然后将下面的内容复制到Dockerfile文件中

```shell
FROM jdk8 
MAINTAINER qiuhuu
ADD demo-0.0.1-SNAPSHOT.jar demo.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","demo.jar"]

from jdk8  拉取一个jdk为8的docker image

maintainer  作者是qiuhuu

demo-0.0.1-SNAPSHOT.jar 就是你上传的jar包，替换为jar包的名称

demo.jar  是你将该jar包重新命名为什么名称，在容器中运行

expose  该容器暴露的端口是多少，就是jar在容器中以多少端口运行

entrypoint 容器启动之后执行的命令，java -jar demo.jar  即启动jar
```

 

   4. 创建好Dockerfile文件之后，执行命令 构建镜像：

      ```shell
      docker build -t demo .
      
      注意最后的 .  表示 Dockerfile 文件在当前目录下
      
      demo  构建之后镜像名称
      ```
      
      

 

   5. 镜像构建成功之后，就可以运行容器了：

       ```
       docker run -d --name demo -p 8080:8080 demo
       
       docker run -d --restart=always --name demo -p 8080:8080  demo   
       --restart=always 这个表示docker容器在停止或服务器开机之后会自动重新启动 
       ```
       
       然后`docker ps` 看看你的容器有没有在运行即可 

  7. docker logs --tail  300 -f  demo  查看启动日志 

   

另： 如果docker run 的时候没有加 --restart=always ，然后已经运行的docker容器怎么设置自动重启？ 执行下面命令：

   docker update –-restart=always demo 

 demo : 你的容器名称