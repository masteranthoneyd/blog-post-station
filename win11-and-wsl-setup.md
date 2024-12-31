# 其他优化

* 关闭还原点, 作用不大

# 开发篇

## WSL2 使用

![](https://image.cdn.yangbingdong.com/image/win11-and-wsl-setup/bac63390af8d5f042d85fc6a1bad45f7-5d4d9c.png)

安装: ***[如何使用 WSL 在 Windows 上安装 Linux](https://learn.microsoft.com/zh-cn/windows/wsl/install)***

> 在 Win11 上使用最新的 WSL2, 不需要在控制面板 - 程序 - 启用或关闭 Windows 功能中进行手动设置了, 安装子系统时会自动检测并开启必要的配置.

### 常用命令

* 进入子系统 `wsl ~`
  * 如果子系统处于 Stopped 状态, 则会启动子系统
  * 加 `--user` 或者 `-u` 可指定登录用户
  * 加 `~` 是为了登录进去后在用户目录下
* 查看可安装的子系统: `wsl -l -o`
* 安装指定版本的子系统: `wsl --install -d <Distribution Name  --web-download`
  * 加 `--web-download` 是为了从 Internet 而不是 Microsoft Store 下载分发版.
  * 在 2.4+ 版本后, 支持 `--name` 指定名字以及 `--location` 参数指定安装目录
* 查看已安装的子系统: `wsl -l -v`
* 停止运行某个子系统: `wsl --terminate <Distro>`
* 指定默认子系统: `wsl --set-default <Distro>`
* 卸载某个子系统: `wsl --unregister <Distro>`
* 将子系统移动到其他盘: `wsl --manage <Distro> --move <Location>`
  * 其他方法: ***[WSL修改默认安装目录到其他盘](https://www.cnblogs.com/tl542475736/p/14855863.html)***
* 更新 wsl: `wsl --update`
  * `--pre-release` 可更新到预览版

* 更多命令通过 `wsl --help` 查看

### 管理工具

GUI 管理工具: ***[wsl2-distro-manager](https://github.com/bostrot/wsl2-distro-manager/tree/main)***

* 通过该工具移动系统目录后, 默认的登录用户会变成 root, 如需更换, 执行 `ubuntu2404 config --default-user Username`
  * `ubuntu2404` 其实是对应了 `ubuntu2404.exe`, 在安装完子系统后会有对应的命令

### 其他设置与优化

**设置默认登陆用户**
`ubuntu2404 config --default-user Username`

> `ubuntu2404` 对应 `ubuntu2404.exe`, 在安装完子系统后会有对应的命令.



**WSL 高级配置**

可以通过 ***[`.wslconfig`](https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config#wslconfig)*** (放在用户根目录下)进行一些全局配置, 但是在开始菜单中搜索 WSL, 可以直接打开 WSL Setting 界面进行设置.

* `networkingMode` 可以设置为 `mirrored`, 启用镜像网络模式, 与主机共享 ip.
* `sparseVhd` 设为 `true` 让 wsl 空闲时候整理回收磁盘空间, 默认 vhdx是不会自动回收的.
* `autoMemoryReclaim` `gradual` 空闲时缓慢释放内存，机器物理内存不大的可以设置  `dropcache` 即可释放，你要是内存大可以不设置或者配合 memory 限制即可

这是我的 `.wslconfig` 配置:

```
[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
networkingMode=mirrored
[wsl2]
swap=0
```



## WSL Linux 开发环境配置

* 发行版我选择 Ubuntu 24.04.

* 另外我将这些安装的东西写成了自用的一件安装脚本: ***[https://github.com/masteranthoneyd/shell-hub/blob/master/ubuntu/ubuntu4dev.sh](https://github.com/masteranthoneyd/shell-hub/blob/master/ubuntu/ubuntu4dev.sh)***

### 换源

将 `/etc/apt/sources.list.d/ubuntu.sources` 中的内如替换为如下:

```
Types: deb
URIs: http://mirrors.aliyun.com/ubuntu/
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

### 通过 SDK Man 管理 JDK 以及其他工具

> 详细使用方法查看官网: ***[Usage](https://sdkman.io/usage)***

安装:

```
curl -s https://get.sdkman.io | bash
```

安装完后需要新开一个 Shell 才能使用, 如需在当前 Shell 使用, 执行:

```
/root/.sdkman/bin/sdkman-init.sh
```

安装 JDK 以及 Maven:

```
sdk install java ${JAVA_VERSION}
sdk install maven ${MAVEN_VERSION}
```

* 默认的安装目录是 `/root/.sdkman/candidates/<candidate>/<version>`
* 如果是第一个安装的 `candidate`, 会被设置为 default.
* 会自动设置 candidate home, 比如安装了 JDK, 会设置 `JAVA_HOME`(需要新起一个 Shell 生效)



常用命令:

```
# 查看可安装的 candidates 版本:
sdk list <candidates>

# 安装
sdk install <candidates> <version>

# 当前 shell 环境使用某个版本
sdk use <candidates> <version>

# 设置全局默认版本
sdk default <candidates> <version>

# 查看当前使用的版本
sdk current <candidates>

# 查看 candidates home 目录
sdk home <candidates> <version>
```

#### env 命令

另外 SDK Man 还提供了一个 env 命令, 用于识别项目根目录下的 `.sdkmanrc` 文件中定义的 candidates 版本从而切换到不同项目只需要执行 env 命令即可统一版本.

在项目根路径下执行 `sdk env init`, 生成 `.sdkmanrc` 文件, 内容大致如下:

```
# Enable auto-env through the sdkman_auto_env config
# Add key=value pairs of SDKs to use below
java=21.0.4-tem
```

使用:

```
# 使用 .sdkmanrc 定义的版本
sdk env

# 还原
sdk env clear

# 安装
sdk env install
```



## WSL2 + Jetbrains IDEA 开发

探索下来, 结合 WSL2 开发有四种方法:

* **[IDEA WSL 集成](https://www.jetbrains.com/help/idea/how-to-use-wsl-development-environment-in-product.html#create_project_for_wsl)**: IDEA 在打开或创建项目时能检测到 wsl, 会看到 `\\wsl$`  开头的路径.
  
  * 注意 JDK 需要选中 wsl 中安装的.
  
  * 运行时也是在 wsl 中运行.
  
  * ![](https://image.cdn.yangbingdong.com/image/win11-and-wsl-setup/cb4ad8bfddd637bd1f9c0b1a652da3cc-8cd062.png)
  
    
  
* **[基于 WSL 的远程开发](https://www.jetbrains.com/help/idea/remote-development-a.html#connect_to_wsl)**: 在远程开发中也可以看到与 wsl 的集成, 下载 IDEA 到 wsl 中再通过本地 Jetbrains Client 连接过去.

* **在 wsl 中安装 Jetbrains 全家桶(Toolbox)**: 需要开启 GUI(wslg) 的支持, win11 最新安装的 wsl2 默认开启了 GUI 的支持.
  
  * 如果打开 Toolbox 报了这个错: `dlopen(): error loading libfuse.so.2`, 执行一下: `sudo apt install libfuse2` 就可以了.
  * 用久了会卡
  
* **[在本地编辑代码, 但 run 在 wsl2 环境中](https://www.jetbrains.com/help/idea/how-to-use-wsl-development-environment-in-product.html#local_project)**: 原理是使用在 wsl 中挂载的本地目录.
  
  * 相对于上面三种方式, 这种方式的代码仓库是在宿主机上

## Docker

Docker 这里我选择使用 Docker Desktop 并启用 WSL 集成, 当然另外一个选择是直接在 WSL 子系统中安装 Docker Engine, 但考虑到我可能会频繁安装卸载 WSL(问就是喜欢折腾...), Docker Desktop 带来的好处就是它是一个单独的 WSL, 不会因为我卸载了开发用的 Ubuntu WSL 而废掉, 同时在 Windows 以及 WSL 里面都能用.

如果安装了多了 WSL 并且都需要 Docker, 可以在 Docker 设置中的 `Resources` -> `WSL integration` 中启用, 这样 Docker 会以挂载的形式集成到 WSL 中.

![](https://image.cdn.yangbingdong.com/image/win11-and-wsl-setup/78703435462c96e8bef5e90223a7c413-29c3e0.png)

## 终端美化

> zsh + oh-my-zsh + starship

![](https://image.cdn.yangbingdong.com/image/win11-and-wsl-setup/d2c00c5639d246e81c370ed41a9ce3f6-b6fe94.png)

### 安装 zsh 与 oh-my-zsh

>  zsh 是一个强大的 shell, 但配置较为繁琐, 而 oh-my-zsh 是一个已经调优好的 zsh 配置, 开箱即用.

```
apt install -y zsh
chsh -s /bin/zsh
```

> 需要起一个新的 shell 才能生效

**安装 *[oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)* **

一键安装(下载失败则需要代理😜):

```Shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

没有代理可以使用其他镜像安装, 比如***[清华大学镜像](https://mirrors.tuna.tsinghua.edu.cn/help/ohmyzsh.git/)***:

```
git clone https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
cd ohmyzsh/tools
REMOTE=https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git sh install.sh
```

### 安装 Starship

虽然 oh-my-zsh 预设了一些主题, 而 ***[Starship](https://github.com/starship/starship)*** 是另外一种主题方案,  提供了了轻量、迅速、客制化的高颜值终端.

前置要求:

- 安装并在**终端**启用 [***Nerd Font***](https://www.nerdfonts.com/) 字体(如 [***Fira Code Nerd Font***](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.3.0/FiraCode.zip) )

一键安装:

```
curl -sS https://starship.rs/install.sh | sh
```

在 `~/.zshrc` 的最后, 添加以下内容:

```
eval "$(starship init zsh)"
```

前往***[预设](https://starship.rs/presets/)***查看心仪的主题并启用, 比如:

```
starship preset gruvbox-rainbow -o ~/.config/starship.toml
```

### oh-my-zsh 插件

插件添加方式很简单, 将插件添加到 `.zshrc` 的 plugins 中, 例如下面是一些常规插件:

```
plugins=(
  git
  docker
  mvn
  ......
)
```



***[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)***: 根据命令历史记录提示命令, 按右方向键 `→` 补全.

![](https://image.cdn.yangbingdong.com/image/win11-and-wsl-setup/69f4f98c6b5fb9fbbd9b44da03b487c7-50b55f.png)

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```



***[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)***: 语法高亮

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```



### 终端字体颜色

颜色推荐: `#00FF00`

一般来说, 终端模拟器都会提供配色方案配置, 以 Windows Terminal, 将前景颜色改为 `#00FF00`:

![](https://image.cdn.yangbingdong.com/image/win11-and-wsl-setup/d9f8854a2bd3eb139a229b9f5d0967db-981228.png)

效果图:

![](https://image.cdn.yangbingdong.com/image/win11-and-wsl-setup/8113c44ab4e912b91d387df57a87d442-8aead8.png)

### 其他

#### 修改 ls 输出颜色

可以通过修改 LS_COLORS 变量: ***[changing the directory colour in a ls command](https://github.com/ohmyzsh/ohmyzsh/discussions/10493)***

可以通过替换 ls 命令, 比如使用 ***[lsd](https://github.com/lsd-rs/lsd)*** 或者***[exa](https://github.com/ogham/exa)***

![](https://image.cdn.yangbingdong.com/image/win11-and-wsl-setup/1b035fb473f0e70cbb13bbd4faa7818b-937d45.png)

