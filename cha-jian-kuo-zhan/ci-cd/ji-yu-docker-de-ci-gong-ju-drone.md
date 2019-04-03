# 基于DOCKER的CI工具—DRONE



![drone](https://p1.ssl.qhimg.com/t01eefa1424b3ec2668.jpg)

我在很多 Github 项目中都会使用 [travis-ci](http://travis-ci.org/) 来做自动化构建任务，其中的一项主要内容就是代码提交后自动执行单元测试计算测试覆盖率。但是 travis-ci 对于其它平台（例如 Gitlab）以及公司内网仓库来说都是不支持的，所以萌生了想要找另外一款 CI 工具代替 travis-ci 的想法。对比了几款 CI 产品之后，我最终选择了 [Drone](http://drone.io/)。它是一款使用 Go 开发的开源的 CI 自动构建平台，能够单独部署，支持常见的 Git 仓库，例如 Github, Gitlab, Bitbucket 以及 Gogs 等。[Drone](http://drone.io/) 主要有以下几个方面吸引我：

### 原生 Docker 支持

环境部署算是自动化构建比较复杂的一点了，目前最优解就是使用 docker 容器化技术打包我们的环境。原生 docker 的支持让我们不需要额外的在构建脚本中增加 docker 的命令，简单配置就可以为我们带来集成 docker 所引入的一系列的好处：环境隔离、标准化镜像。并且它的每一个插件都是一个镜像，让我们的自动构建环境模块化。

新版的 Drone 将其拆分成了 Server 端和 Agent 端，Server 用来处理任务分发，Agent 用来执行自动化构建任务。简单增加 Agent 容器实例，我们就可以非常方便的横向扩容 Drone。

### PipeLine AS Code

在项目根目录有一个 `.drone.yml` 的文件，这个是 Drone 构建脚本的配置文件，它随项目一块进行版本管理，开发者不需要额外再去维护一个配置脚本。其实现代 CI 程序都是这么做了，这个主要是相对于 Jekins 来说的。虽然 Jekins 也有插件支持，但毕竟还是需要配置。

另外就是基于 yaml 的配置文件非常简单（当然也要视需求而定），例如一个简单的构建脚本如下：

```text
pipeline:
  build:
    image: node:6.6.0
    commands:
      - npm install --silent
      - npm test
```

比起 Jekins 纷繁复杂的配置，Drone 几行就搞定了构建的需求。

### 丰富的插件支持

http://plugins.drone.io/ 这个是 Drone 的插件商店，包含了常见的一些需求插件，例如：

* 构建后发送消息：[slack](http://plugins.drone.io/drone-plugins/drone-slack/), [telegram](http://plugins.drone.io/appleboy/drone-telegram/), [line](http://plugins.drone.io/appleboy/drone-line/), [facebook](http://plugins.drone.io/appleboy/drone-facebook/), [discord](http://plugins.drone.io/appleboy/drone-discord/), [gitter](http://plugins.drone.io/drone-plugins/drone-gitter/), [email](http://plugins.drone.io/drillster/drone-email/)...
* 构建成功后发布：[npm](http://plugins.drone.io/drone-plugins/drone-npm/), [docker](http://plugins.drone.io/drone-plugins/drone-docker/), [github release](http://plugins.drone.io/drone-plugins/drone-github-release/), [google container](http://plugins.drone.io/drone-plugins/drone-gcr/)...
* 构建成功后部署：[AWS](http://plugins.drone.io/devops-israel/drone-ecs-deploy/), [Kubernetes](http://plugins.drone.io/mactynow/drone-kubernetes/), [rsync](http://plugins.drone.io/drillster/drone-rsync/), [scp](http://plugins.drone.io/appleboy/drone-scp/), [ftp](http://plugins.drone.io/christophschlosser/drone-ftps/)...

pipeline 中简单的增加插件镜像配置就可以支持插件，例如：

```text
pipeline:
pipeline:
  build:
    image: node:6.6.0
    commands:
      - npm install --silent
      - npm test
  
  publish:
    image: plugins/npm
    secret: [ npm_username, npm_pwd, npm_email ]
    username: ${NPM_USERNAME}
    password: ${NPM_PWD}
    email: ${NPM_EMAIL}
    when:
      event: [ tag ]
  notify:
    image: appleboy/drone-telegram
    secret: [ telegram_token, telegram_to ]
    token: ${TELEGRAM_TOKEN}
    to: ${TELEGRAM_TO}
```

由于 Drone 是基于 Docker 的，甚至是所有的插件也都是一个 docker 镜像。这代表着我们可以使用任何语言来开发 Drone 插件，最后只要打包成镜像就好了。

### 其它 CI 工具

Drone 相对于 Jekins 来说优势比较明显，复杂的配置以及需要插件支持的构建脚本 pipeline 化都是我没考虑它的原因。不过 Jekins 其丰富的插件扩展，针对于无项目的自动化构建来说还是非常不错的。Gitlab-CI 也是一款不错的工具，不过从名字上也能看出它的缺点，就是只支持 Gitlab，而且无法扩展配置文件，对于正好在使用 Gitlab 且需求不高的同学可以试试。

### 总结

基本上 Drone 的流程是我想要的 CI 模式了。不过人无完人，drone 也有一些不足，主要是：

* 不同版本的文档太多，老文档的一些功能都不支持了。
* 社区还不够发展，找个问题比较困难。不过好在有个官方论坛，作者回复倒是挺勤快的。
* 前端界面比较简单，很多功能还是得依靠命令行。

不过这些问题都不影响我对它的看法，相信它会越来越好。

**相关资料：**

**原文链接：**[**https://imnerd.org/drone.html**](https://imnerd.org/drone.html)\*\*\*\*

* [《Drone 一个原生支持 Docker 的 CI》](https://aisensiy.github.io/2017/08/04/drone-best-ci/)
* [《為什麼我用 Drone 取代 Jenkins 及 GitLab CI》](https://blog.wu-boy.com/2017/09/why-i-choose-drone-as-ci-cd-tool/)

