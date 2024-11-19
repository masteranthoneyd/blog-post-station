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

Devcontainer 原理上就是在 docker 容器里面构建一个开发环境, 然后通过 Gateway IDEA 客户端连接上去.

个人觉得 Devcontainer 有一些不足:

* 虽然可通过 features 关键字添加一些开发工具, 但原理上是在容器启动之后再安装, 网络不好的情况下会导致启动时间很久, 所以建议预先将环境构建在 Dockerfile 中声明并实现构建好镜像.
* 每次新建一个 Devcontainer 会下载一个新的 IDEA, 有些个性化配置或者插件需要重新弄一遍, 但这个问题应该可以通过共享 cache 目录来解决.