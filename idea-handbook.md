# 个人设置

* Editor -> Inlay Hints -> Code vision -> Inheritors, Usages, Code author

# 插件篇

* CharAutoReplace: 自动将中文符号替换成英文

# 远程开发篇

![](https://image.cdn.yangbingdong.com/image/how-to-use-idea/9dac82d2775e449bcc3c2850a42ab5c4-4ae1d0.png)

在 IDEA 或者 JetBrains Gateway 中支持多种远程开发的方式, 这里介绍一下 SSH 以及 Dev Containers.

## SSH

首先得配置一个 SSH 连接的配置, 推荐使用公钥的方式.

1) 首先本地先生成密钥对(会有一些问答环节, 没特殊要求直接回车):

```
ssh-keygen -t rsa -b 4096 -C "your_email@domain.com"
```

之后会在  `$HOME/.ssh/` 目录下生成公钥 `id_rsa.pub` 以及私钥 `id_rsa`.

2) 将公钥追加到远程服务器上

```
cat .ssh/id_rsa.pub | ssh root@{host} 'cat >> ~/.ssh/authorized_keys'
# 或者
ssh-copy-id -i id_rsa.pub root@{host}
```

如果失败了注意是不是文件权限问题,  `.ssh` 目录需要 700, `authorized_keys` 是 600.

修改  `/etc/ssh/sshd_config`, 将  `PubkeyAuthentication` 改为 `yes`. 

重启 ssh:

```
systemctl restart sshd
```

然后可以配置 IDEA 中的 SSH 了:

![](https://image.cdn.yangbingdong.com/image/how-to-use-idea/44a45199b91746ee195abad92943b969-aafc74.png)

## Devcontainer

> ***[Development Containers](https://containers.dev/)***
>
> ***[IDEA Dev Container overview](https://www.jetbrains.com/help/idea/connect-to-devcontainer.html)***

Docker 也可以被当做一种远程系统, Devcontainer 就是利用了在 Docker, 通过声明镜像名, 必要的类库软件(比如 Git), 以及一些 Docker 声明周期的管理等配置到一个配置文件中(一般叫 `devcontainer.json`), 下面是一个 demo:

```
{
  "name": "minimal_ubuntu",
  "image": "ubuntu:latest",
  "onCreateCommand": {
    "update": "apt update && apt -y upgrade",
    "installDependencies": "apt install -y git"
  },
  "mounts": [{
    "source": "${localEnv:HOME}",
    "target": "/hostHome",
    "type": "bind"
  }, {
    "source": "mount",
    "target": "/volume/mount",
    "type": "volume"
  }],
  "customizations": {
    "jetbrains" : {"backend": "IntelliJ"},
  },
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  }
}
```

> 这里是一个 Jetbrains 官方的 devcontainer examples:  ***[devcontainers-examples](https://github.com/JetBrains/devcontainers-examples)*** 

通过 IDEA 打开:

![](https://image.cdn.yangbingdong.com/image/how-to-use-idea/eb85be0627d1262aed4393f91361966e-fd1a01.png)

之后就会根据 devcontainer.json 中的定义启动镜像, 准备环境变量, 执行定义的命令, 安装定义的 features, 下载 IDEA, 将代码 check in 等等, 完成之后就会打开一个 IDEA 窗口了.

![](https://image.cdn.yangbingdong.com/image/how-to-use-idea/a82ff1242095603adcb6fff923d4f42e-58a88d.png)



注意:** 如果是远程 Docker 的方式需要使用 SSH 连接方式(因为需要正确识别上下文):

![](https://image.cdn.yangbingdong.com/image/how-to-use-idea/a5cf419986fac82cdd3e1011d192da60-adc3d5.png)



目前就个人体验上来说不如 SSH 的方式, 虽然可通过 features 关键字添加一些开发工具, 但原理上是在容器启动之后再安装, 网络不好的情况下会导致启动时间很久, 所以建议预先将环境构建在 Dockerfile 中声明并实现构建好镜像. 而且Devcontainer 会下载一个新的 IDEA, 个性化配置或者插件需要重新弄一遍, 虽然有共享 volumn, 但这个被删了就需要重新弄了.