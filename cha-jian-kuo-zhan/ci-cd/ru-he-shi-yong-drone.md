# 如何使用DRONE



在上文《[DRONE安装指南](https://imnerd.org/drone-installation.html)》中介绍了如何安装 Drone 服务，下面我将来介绍重头戏——如何在项目中使用 Drone 完成自动化构建任务。首先我们要在 drone 中开启自动构建，实际上它会在对象的 Git 仓库中添加 webhooks 方便当开发者执行对应 git 操作后触发自动构建任务。

![drone activation](https://p5.ssl.qhimg.com/t01e1f991a862a1b7b4.jpg)

### .drone.yml

在《[基于DOCKER的CI工具—DRONE](https://imnerd.org/drone.html)》一文中我们有说过它的每一个构建插件都是镜像，能够模块化的配置我们的构建任务。将构建任务理解成是一条管道，项目流经管道做各种具体的任务。以下是一个简单的管道流程，代码克隆下来后执行安装并执行测试，完成后使用使用 Telegram 通知用户。

```text
+-------+     +-------+     +------+     +----------+| Clone | ==> | Build | ==> | Test | ==> | Telegram |+-------+     +-------+     +------+     +----------+
```

通过编辑放置在项目的根目录下的 `.drone.yml` 文件，我们可以对构建流进行配置。下面就是上图构建流程的配置：

```text
pipeline:  build:    image: node:6.6.0    commands:      - npm install --silent      - npm test        telegram:    image: appleboy/drone-telegram    token: 483467761:AAGVOmbOitx0V_1Vx4vlUwy-eoMBVYnSqP0    to: 337635385    message: >      {{#success build.status}}      主人，{{repo.owner}}/{{repo.name}}第{{build.number}}次构建成功！      {{else}}      主人，{{repo.owner}}/{{repo.name}}第{{build.number}}次构建失败了，快来修理下吧。      {{/success}}
```

在配置文件中我们总共定义了两个管道任务：

* **build**: 我们为 build 任务指定了 `node:6.6.0` 镜像，并在镜像中执行 `npm install --silent` 命令安装依赖。最后执行 `npm test` 来运行单元测试脚本。
* **telegram**: 我们为 telegram 任务指定了 `appleboy/brone-telegram` 镜像，并定义了一些镜像需要的环境参数。包括 telegram 的机器人 token，指定了消息的发送对象 to 以及消息的模板。

将其保存提交后就能正常触发任务并使用 telegram 接收消息了。

![build result](https://p2.ssl.qhimg.com/t017f906baba6e05df9.jpg)

### telegram

telegram 插件需要设置 bot token，没有的需要先关注 @BotFather 并输入 `/netbot` 命令创建一个 bot，这样你就能获取到 token 了。

而 `to` 参数需要配置的是发送用户的 ID，这个比较坑爹了。最开始以为是 username，输入之后一直没有反应，后来才反应过来可能是唯一 ID。不过 telegram ID 的获取比较困难，有人为此特地做了一个机器人。关注 @userinfobot 后和它发个消息它就会告诉你你自己的 ID 啦。这里有个小技巧是把别人的消息转发给它，它会给出对象的 ID。

关于 telegram 插件更多的配置可以参考：http://plugins.drone.io/appleboy/drone-telegram/

### Secrets

由于我们将 token 直接写在了 `.drone.yml` 的配置文件中，项目其它人也都可见，这会带来一些潜在的安全问题。为此我们可以使用 drone 的 secrets 功能。我们小改一下配置文件：

```text
telegram:    image: appleboy/drone-telegram    secrets: [ telegram_token, telegram_to ]    token: ${TELEGRAM_TOKEN}    to: ${TELEGRAM_TO}
```

同时我们在网站后台添加两个名为 `telegram_token` 和 `telegram_to` 的密钥：

![drone secret](https://p3.ssl.qhimg.com/t010daa0af390eff524.jpg)

保存后重新执行构建任务即可，这样我们就做到了隐藏敏感信息的目的。当然其实这么做也有缺陷，因为在网站后台添加的密钥是对管道中所有的构建插件全局共享的，如果开发者在管道中增加其它插件并 echo 或者其它操作，一样也是能获取到密钥的。为此我们就需要使用 drone-cli 命令来添加密钥，它支持指定密钥的作用域镜像。

```text
$ drone secrets add \    --repository=lizheming/qalarm \    --name=telegrame_to \    --value=337635385 \    --image=appleboy/telegram
```

**参考资料：**

* 《[Getting Started](http://docs.drone.io/getting-started/)》
* 《[Telegram Bot API](https://core.telegram.org/bots/api)》
* 《[Drone Secret 安全性管理](https://blog.wu-boy.com/2017/11/drone-secret-security/)》

