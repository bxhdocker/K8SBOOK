# Git 实战教程

#### Git是什么？

Git是目前世界上最先进的分布式版本控制系统。  


* Workspace：工作区
* Index / Stage：暂存区
* Repository：仓库区（或本地仓库）
* Remote：远程仓库

  
工作原理 / 流程：

![](../.gitbook/assets/image%20%2850%29.png)

#### SVN与Git的最主要的区别？

SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而干活的时候，用的都是自己的电脑，所以首先要从中央服务器那里得到最新的版本，然后干活，干完后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，如果在局域网还可以，带宽够大，速度够快，如果在互联网下，如果网速慢的话，就纳闷了。  
  
Git是分布式版本控制系统，那么它就没有中央服务器的，每个人的电脑就是一个完整的版本库，这样，工作的时候就不需要联网了，因为版本都是在自己的电脑上。既然每个人的电脑都有一个完整的版本库，那多个人如何协作呢？比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。  


#### 在windows上如何安装Git？

MSysGit是Windows版的Git，如下：  
[![1.jpg](http://dockone.io/uploads/article/20190119/6bf19d91eada9e6011b5671f2f7ee251.jpg)](http://dockone.io/uploads/article/20190119/6bf19d91eada9e6011b5671f2f7ee251.jpg)  
需要从网上下载一个，然后进行默认安装即可。安装完成后，在开始菜单里面找到 "Git --&gt; Git Bash"，如下：  
[![2.jpg](http://dockone.io/uploads/article/20190119/0623b1abb4e7ba77766d527a04fc3ef6.jpg)](http://dockone.io/uploads/article/20190119/0623b1abb4e7ba77766d527a04fc3ef6.jpg)  
会弹出一个类似的命令窗口的东西，就说明Git安装成功。如下：  
[![3.jpg](http://dockone.io/uploads/article/20190119/587e39fbbffb33639e2c8ee3cd14a6b0.jpg)](http://dockone.io/uploads/article/20190119/587e39fbbffb33639e2c8ee3cd14a6b0.jpg)  
安装完成后，还需要最后一步设置，在命令行输入如下：  
[![4.jpg](http://dockone.io/uploads/article/20190119/ce0d7280998f7e3be56aba4cfb1d188e.jpg)](http://dockone.io/uploads/article/20190119/ce0d7280998f7e3be56aba4cfb1d188e.jpg)  
因为Git是分布式版本控制系统，所以需要填写用户名和邮箱作为一个标识。  
  
注意：`git config --global`参数，有了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然你也可以对某个仓库指定的不同的用户名和邮箱。

**理解工作区与暂存区的区别？**

 我们前面说过使用Git提交文件到版本库有两步：  
  
第一步：是使用`git add`把文件添加进去，实际上就是把文件添加到暂存区。  
  
第二步：使用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支上





**原文链接**

\*\*\*\*[**http://dockone.io/article/8502**](http://dockone.io/article/8502)\*\*\*\*

