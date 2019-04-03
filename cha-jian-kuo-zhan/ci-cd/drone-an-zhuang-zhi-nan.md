# DRONE安装指南

在上文《[基于Docker的CI工具—Drone](https://imnerd.org/drone.html)》中我们介绍了 Drone 以及它的优缺点，下面我将来介绍下如何配置安装 Drone。官方推荐使用 [docker-compose](https://docs.docker.com/compose/) 来启动服务，所以首先我们要准备好 docker 和 docker-compose。

### 安装 docker

docker 服务提供了桌面版，服务器版和云服务版不同操作系统的多种支持，可以在[官方文档](https://docs.docker.com/engine/installation/#supported-platforms)中找到对应的安装方法。这里以 [Debain 服务器安装](https://docs.docker.com/engine/installation/linux/docker-ce/debian/)为例。Docker 支持 Debian 7.7+ 以上的系统版本， 需要 3.10 版本以上的 Linux 内核环境。

首先我们添加 docker 源的密钥：

```text
$ sudo apt-get update$ sudo apt-get install \     apt-transport-https \     ca-certificates \     curl \     gnupg2 \     software-properties-common$ curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
```

然后我们添加 docker 源并使用 apt 安装 docker：

```text
$ sudo add-apt-repository \   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \   $(lsb_release -cs) \   stable"$ sudo apt-get update$ sudo apt-get install docker-ce
```

执行以下命令确保我们已经安装正确：

```text
$ sudo docker run hello-world
```

相比 docker 的安装，docker-compose 的安装就比较简单了。它是用 python 写的一个 docker 多镜像编排启动的工具，我们可以直接使用 `pip` 来安装它：

```text
$ pip install docker-compose
```

\#\# Git仓库准备

CI 可持续构建的一个大前提是基于仓库的，所有的构建第一步都需要去仓库拉取代码。Drone 支持 [Github](http://docs.drone.io/install-for-github/)，[Gitlab](http://docs.drone.io/install-for-gitlab/)，[Bitbucket](http://docs.drone.io/install-for-bitbucket-cloud/)，[Gogs](http://docs.drone.io/install-for-gogs/)， [Gitea](http://docs.drone.io/install-for-gitea/), [coding](http://docs.drone.io/install-for-coding/) 多个仓库类型，其中支持 OAuth 的仓库需要在网站申请密钥。这里以 Github 为例。在 [New Oauth Application](https://github.com/settings/applications/new) 页申请密钥，其中 callback URL 需要填写为`<你的CI地址>/authorize`。申请好后记录下你的 client id 和 client secret 以便后续使用。

![github oauth apply](http://docs.drone.io/images/github_oauth.png)

### 创建 compose 配置文件

我们新建一个空目录防止我们的配置文件文件：

```text
$ mkdir drone$ cd drone$ vim docker-compose.yml
```

在配置文件中，我们设置 `docker-compose.yml` 的格式为 2 号版本，定义两种服务。

* `drone-server` ：使用 `drone/drone:0.8.2` 版本镜像，将启动监听 3800 上的主 Drone 服务容器，9000 端口来开放给 Agent。我们将在容器内挂载 `/etc/drone` 目录，以便 drone 可以保留数据。配置服务自动重新启动，并添加一些环境变量。
* `drone-agent`：使用 `drone/agent:0.8.2` 版本镜像，将 docker 启动句柄挂载到容器 `/var/run/docker.sock` 文件中，以便 drone 可以使用 docker 来执行镜像构建任务。环境变量中需要配置 drone server 的地址以及 server 的密钥，以便于 server 进行通信。

```text
version: '2'
services:
  drone-server:
    image: drone/drone:0.8.2
    ports:
      - 3800:8000
      - 9000
    volumes:
      - /etc/drone:/var/lib/drone/
    restart: always
    environment:
      # 是否允许注册，false 后只有在 DRONE_ADMIN 变量中指定的账户才能登录
      - DRONE_OPEN=true
      # Drone可公开访问的地址
      - DRONE_HOST=http://ci.eming.li
      # 配置 Git 仓库，只能同时使用一种仓库
      # Github 仓库需要配置上问申请到的 client id 和 client secret
      - DRONE_GITHUB=true
      - DRONE_GITHUB_CLIENT=003bd43da3380e8d7982
      - DRONE_GITHUB_SECRET=9f4a76bf7c70e5caf1120af5db1b3dcd8db5f209
      # Gogs 配置仓库地址即可
      - DRONE_GOGS=false
      - DRONE_GOGS_URL=http://git.eming.li
      
      # Drone Server 和 Agent 的通信密钥
      - DRONE_SECRET=4a6a6J8LJ4Wc82Ukjww1634ev
  drone-agent:
    image: drone/agent:0.8.2
    command: agent
    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      # 配置 SERVER 地址
      - DRONE_SERVER=drone-server:9000
      # 配置与 SERVER 通信的密钥，需要与 Server 配置的保持一致
      - DRONE_SECRET=4a6a6J8LJ4Wc82Ukjww1634ev
```

其中的通信密钥相当于 drone 的密码，最好为一个长传的随机字串防止被破解。可以在命令行中使用一下内容生成：

```text
$ LC_ALL=C </dev/urandom tr -dc A-Za-z0-9 | head -c 65 && echo
```

配置文件完成之后，我们就可以使用以下命令启动服务了：

```text
$ docker-compose -f /etc/drone/docker-compose up
```

docker-compose 会自动帮我们去下载镜像并根据配置初始化容器。一切就绪之后，我们使用 `<host>:3800` 就可以访问到 drone 了。

### 配置 Nginx

带端口访问总是让人不爽，老方法我们配置下 Nginx 做下反向代理就可以了，以下是具体的 nginx 配置文件示例：

```text
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
server {
    listen 80;
    server_name ci.eming.li;
    set $drone_port 3800;
    location / {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:$drone_port;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_buffering off;
        chunked_transfer_encoding off;
    }
    location ~* /ws {
        proxy_pass http://127.0.0.1:$drone_port;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
    }
}
```

修改 `server_name` 和 `$drone_port` 为正确的值即可，最后记得别忘了重启 Nginx 服务。这样我们就能直接使用 http://ci.eming.li 来访问 drone 了。

### 进程守护

刚才使用 `docker-compose up` 命令启动如果退出终端了之后服务就会停止，所以我们需要后台执行。我们可以直接使用 `nohup` 方法启动：

```text
$ nohup docker-compose -f docker-compose.yml up &
```

除了使用 `nohup` 之外，我们还可以使用 systemctl 来启动进程。我们创建一个 `drone.service` 服务文件：

```text
$ vim /etc/systemd/system/drone.service
```

复制以下内容：

```text
[Unit]Description=Drone serverAfter=docker.service nginx.service[Service]Restart=alwaysExecStart=/usr/local/bin/docker-compose -f /etc/drone/docker-compose.yml upExecStop=/usr/local/bin/docker-compose -f /etc/drone/docker-compose.yml stop[Install]WantedBy=multi-user.target
```

第一部分告诉 systemd 在 Docker 和 Nginx 可用之后启动此服务。 第二部分告诉 init 系统在发生故障时自动重新启动服务。 然后，它使用 Docker Compose 和我们之前创建的配置文件定义启动和停止 Drone 服务的命令。 最后，最后一节定义了如何使服务在启动时启动。完成后保存文件并使用如下命令启动服务：

```text
$ systemctl start drone
```

使用如下命令可以查看服务启动状态：

```text
$ systemctl status drone
```

### 问题

至此 drone 的安装就结束了。不过我在具体使用的过程中有两个属于安装的小问题，在此记录一下：

#### 容器无法访问网络

我先尝试了在本机 Mac 安装，一切正常。随后我在我的一台服务器上安装，访问后发现无法连接 Github 内部的所有网络服务都不正常。在论坛询问，作者回复告知可能是 docker nat 的问题。调试了很久也不行，最后重新开了一台镜像就好了。

#### 构建任务无故退出

实际使用过程中我发现我的构建任务总是莫名其妙就失败，提示我被 killed 掉，并显示 `exit code 137`。最开始我一直在查 137 退出码，发现就是 `kill -9` 的退出，不明所以。后来偶然尝试以 docker killed 为关键词搜索，才发现原来是内存爆了 docker 执行 OOM 把容器杀掉了。后来按照作者建议给机器加到 2G 的内存就没问题了。

![docker killed](https://p5.ssl.qhimg.com/t01ac191795724044e5.jpg)

### 总结

这样我们的安装过程就结束了，访问 drone，使用对应仓库的账户登录。如果是 Github 的话会使用 OAuth 连接到 Github 进行授权申请，如果是 Gogs 则需要输入 Gogs 的账号和密码。进去之后

**参考资料：**

* 《[Drone Installation Overview](http://docs.drone.io/installation/)》
* 《[如何在Ubuntu 16.04上安装和配置Drone](https://www.howtoing.com/how-to-install-and-configure-drone-on-ubuntu-16-04)》
* [Http request in drone are all connect timeout](https://discourse.drone.io/t/http-request-in-drone-are-all-connect-timeout/1083/2)
* [Drone killed build](https://github.com/drone/drone/issues/825)
* [Docker build fails with "pkg/template: signal: killed"](https://github.com/drone/drone/issues/390)

