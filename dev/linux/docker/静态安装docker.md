### 安装静态二进制文件

1. 下载静态二进制存档。前往 https://download.docker.com/linux/static/stable/ （或更改`stable`为`nightly`或`test`），选择您的硬件平台，并下载`.tgz`与您要安装的 Docker Engine 版本相关的文件。

2. 使用该`tar`实用程序提取存档。在`dockerd`和`docker` 二进制文件被提取。

	```shell
	tar xzvf /path/to/<FILE>.taz
	```

3. **可选**：将二进制文件移动到可执行路径上的目录，例如`/usr/bin/`. 如果跳过此步骤，则必须在调用`docker`或`dockerd`命令时提供可执行文件的路径。

	```powershell
	sudo cp docker/* /usr/bin/
	```

4. 启动 Docker 守护进程：

	```shell
	sudo dockerd &
	```

	如果您需要使用其他选项启动守护程序，请相应地修改上述命令或创建并编辑文件`/etc/docker/daemon.json` 以添加自定义配置选项。

5. 通过运行`docker version` 映像验证 Docker 是否已正确安装。



docker加入服务

```bash
vi /lib/systemd/system/docker.service
```

`docker.service`默认内容如下：

```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
BindsTo=containerd.service
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target
```

`docker.service`文件被修改后请执行`systemctl daemon-reload && systemctl restart docker`，如果配置未生效，请执行`systemctl status docker`查看服务状态。