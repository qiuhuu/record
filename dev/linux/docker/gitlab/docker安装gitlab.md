## 1、拉取gitlab镜像

`docker pull gitlab/gitlab-ce`

## 2、设置卷位置(非必须)

在设置其他所有内容之前，请配置一个新的环境变量`$GITLAB_HOME` ，该变量指向配置，日志和数据文件将驻留的目录。确保该目录存在，并且已授予适当的权限。

对于Linux用户，将路径设置为`/srv/gitlab`：

```
export GITLAB_HOME=/srv/gitlab
```

对于macOS用户，请使用用户`$HOME/gitlab`目录：

```
export GITLAB_HOME=$HOME/gitlab
```

GitLab容器使用主机安装的卷来存储持久性数据：

| 宿主机位置            | 容器位置          | 用法                     |
| :-------------------- | :---------------- | :----------------------- |
| `$GITLAB_HOME/data`   | `/var/opt/gitlab` | 用于存储应用程序数据。   |
| `$GITLAB_HOME/logs`   | `/var/log/gitlab` | 用于存储日志。           |
| `$GITLAB_HOME/config` | `/etc/gitlab`     | 用于存储GitLab配置文件。 |



## 3、使用Docker Engine安装GitLab

您可以微调这些目录以满足您的要求。设置`GITLAB_HOME`变量后，即可运行该映像：

```
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  gitlab/gitlab-ee:latest
```

这将下载并启动一个GitLab容器，并发布访问SSH，HTTP和HTTPS所需的端口。所有的GitLab数据都将存储为的子目录 `$GITLAB_HOME`。`restart`系统重启后，容器将自动运行。

确保Docker进程具有足够的权限以在已安装的卷中创建配置文件。