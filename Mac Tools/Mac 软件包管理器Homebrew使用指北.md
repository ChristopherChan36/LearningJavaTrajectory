# Mac 软件包管理器Homebrew使用指北

## Homebrew

`Homebrew`由开发者 Max Howell 开发，并基于 BSD 开源，是一个非常方便的软件包包管理器工具。

[Homebrew 官网](https://brew.sh/index_zh-cn)

![](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/2020-03-08-%E6%88%AA%E5%B1%8F2020-03-0812.33.47.png)

## Homebrew 的几个核心概念

在正式介绍 Homebrew 的使用之前，我先为你介绍一下 Homebrew 中的一些核心的概念，了解这些概念，就可以帮助你更好的去使用 Homebrew。

| 词汇        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| formula (e) | 安装包的描述文件，formulae 为复数                            |
| cellar      | 安装好后所在的目录                                           |
| keg         | 具体某个包所在的目录，keg 是 cellar 的子目录                 |
| bottle      | 预先编译好的包，不需要现场下载编译源码，速度会快很多；官方库中的包大多都是通过 bottle 方式安装 |
| tap         | 下载源，可以类比于 Linux 下的包管理器 repository             |
| cask        | 安装 macOS native 应用的扩展，你也可以理解为有图形化界面的应用。 |
| bundle      | 描述 Homebrew 依赖的扩展                                     |

其中，最关键的是 tap 、cask，我们在后续会经常用到。

## Homebrew 常用操作

### 安装 Homebrew

#### 1. 自动安装（推荐）

在使用 Homebrew 之前，首先我们需要完成 Homebrew 的安装工作。Homebrew 的安装工作非常简单，只需要执行如下代码，就可以自动开始安装流程，后续根据提示操作即可。

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- 上边的命令执行两个命令，首先下载install文件，然后用系统的ruby工具安装
- 尽量在bash或者zsh下安装，fish下会提示不识别'$'
- 不需要使用超级权限（sudo），该文件会将HomeBrew安装至`usr/local`目录下。安装过程中会提示你执行哪些动作

后边还会有一些提示。继续的话会提示输入密码，等待安装完成。

安装完成后输入`brew -v` 即可显示是否安装成功：

![Homebrew version](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/2020-03-08-093112.png)

#### 2. 手动安装

执行如下命令：

```shell
mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
```

避免以下两点：

1. 目录内包含空格
2. 不要安装在`/sw`或者`/opt/local`目录下

当然也可以手动下载安装脚本，然后修改HOMEBREW_PREFIX变量的值，修改为自己的安装目录。

### Homebrew 常用命令总结

#### 1. 安装卸载软件

- `brew --version`或者`brew -v` 显示brew版本信息
- `brew install [软件名] ` 安装指定软件
- `brew uninstall [软件名]`  卸载指定软件
- `brew info [软件名]` 查看某个特定软件的信息

- `brew list`  显示所有的已安装的软件
- `brew search text` 搜索本地远程仓库的软件，已安装会显示绿色的勾
- `brew search /text/` 使用正则表达式搜软件

#### 2. 升级软件相关

- `brew update` 自动升级homebrew（从github下载最新版本）
- `brew outdated` 查看所有待更新版本的软件
- `brew upgrade`  升级所有已过时的软件，即列出的已过时软件
- `brew upgrade [软件名]`升级指定的软件
- `brew pin ` 禁止指定软件升级
- `brew unpin ` 解锁禁止升级
- `brew upgrade --all` 升级所有的软件包，包括未清理干净的旧版本的包

#### 3. 清理相关

Homebrew 用久了，会有一些历史版本的软件遗留在系统里，这个时候，你可以使用 `brew cleanup` 命令来清理系统中所有软件的历史版本，或者可以使用 `brew cleanup [软件名]`来清理特定软件的旧版。

- `brew cleanup -n` 列出需要清理的内容
- `brew cleanup [软件名]` 清理指定的软件过时包
- `brew cleanup` 清理所有的过时软件
- `brew unistall [软件名]` 卸载指定软件
- `brew unistall [软件名] --force` 彻底卸载指定软件，包括旧版本

通过brew安装的文件会自动设置环境变量，所以不用担心命令行不能启动的问题。 比如安装好了gradle，即可运行 `gradle -v`

### 搜索软件

很多时候，我们并不知道自己想要安装的软件是否有，又或者不知道软件的具体名字是什么，这个时候有两种方式来完成搜索

#### 1. 使用命令搜索

在命令行中，你可以直接使用 `brew search [关键词]` 来进行搜索

<img src="https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-03-08-102318.png" alt="命令行搜索软件" style="zoom:50%;" />

输入你想要的关键词，来搜索即可得到结果。

> 在搜索时应当遵循宁可少字，不能错字的原则来搜索。

####  2. 使用网页搜索

除了使用命令行搜索以外，你可以使用网页端的搜索工具来辅助你进行搜索。在 Homebrew 的官网，你可以找到 formulae 的链接，或者直接访问 https://formulae.brew.sh/ 来进行搜索。你只需要在界面的输入框中输入你要搜索的命令，然后就会出现对应的候选命令

<img src="https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-03-08-102557.jpg" alt="搜索软件" style="zoom:50%;" />



选择其中你要使用的那个，点击就会进入到软件的介绍页面

<img src="https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-03-08-102716.jpg" alt="查看软件介绍" style="zoom:50%;" />

你就可以看到 Homebrew 中的软件具体信息。

### 管理后台软件

诸如 Nginx、MySQL 等软件，都是有一些服务端软件在后台运行，如果你希望对这些软件进行管理，可以使用 `brew services` 命令来进行管理

- `brew services list`： 查看所有服务
- `brew services run [服务名]`: 单次运行某个服务
- `brew services start [服务名]`: 运行某个服务，并设置开机自动运行
- `brew services stop [服务名]`：停止某个服务
- `brew services restart`：重启某个服务

<img src="https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-03-08-104137.jpg" alt="管理后台软件" style="zoom:50%;" />

### 检查 Hombrew 环境

如果你的 Hombrew 没有办法正常的工作，你可以执行 `brew doctor` 来开启 Homebrew 自带的检查，从而确认有哪些问题，并进行修复。

<img src="https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-03-08-104211.jpg" alt="检查 Hombrew 环境" style="zoom:50%;" />

### 添加一个新的 tap

Homebrew 官方在安装的时候会有一些 tap 但是在使用时，依然会需要安装一些特殊的 tap ，这个时候，我们就要用到 tap 的命令来添加新的 tap.

在添加 tap 时，输入命令 `brew tap [user/repo]` ，就可以完成添加 tap 了

## Homebrew 常用 tap

在使用 Homebrew 时，我们一般会添加几个常用的 tap，来确保我们有足够的软件来安装。

### Caskroom

Caskroom 是 Homebrew 下一个非常出名的 tap ，有了 caskroom，我们就可以安装一些有图形化界面的软件了，比如 VSCode、Typora 等软件。

使用起来也非常简单，最新版 Homebrew 中，你可以直接使用 `brew cask install [软件名]` 来安装特定的软件，homebrew 会自动安装 Caskroom。

### homebrew-cask-fonts

程序员难免要安装一些代码字体，这样才能更好的写代码，Homebrew 也提供了方便我们安装字体的 tap。

在使用时，你需要先添加对应的 tap ，然后执行安装即可了，比如我们要安装 source code pro ，只需要执行如下命令。

```shell
brew tap homebrew/cask-fonts
brew cask install font-source-code-pro
```

## Homebrew 进阶技巧

### 切换国内的镜像源

Homebrew 默认使用的是国外的源，在下载时速度可能会比较慢。好在国内的清华大学和中科大提供了 Homebrew 的镜像源，我们可以很轻松的切换源，从而提升我们的下载速度。

#### 使用中科大的镜像

执行如下命令，即可切换为中科大的镜像

```shell
# 替换 Homebrew
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 替换 Homebrew Core
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

# 替换 Homebrew Cask
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# 替换 Homebrew-bottles
# 对于 bash 用户：
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
# 对于 zsh 用户：
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```

#### 使用清华大学的镜像

执行如下命令，即可切换为清华大学的镜像

```bash
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
```

### 使用 Brewfile 完成环境迁移

设备永久了，我们的电脑中会有大量的软件，如果你需要迁移环境，重新安装会是一个大麻烦，好在 Homebrew 本身为我们提供了一个非常好用的环境迁移的工具 —— Homebrew Bundle

你首先需要在之前的电脑中执行 `brew bundle dump` 来完成当前环境的导出,导出完成后，你会得到一个 *Brewfile*。

<img src="https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-03-08-111115.jpg" alt="环境迁移" style="zoom:50%;" />然后将 *Brewfile* 复制到新的电脑中，并执行 `brew bundle` 来开始安装的过程。

### 使用网页搜索 Caskroom 的软件

Brew Caskroom 并没有提供搜索的命令，不过我们可以借助一些网站来进行搜索，一个是 Homebrew 官方的 Caskrrom 页面：https://formulae.brew.sh/cask/

<img src="https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-03-08-111159.jpg" alt="Caskroom 网页搜索" style="zoom:50%;" />

在这个页面，你可以看到所有被收录的页面，在命令行中输入对应的软件就可以安装了。

你也可以访问 http://macappstore.org/，在网站中输入你要安装的软件，点击搜索，在弹出的页面中，查看安装指南即可。



<img src="https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/03/2020-03-08-111234.jpg" alt="img" style="zoom:50%;" />

## Homebrew 辅助软件

除了命令行，还有两款软件可以帮助我们更好的使用 Homebrew ，他们分别是 Cakebrew 和 launchrocket。

### Cakebrew

Cakebrew 是 Homebrew 的 GUI 管理器，在 Cakebrew 中，你可以看到当前所有已经安装的软件，并可以在 Caskbrew 中对其他软件执行升级等操作。

你只需要执行 `brew cask install cakebrew` 就可以完成 Cakebrew 的安装。

安装完成后，在 LaunchPad 中打开即可。

### launchrocket

launchrocket 可以用于管理 Homebrew 安装的服务，在使用时，你需要先添加对应的tap，然后再安装软件。

```text
brew tap jimbojsb/launchrocket
brew cask install launchrocket
```

安装完成后，在 LaunchPad 中打开即可。

## 查找homebrew的缓存路径

执行如下命令

```bash
brew --cache
```

一般情况下是如下路径`~/Library/Caches/Homebrew`

直接进入缓存路径

```bash
cd (brew --cache)
```

此时直接进入了缓存目录。

缓存目录可在安装时指定：（不建议） 首先下载安装脚本：

https://raw.githubusercontent.com/Homebrew/install/master/install 将这个脚本内容保存至本地随便命名install 下载好install脚本后，找到HOMEBREW_CACHE变量，修改为自己想要的文件夹，然后执行安装命令：

```bash
/usr/bin/ruby -e "$(cat install)"
```

即可安装homebrew。

## 卸载homebrew

执行如下命令

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```

## Reference

- Homebrew 官网：[https://brew.sh](https://brew.sh/)
- Homebrew Github：https://github.com/Homebrew/brew
- 程序员 Homebrew 使用指北: [https://sspai.com/post/56009#toc_0](https://sspai.com/post/56009#toc_0)
- HomeBrew常规使用教程：[https://juejin.im/post/5a559b9f6fb9a01cba42772f](https://juejin.im/post/5a559b9f6fb9a01cba42772f)