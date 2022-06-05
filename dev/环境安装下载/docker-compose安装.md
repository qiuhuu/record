### 在Linux系统上安装

在Linux上，您可以从[GitHub上](https://github.com/docker/compose/releases)的[Compose存储库发行页面](https://github.com/docker/compose/releases)下载Docker Compose二进制文件 。按照链接中的说明进行操作，其中包括`curl`在终端中运行命令以下载二进制文件。这些分步说明也包含在下面。

> 对于`alpine`，需要以下依赖包： `py-pip`，`python-dev`，`libffi-dev`，`openssl-dev`，`gcc`，`libc-dev`，和`make`。

1. 运行以下命令以下载Docker Compose的当前稳定版本：

	```sh
	sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
	```

	> 要安装其他版本的Compose，请替换`1.26.2` 为要使用的Compose版本。

	如果使用进行安装时遇到问题`curl`，请参见 上方的“ [备用安装选项”](https://docs.docker.com/compose/install/#alternative-install-options)标签。

2. 将可执行权限应用于二进制文件：

	```sh
	sudo chmod +x /usr/local/bin/docker-compose
	```

> **注意**：如果命令`docker-compose`在安装后失败，请检查您的路径。您也可以创建指向`/usr/bin`或路径中任何其他目录的符号链接。

例如：

```sh
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

1. （可选）为 和shell 安装[命令完成](https://docs.docker.com/compose/completion/)。`bash` `zsh`

2. 测试安装。

	```sh
	$ docker-compose --version
	docker-compose version 1.26.2, build 1110ad01
	```

------

## 安装预发行版本

如果想试用预发布版本，可以从[GitHub上](https://github.com/docker/compose/releases)的[Compose存储库发布页面](https://github.com/docker/compose/releases)下载候选`releases`。按照链接中的说明进行操作，该链接涉及`curl`在终端中运行命令以下载二进制文件。

也可以从https://dl.bintray.com/docker-compose/master/下载从“ master”分支构建的预发行版本 。

> 发行前的版本使您可以在发行新功能之前对其进行试用，但可能会使其不稳定。

## 升级

如果要从Compose 1.2或更早版本进行升级，请在升级Compose之后删除或迁移现有容器。这是因为从1.3版开始，Compose使用Docker标签来跟踪容器，并且需要重新创建容器以添加标签。

如果Compose检测到创建的没有标签的容器，它将拒绝运行，这样就不会得到两组标签。如果要继续使用现有容器（例如，因为它们具有要保留的数据量），则可以使用Compose 1.5.x通过以下命令迁移它们：

```sh
docker-compose migrate-to-labels
```

另外，如果您不担心保留它们，可以将其删除。撰写只是创建新的。sh

```sh
docker container rm -f -v myapp_web_1 myapp_db_1 ...
```

## 卸载

如果使用`curl`以下命令进行安装，则要卸载Docker Compose ：

```sh
sudo rm /usr/local/bin/docker-compose
```

如果使用`pip`以下命令进行安装，则要卸载Docker Compose ：

```sh
pip uninstall docker-compose
```

> 遇到“权限被拒绝”错误？
>
> 如果使用以上两种方法中的任何一种都会出现“权限被拒绝”错误，则可能没有适当的权限删除 `docker-compose`。要强制删除，请先`sudo`执行以上任一命令，然后再次运行。