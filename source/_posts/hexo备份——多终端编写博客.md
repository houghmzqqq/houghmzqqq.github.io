---
title: hexo备份——多终端编写博客
date: 2018-10-25 14:39:41
tags:
	- 随笔
---

​        之前一直在个人笔记本上写博客，最近有需要在另一台终端上去写博客，所以需要将hexo上的资源文件备份到另一台电脑上。在这里记录一下操作过程，方便下一次需要备份博客时使用。

#### 1 github上备份

​        按照网上的教程，安装好node.js并搭建hexo环境后，使用```hexo d -g```命令后，hexo会生成一些静态文件并上传到github上。

![github静态文件](github上静态文件.png)

这些静态文件直接拉取下来是没用的，我们需要将资源文件也放到github上。



<!-- more -->



在github存放博客的仓库上，创建一个分支。对于我来说，就是在**houghmzqqq.github.io**仓库上创建**hexo**分支。

将你的hexo资源文件push到hexo分支上（别忘记先加个.gitignore文件）

![就是这些文件](hexo分支.png)





#### 2 终端拷贝

在你需要备份hexo的电脑上，新建一个文件夹，在该文件夹中进行如下操作：

1.```cnpm install hexo-cli -g``` 安装hexo（如果还没有安装hexo的话）

2.```cnpm install``` 安装其他依赖（如果没安装过的话）

3.```hexo init``` 初始化hexo，执行完后，会生成_config.yml、source等资源文件

4.```cnpm install --save hexo-deployer-git``` hexo生成静态文件并保存到github所需要的依赖

5.```npm i hexo-generator-json-content --save``` 标签支持依赖（如果你需要使用到标签的话）

6.然后克隆github博客仓库的hexo分支到该目录，然后就会将hexo初始化时生成的一些资源文件覆盖掉，就相当于将你的博客拷贝下来了，具体操作是：

> 1)、用tortoiseGit在hexo（你的博客文件夹）目录下创建版本库
>
> 2)、复制.gitignore文件，add文件，提交文件
>
> 3)、拉取文件，可以在拉取时配置远端信息，就是github仓库地址，对我来说是https://github.com/houghmzqqq/houghmzqqq.github.io.git
>
> 4)、合并文件，解决冲突（全部替换为远端文件就行）
>
> 5)、解决完冲突，再次提交并推送

然后就可以愉快的```hexo d -g``` 提交博客了

