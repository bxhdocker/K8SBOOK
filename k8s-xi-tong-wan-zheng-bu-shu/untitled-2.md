# 镜像仓库-Harbor

## 一、简介

CentOS下安装Harbor镜像仓库

harbor的[官方安装指南](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md)介绍了harbor有三种安装方式，分别是在线安装、离线安装和OVA安装，由于网络环境受限，本文主要采用离线安装的方式。  
   官方文档上面说明需要依赖Python 2.7或以上版本，Docker引擎1.10以上，还有Docker Compose 1.6.0或以上版本。  
   CentOS 7.2自带Python 2.7.5

官方中文文档：[https://vmware.github.io/harbor/cn](https://vmware.github.io/harbor/cn)

安装配置指南：[链接](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)

用户使用指南：[链接](https://github.com/goharbor/harbor/blob/master/docs/user_guide.md)

## 二、架构分解

[![Docker&#xFF1A;&#x4F01;&#x4E1A;&#x7EA7;&#x79C1;&#x6709;&#x955C;&#x50CF;&#x4ED3;&#x5E93;Harbor&#x4F7F;&#x7528;](http://www.ywnds.com/wp-content/uploads/2017/01/2017011702141589.png)](http://www.ywnds.com/wp-content/uploads/2017/01/2017011702141589.png)

Harbor在架构上主要由五个组件构成：

**Proxy**

Harbor的registry, UI, token等服务，通过一个前置的反向代理统一接收浏览器、Docker客户端的请求，并将请求转发给后端不同的服务。

**Registry**

负责储存Docker镜像，并处理dockerpush/pull 命令。由于我们要对用户进行访问控制，即不同用户对Docker image有不同的读写权限，Registry会指向一个token服务，强制用户的每次docker pull/push请求都要携带一个合法的token,Registry会通过公钥对token 进行解密验证。

**Core services**

这是Harbor的核心功能，主要提供以下服务：

**UI（harbor-ui）**

提供图形化界面，帮助用户管理registry上的镜像（image）, 并对用户进行授权。

**webhook**

为了及时获取registry上image状态变化的情况， 在Registry上配置webhook，把状态变化传递给UI模块。

**token服务**

负责根据用户权限给每个docker push/pull命令签发token.Docker客户端向Regiøstry服务发起的请求,如果不包含token，会被重定向到这里，获得token后再重新向Registry进行请求。

**Database（harbor-db）**

为coreservices提供数据库服务，负责储存用户权限、审计日志、Docker image分组信息等数据。

**Log collector（harbor-log）**

为了帮助监控Harbor运行，负责收集其他组件的log，供日后进行分析。

Harbor的每个组件都是以Docker容器的形式构建的，因此很自然地，我们使用Docker Compose来对它进行部署。在源代码中\(https://github.com/vmware/harbor\), 用于部署Harbor的Docker Compose模板位于harbor/make/docker-compose.tpl。

Harbor安装步骤归结为以下几点：

1. 下载安装程序;
2. 配置**harbor.cfg文件**;
3. 运行**install.sh**来安装并启动Harbor;

## 三、安装

建议在centos7上安装，本文以centos7.4 64为例

### 1、安装docker

```text
yum install docker
systemctl start docker  启动服务
```

### 2、安装Docker Compose

下载1.21.2版本的Docker Compose，[参考文档](https://github.com/docker/compose/releases/)

```text
curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

修改docker-compose为可执行文件：

```text
cd /usr/local/bin/
chmod +x docker-compose
```

或者使用pip安装

```text
yum install python-pip -y
pip install docker-compose -y
```

测试是否安装成功

运行命令【docker-compose version】测试安装是否成功

### 3、安装Harbor

#### **3.1、下载离线安装包**

  在[GitHub](https://github.com/vmware/harbor/releases)上找到下载地址并下载，我这里下载的1.5.1版本，整个文件800多MB：

![](../.gitbook/assets/image%20%2874%29.png)

```text
下载以v1.6.2为例 
mkdir /home/harbor
cd /home/harbor
wget https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-offline-installer-v1.6.2.tgz
```

解压安装包

tar xvf harbor-offline-installer-v1.5.1.tgz

![](../.gitbook/assets/image%20%2843%29.png)

#### 3.2、修改配置文件【harbor.cfg】

```text
vim harbor.cfg
```

必需的参数有：

1. **hostname**：目标主机的主机名，用于访问UI和注册服务。不能使用localhost和127.0.0.1，因为harbor需要被外部客户端访问，我这里修改成了IP地址。

![](../.gitbook/assets/image%20%2837%29.png)

2. **ui\_url\_protocol**：用于访问UI和令牌/通知服务的协议，默认为http，如果在Nginx上启用了SSL认证可以设置成https，我这里用的默认的http。

![](../.gitbook/assets/image%20%2866%29.png)

3.电子邮件设置：Harbor需要这些参数才能向用户发送“密码重置”电子邮件，并且只有在需要该功能时才需要。还有，千万注意，在默认情况下SSL连接是没有启用-如果你的SMTP服务器需要SSL，但不支持STARTTLS，那么你应该通过设置启用SSL 

```text
mail_ssl = TRUE。
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin \<sample_admin@mydomain.com\>
email_ssl = false
```

4.harbour\_admin\_password：管理员的初始密码。此密码仅在港口首次发布时生效。之后，将忽略此设置，并且应在UI中设置管理员的密码。

```text
请注意，默认用户名/密码为admin / Harbor12345
```

auth\_mode：使用的认证类型。默认情况下，它是db\_auth，即凭据存储在数据库中。对于LDAP认证，请将其设置为ldap\_auth。

ldap\_url：LDAP端点URL（例如ldaps://ldap.mydomain.com）。 仅当auth\_mode设置为ldap\_auth时使用。

ldap\_searchdn：具有搜索LDAP / AD服务器（例如uid=admin,ou=people,dc=mydomain,dc=com）的权限的用户的DN 。

ldap\_search\_pwd：由指定的用户的密码ldap\_searchdn。

ldap\_basedn：查找用户的基本DN，例如ou=people,dc=mydomain,dc=com。 仅当auth\_mode设置为ldap\_auth时使用。

ldap\_filter：用于查找用户的[搜索过滤器](https://www.baidu.com/s?wd=%E6%90%9C%E7%B4%A2%E8%BF%87%E6%BB%A4%E5%99%A8&tn=24004469_oem_dg)，例如\(objectClass=person\)。

ldap\_uid：用于在LDAP搜索期间匹配用户的属性，可以是uid，cn，电子邮件或其他属性。

ldap\_scope：用于搜索用户的范围，1-LDAP\_SCOPE\_BASE，2-LDAP\_SCOPE\_ONELEVEL，3-LDAP\_SCOPE\_SUBTREE。默认值为3。

db\_password：用于db\_auth的MySQL数据库的根密码。更改此密码以用于任何生产使用！

self\_registration：（on或off。默认为on）启用/禁用用户注册自己的能力。禁用时，新用户只能由管理员用户创建，只有管理员用户才能在Harbor中创建新用户。 注意：当auth\_mode设置为ldap\_auth时，将始终禁用自注册功能，并且将忽略此标志

5. **max\_job\_workers**：作业服务中的最大复制worker数，这里默认写的50，考虑到我的服务器的性能，我这里修改成了5。  


![](../.gitbook/assets/image%20%2846%29.png)

6. **customize\_crt**：设置为on，prepare脚本创建用于生成/验证注册表令牌的私钥和根证书。如果设置成off，密钥和根证书将由外部源提供，我设置的是on。

![](../.gitbook/assets/image%20%2890%29.png)

7. **ssl\_cert**：SSL证书的位置，只有协议设置成https的时候，这个属性才会生效。

8. **ssl\_cert\_key**：SSL秘钥的位置，只有协议设置成https的时候，这个属性才会生效。

9. **secretkey\_path**：密码存放的路径，这里最好别修改，否则后面会报错，我修改成了【/data/admin/】。  


![](../.gitbook/assets/image%20%28114%29.png)

10. **log\_rotate\_count**：日志文件保留的数量，达到最大值后会循环删除之前的日志。

11. **log\_rotate\_size**：每个日志的大小，我为了节省空间设置日志最多保留5个，每个最大200MB。  


![](../.gitbook/assets/image%20%288%29.png)



#### **3.3、运行【prepare】**使用官方自带脚本更新参数

```text
./prepare
```

![](../.gitbook/assets/image%20%2850%29.png)

#### **3.4、开始安装**

执行【install.sh】自动进行安装

```text
[root@localhost harbor]# ./install.sh 
注意事项：
    A、这个脚本有点操蛋，1、ldap相关不能注释掉，不用也不能注释掉 2、hostname =reg.xx.com默认的不能有，注释掉也不行哦。
    B、在线安装需要去拉镜像 vmware/harbor-ui:0.5.0 一般会很慢，可以换成国内的来拉，我换成了daocloud，结果快很多
[Step 0]: checking installation environment ...

Note: docker version: 17.03.2

Note: docker-compose version: 1.21.2

[Step 1]: loading Harbor images ...
52ef9064d2e4: Loading layer [==================================================>] 135.9 MB/135.9 MB
4a6862dbadda: Loading layer [==================================================>] 23.25 MB/23.25 MB
58b7d0c522b2: Loading layer [==================================================>]  24.4 MB/24.4 MB
9cd4bb748634: Loading layer [==================================================>] 7.168 kB/7.168 kB
c81302a14908: Loading layer [==================================================>] 10.56 MB/10.56 MB
7848e9ba72a3: Loading layer [==================================================>] 24.39 MB/24.39 MB
Loaded image: vmware/harbor-ui:v1.5.1
f1691b5a5198: Loading layer [==================================================>] 73.15 MB/73.15 MB
a529013c99e4: Loading layer [==================================================>] 3.584 kB/3.584 kB
d9b4853cff8b: Loading layer [==================================================>] 3.072 kB/3.072 kB
3d305073979e: Loading layer [==================================================>] 4.096 kB/4.096 kB
c9e17074f54a: Loading layer [==================================================>] 3.584 kB/3.584 kB
956055840e30: Loading layer [==================================================>] 9.728 kB/9.728 kB
Loaded image: vmware/harbor-log:v1.5.1
185db06a02d0: Loading layer [==================================================>] 23.25 MB/23.25 MB
835213979c70: Loading layer [==================================================>]  20.9 MB/20.9 MB
f74eeb41c1c9: Loading layer [==================================================>]  20.9 MB/20.9 MB
Loaded image: vmware/harbor-jobservice:v1.5.1
9bd5c7468774: Loading layer [==================================================>] 23.25 MB/23.25 MB
5fa6889b9a6d: Loading layer [==================================================>]  2.56 kB/2.56 kB
bd3ac235b209: Loading layer [==================================================>]  2.56 kB/2.56 kB
cb5d493833cc: Loading layer [==================================================>] 2.048 kB/2.048 kB
557669a074de: Loading layer [==================================================>]  22.8 MB/22.8 MB
f02b4f30a9ac: Loading layer [==================================================>]  22.8 MB/22.8 MB
Loaded image: vmware/registry-photon:v2.6.2-v1.5.1
5d3b562db23e: Loading layer [==================================================>] 23.25 MB/23.25 MB
8edca1b0e3b0: Loading layer [==================================================>] 12.16 MB/12.16 MB
ce5f11ea46c0: Loading layer [==================================================>]  17.3 MB/17.3 MB
93750d7ec363: Loading layer [==================================================>] 15.87 kB/15.87 kB
36f81937e80d: Loading layer [==================================================>] 3.072 kB/3.072 kB
37e5df92b624: Loading layer [==================================================>] 29.46 MB/29.46 MB
Loaded image: vmware/notary-server-photon:v0.5.1-v1.5.1
0a2f8f90bd3a: Loading layer [==================================================>] 401.3 MB/401.3 MB
41fca4deb6bf: Loading layer [==================================================>] 9.216 kB/9.216 kB
f2e28262e760: Loading layer [==================================================>] 9.216 kB/9.216 kB
68677196e356: Loading layer [==================================================>]  7.68 kB/7.68 kB
2b006714574e: Loading layer [==================================================>] 1.536 kB/1.536 kB
Loaded image: vmware/mariadb-photon:v1.5.1
a8c4992c632e: Loading layer [==================================================>] 156.3 MB/156.3 MB
0f37bf842677: Loading layer [==================================================>] 10.75 MB/10.75 MB
9f34c0cd38bf: Loading layer [==================================================>] 2.048 kB/2.048 kB
91ca17ca7e16: Loading layer [==================================================>] 48.13 kB/48.13 kB
5a7e0da65127: Loading layer [==================================================>]  10.8 MB/10.8 MB
Loaded image: vmware/clair-photon:v2.0.1-v1.5.1
0e782fe069e7: Loading layer [==================================================>] 23.25 MB/23.25 MB
67fc1e2f7009: Loading layer [==================================================>] 15.36 MB/15.36 MB
8db2141aa82c: Loading layer [==================================================>] 15.36 MB/15.36 MB
Loaded image: vmware/harbor-adminserver:v1.5.1
3f87a34f553c: Loading layer [==================================================>] 4.772 MB/4.772 MB
Loaded image: vmware/nginx-photon:v1.5.1
Loaded image: vmware/photon:1.0
ad58f3ddcb1b: Loading layer [==================================================>] 10.95 MB/10.95 MB
9b50f12509bf: Loading layer [==================================================>]  17.3 MB/17.3 MB
2c21090fd212: Loading layer [==================================================>] 15.87 kB/15.87 kB
38bec864f23e: Loading layer [==================================================>] 3.072 kB/3.072 kB
6e81ea7b0fa6: Loading layer [==================================================>] 28.24 MB/28.24 MB
Loaded image: vmware/notary-signer-photon:v0.5.1-v1.5.1
897a26fa09cb: Loading layer [==================================================>] 95.02 MB/95.02 MB
16e3a10a21ba: Loading layer [==================================================>] 6.656 kB/6.656 kB
85ecac164331: Loading layer [==================================================>] 2.048 kB/2.048 kB
37a2fb188706: Loading layer [==================================================>]  7.68 kB/7.68 kB
Loaded image: vmware/postgresql-photon:v1.5.1
bed9f52be1d1: Loading layer [==================================================>] 11.78 kB/11.78 kB
d731f2986f6e: Loading layer [==================================================>]  2.56 kB/2.56 kB
c3fde9a69f96: Loading layer [==================================================>] 3.072 kB/3.072 kB
Loaded image: vmware/harbor-db:v1.5.1
7844feb13ef3: Loading layer [==================================================>] 78.68 MB/78.68 MB
de0fd8aae388: Loading layer [==================================================>] 3.072 kB/3.072 kB
3f79efb720fd: Loading layer [==================================================>]  59.9 kB/59.9 kB
1c02f801c2e8: Loading layer [==================================================>] 61.95 kB/61.95 kB
Loaded image: vmware/redis-photon:v1.5.1
454c81edbd3b: Loading layer [==================================================>] 135.2 MB/135.2 MB
e99db1275091: Loading layer [==================================================>] 395.4 MB/395.4 MB
051e4ee23882: Loading layer [==================================================>] 9.216 kB/9.216 kB
6cca4437b6f6: Loading layer [==================================================>] 9.216 kB/9.216 kB
1d48fc08c8bc: Loading layer [==================================================>]  7.68 kB/7.68 kB
0419724fd942: Loading layer [==================================================>] 1.536 kB/1.536 kB
543c0c1ee18d: Loading layer [==================================================>] 655.2 MB/655.2 MB
4190aa7e89b8: Loading layer [==================================================>] 103.9 kB/103.9 kB
Loaded image: vmware/harbor-migrator:v1.5.0


[Step 2]: preparing environment ...
Clearing the configuration file: ./common/config/adminserver/env
Clearing the configuration file: ./common/config/ui/env
Clearing the configuration file: ./common/config/ui/app.conf
Clearing the configuration file: ./common/config/ui/private_key.pem
Clearing the configuration file: ./common/config/db/env
Clearing the configuration file: ./common/config/jobservice/env
Clearing the configuration file: ./common/config/jobservice/config.yml
Clearing the configuration file: ./common/config/registry/config.yml
Clearing the configuration file: ./common/config/registry/root.crt
Clearing the configuration file: ./common/config/nginx/nginx.conf
Clearing the configuration file: ./common/config/log/logrotate.conf
loaded secret from file: /data/admin/secretkey
Generated configuration file: ./common/config/nginx/nginx.conf
Generated configuration file: ./common/config/adminserver/env
Generated configuration file: ./common/config/ui/env
Generated configuration file: ./common/config/registry/config.yml
Generated configuration file: ./common/config/db/env
Generated configuration file: ./common/config/jobservice/env
Generated configuration file: ./common/config/jobservice/config.yml
Generated configuration file: ./common/config/log/logrotate.conf
Generated configuration file: ./common/config/jobservice/config.yml
Generated configuration file: ./common/config/ui/app.conf
Generated certificate, key file: ./common/config/ui/private_key.pem, cert file: ./common/config/registry/root.crt
The configuration files are ready, please use docker-compose to start the service.


[Step 3]: checking existing instance of Harbor ...


[Step 4]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating harbor-db          ... done
Creating redis              ... done
Creating registry           ... done
Creating harbor-adminserver ... done
Creating harbor-ui          ... done
Creating harbor-jobservice  ... done
Creating nginx              ... done

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://192.168.242.132 . 
For more details, please visit https://github.com/vmware/harbor .
```

启动：

```text
docker-compose up –d
```

#### 3.5、打开页面访问：

![](../.gitbook/assets/image%20%28134%29.png)

命令行登陆Harbor

由于https原因登陆报错，修改docker配置文件

```text
vi /etc/sysconfig/docker
## 追加参数 --insecure-registry 172.16.22.76
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --insecure-registry 172.16.22.76
```



## 四、**升级Harbor和迁移数据**

1.      登到harbor所在的服务器上，如果harbor还在运行，就停止并删除对应的Harbor实例。

cd harbor

docker-compose down

2.     备份harbor当前的文件，确保在需要的时候可以回滚到当前的这个版本。

cd ..

mv harbor /my\_backup\_dir/harbor

3.      在github上获取最新的harbor发布版安装包，下载地址：[https://github.com/vmware/harbor/releases](https://github.com/vmware/harbor/releases)

4.      在更新harbor之前，先做数据库迁移操作。这个迁移工具以docker镜像的方式提供，所以你需要从docker hub上pull镜像。在下面的命令里，用harbor的发布版本号来替换\[tag\]：

docker pull vmware/harbor-db-migrator:\[tag\]

5.      备份数据库到一个目录，比如/path/to/backup。如果目录不存在的话，你需要自己创建，并且数据库的用户名和密码需要通过环境变量“DB\_USR”和“DB\_PWD”来提供。

docker run -ti --rm -e DB\_USR=root -e DB\_PWD=xxxx -v/data/database:/var/lib/mysql -v /path/to/backup:/harbor-migration/backupvmware/harbor-db-migrator:\[tag\] backup

6.     更新数据库模式并迁移数据：

docker run -ti --rm -e DB\_USR=root -e DB\_PWD=xxxx -v/data/database:/var/lib/mysql vmware/harbor-db-migrator:\[tag\] up head

7.     解压新的harbor安装包，并切换到工作目录./harbor中去。通过修改harbor.cfg来配置harbor。

* 通过修改harbor.cfg来配置harbor，你可能需要参考第二步操作时备份的配置文件。参考[安装和配置手册](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md)获取更多的信息。由于新版本的harbor.cfg配置文件的格式和内容可能会发生改变，所以**不能**直接从之前的版本来复制harbor.cfg配置文件。

**重要**：如果你更新harbor之前使用的认证方式为LDAP/AD，那边在你加载启动新版本的harbor之前，必须要确保harbor.cfg中的auth\_mode配置成**ldap\_auth**，否则，更新之后用户将无法登陆

**升级后回滚**

不管什么原因，如果你想回滚到之前的harbor版本，可以参考如下步骤：

1.     停harbor服务。

cd harbor

docker-compose down

2.     从备份文件/path/to/backup中恢复数据库。

docker run -ti --rm -e DB\_USR=root -e DB\_PWD=xxxx -v/data/database:/var/lib/mysql -v /path/to/backup:/harbor-migration/backupvmware/harbor-db-migrator:\[tag\] restore

3.     删除当前的harbor实例。

rm -rf harbor

4.     恢复老版本的harbor文件。

mv /my\_backup\_dir/harbor harbor

5.     使用之前的配置重启harbor服务。

如果之前版本是通过发布的二进制包安装的：

cd harbor

./install.sh

**注意**：如果你安装harbor选择其他组件，比如Notary或者Clair，可参考[安装和配置手册](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md)获取更新信息。

如果之前的harbor版本是通过源码安装的：

cd harbor

docker-compose up --build -d

**迁移工具参考**

* 使用help命令显示迁移工具帮助信息：

docker run --rm -e DB\_USR=root -e DB\_PWD=xxxxvmware/harbor-db-migrator:\[tag\] help

* 使用test命令测试mysql连接：

docker run --rm -e DB\_USR=root -e DB\_PWD=xxxx -v/data/database:/var/lib/mysql vmware/harbor-db-migrator:\[tag\] test

## 五、排障：

查看日志：例如查看adminserver的日志

```text
tail -n 50 /var/log/harbor/adminserver.log
```



文章参考：[https://www.jianshu.com/p/bca5251c9a08](https://www.jianshu.com/p/bca5251c9a08)

[http://blog.51cto.com/dangzhiqiang/1962874](http://blog.51cto.com/dangzhiqiang/1962874)



