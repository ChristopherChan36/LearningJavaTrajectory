#Hexo + Github 博客搭建详细教程

## 前置环境

### 安装 Node.js

首先下载 [Node.js](https://nodejs.org/zh-cn/),直接在官网下载安装包，有稳定版和最新版两个版本可供下载，安装选项全部默认，一路 Next 就可以了。

最后，安装完毕后，通过 `node -v` 和 `npm -v` 检查下版本号。

当然，如果你跟我一样比较懒，可以使用包管理器进行安装，参考 https://nodejs.org/zh-cn/download/package-manager/，因为我是 macOS 系统，使用 Homebrew 进行安装 `brew install node`。如果对 Homebrew 不太了解的，可以看下 [Mac 软件包管理器Homebrew使用指北](https://www.cnblogs.com/christopherchan/p/12444435.html)。

```shell
brew install node
node -v
npm -v
```

### 添加国内镜像源

为了提高 npm 运行速度需要添加淘宝源：

```shell
npm config set registry https://registry.npm.taobao.org
```

### 安装 Git

为了把本地的网页文件上传到github上面去，我们需要用到分布式版本控制工具————Git [下载地址](https://git-scm.com/download/)

Git 对于咱们开发人员是很常用的工具，这里就不过多阐述了。

### GitHub 仓库

接下来就去注册一个github账号，用来存放我们的网站。大多数小伙伴应该都有了吧，作为一个合格的程序猿（媛）还是要有一个的。

打开https://github.com/，新建一个项目，如下所示：

![New repository](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-18-150131.png)

然后如下图所示，输入自己的项目名字，后面一定要加`.github.io`后缀，README初始化也要勾上。**名称一定要和你的github名字完全一样，比如你github名字叫`abc`，那么仓库名字一定要是`abc.github.io`**。

![yourname repo](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-18-150636.png)

建议还是选默认的 Public，我试了 Private，GitHub Pages 服务竟然是要收费。项目建成后，点击`Settings`，向下拉到最后有个`GitHub Pages`，点击`Choose a theme`选择一个主题。然后等一会儿，再回到`GitHub Pages`，会变成下面这样：

![GitHub Pages](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-18-155911.png)

点击那个链接，就会出现自己的网页啦，效果如下：

![pages preview](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-18-160005.png)

## Hexo 安装

Hexo 安装参考官方文档即可：[Hexo 官方文档](https://hexo.io/zh-cn/docs/)

安装完成后，输入 `hexo -v` 验证是否安装成功。

然后就可以初始化我们的博客，输入`hexo init`初始化文件夹，接着输入 `npm install` 安装必备的组件。

这样本地的博客配置就可以了，输入`hexo g`生成静态网页，然后输入`hexo s`打开本地服务器，然后浏览器打开 http://localhost:4000/，就可以看到生成的博客啦！

因为我打算使用 [Volantis](https://github.com/volantis-x/hexo-theme-volantis) 主题，官方为 Mac 用户贴心准备了脚本完成全部流程（博客初始化+主题安装），所以这里我就偷个懒，直接采用脚本完成下载安装，如果你也是 Mac 用户，直接在要存放博客的文件地址下输入 `curl -s https://volantis.js.org/start | bash`即可安装完成。详情参考 [Volantis 官网文档](https://volantis.js.org/v3/getting-started/)。

![volantis 安装脚本](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/08/2020-08-15-032406.png)

本地博客效果图如下：

![volantis 本地效果图](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/08/2020-08-15-032814.png)

### 将 hexo 与 github 关联起来

首先打开终端，输入如下命令在 git 上注册你的 github 账户名和邮箱

```bash
# Your github name and email
git config --global user.name "xxxx"
git config --global user.email "xxxx@163.com"
```

Github 的用户名和邮箱自填。

然后生成秘钥 SSH Key：

```bash
# Your github email
ssh-keygen -t rsa -C "xxxx@163.com"
```

打开 GitHub，进入 `settings`页面，再点击 `SSH and GPG keys`，新建一个 SSH，名字随意，终端输入

`cat ~/.ssh/id_rsa.put`

将输入的内容复制到文本框，然后确定保存。

输入 `ssh -T git@github.com`，如下图所示，即配置成功。

![ssh 配置](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/08/2020-08-15-035131.png)

打开博客根目录下的`_config.yml`文件，这是博客的配置文件，在这里你可以修改与博客相关的各种信息。

修改最后一行的配置：

```bash
deploy:
  type: git
  repository: https://github.com/godweiyang/godweiyang.github.io
  branch: master
```

`repository` 修改为你自己的 github 项目地址。接着需要先安装 `deploy-git`，也就是部署的命令，这样你才能用命令部署到 GitHub。接着 `hexo clean` 清除了你之前生成的东西，也可以不加。 `hexo g` 顾名思义，生成静态文章是 `hexo generate` 的缩写 `hexo deploy` 部署文章，可以用 `hexo d` 缩写：

```bash
$ npm install hexo-deployer-git --save
$ hexo clean | hexo g | hexo d
```

### 备份博客源文件

将本地的 `hexo` 分支推送到远程：

































