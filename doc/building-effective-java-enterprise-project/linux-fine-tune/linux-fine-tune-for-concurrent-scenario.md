# Linux 高并发系统相关参数

## **文件描述符(句柄)**

**File Descriptor**(fd), 也就文件描述符, 又称**句柄**. Linux 中一切都是文件, 创建一个进程, 线程, TCP 连接等资源会返回一个 fd 用来表示. 如果 fd 设置过小, 就会遇到 **`Too many open files`** 的异常.

* `fs.file-max`: **系统**全局总限制

  * **修改**
    * **临时**生效: `sysctl -w fs.file-max=<新值>`
    * **永久**生效: 在 `/etc/sysctl.conf` 文件中添加或修改行 `fs.file-max = <新值>`, 然后执行 `sysctl -p` 生效.
  * **查看：**
    - `sysctl fs.file-max`
    - `cat /proc/sys/fs/file-max`
  * **注意**:  这是最底层的限制, 无论用户级和进程级限制设置得多高, 整个系统的打开文件描述符数量最终都不能超过 `fs.file-max`.

* `nofile`: **用户**级限制

  * **含义**: 针对**特定用户或用户组**设置的软限制 (`soft`) 和硬限制 (`hard`)

    * **软限制 (`soft nofile`)**: 用户进程实际能使用的文件描述符数量上限。进程可以临时超过软限制(如果内核配置允许),但通常会收到 `EMFILE` (Too many open files) 错误, 用户可以在硬限制范围内自行提高软限制(使用 `ulimit -n`)
    * **硬限制 (`hard nofile`)**: 用户进程所能拥有的文件描述符数量的**绝对上限**, 普通用户进程不能超过硬限制, 只有 root 用户可以修改硬限制。

  * **修改**

    * 配置文件 `/etc/security/limits.conf` 或 `/etc/security/limits.d/` 目录下的文件, 格式通常为: `<domain> <type> <item> <value>`:

      * ```
        * soft nofile 65535
        * hard nofile 65535
        ```

    * 如果是 systemd 系统, 还需修改 `/etc/systemd/*.conf`: `sed -i '/DefaultLimitNOFILE/c DefaultLimitNOFILE=65535:65535' /etc/systemd/*.conf`

    * 然后重启生效

      * 如果是 Windows **WSL**, 需要进入 Shell 后使用 `su root` 登录一次, 因为通过 wsl 命令进入的 shell 不被视为"登录会话"
        *  ***[ulimit -n command shows no change inspite of modifying the /etc/security/limits.conf file](https://askubuntu.com/questions/1326406/ulimit-n-command-shows-no-change-inspite-of-modifying-the-etc-security-limits)***
        * ***[Can not raise ulimit when running a command like ubuntu2004 run "ulimit -n"](https://github.com/microsoft/WSL/discussions/6226)***

    * **注意**: `nofile` 不能超过 `nr_open` 的值, `nr_open` 代表单个进程**可**分配的最大文件数, 通过 `/etc/sysctl.conf` 文件 `fs.nr_open` 配置, 默认值是 1048576.

* `ulimit`: **进程**级限制

  * **含义**:  这是针对**当前 shell 进程**及其启动的所有**子进程**的文件描述符数量的软限制, 默认情况下，`ulimit -n` 显示软限制.
  * 查看: 
    * 查看当前软限制: `ulimit -n`
    * 查看当前硬限制: `ulimit -Hn`
  * 修改: `ulimit -n <新的软限制值>`
    * 该设置只对当前 shell 生效

## 网络

### TCP 跟踪表 net.netfilter.nf_conntrack_*

`nf_conntrack` (在老版本的 Linux 内核中叫 `ip_conntrack` )是Linux 内核 **Netfilter 连接跟踪模块**，用于跟**踪一个网络连接的状态**, 这个模块是 `iptables`, `nftables` 等防火墙工具以及 NAT 功能的基础.

涉及 `nf_conntrack` 的一些参数: ***[nf_conntrack-sysctl.txt](https://www.kernel.org/doc/Documentation/networking/nf_conntrack-sysctl.txt)***

其中比较重要的参数:

* `net.netfilter.nf_conntrack_max`: 连接跟踪表的大小, 默认262144
  * 建议根据内存计算该值 `CONNTRACK_MAX = RAMSIZE (in bytes) / 16384 / (x / 32)`, `x` 为 64 或者 32, 并满足 `net.netfilter.nf_conntrack_max=4*nf_conntrack_buckets`
  * 当连接记录数量超过这个值, 通常会出现 **`table full, dropping packet`** 错误
* `net.netfilter.nf_conntrack_buckets`: 哈希表的大小, (`nf_conntrack_max/nf_conntrack_buckets`就是每条哈希记录链表的长度), 大于4G内存的系统默认为 65536
* `net.netfilter.nf_conntrack_tcp_timeout_established`: TCP **就绪**后会话的超时时间，默认是432000 (5天), 非常保守, 对于主要是短连接的服务器(如 Web 服务器)，这个值通常可以安全地减小(比如降到 1 小时 3600 或 1 天 86400), 以更快回收内存, 防止 `conntrack table full` 错误. 其他相关状态超时参数:
  * `net.netfilter.nf_conntrack_icmp_timeout`: ping 超时, 默认 30s
  * `net.netfilter.nf_conntrack_tcp_timeout_syn_sent`: 默认 120s
  * `net.netfilter.nf_conntrack_tcp_timeout_last_ack`: 默认 30s
  * `net.netfilter.nf_conntrack_tcp_timeout_time_wait`: 默认 120s

> 假设宿主机架构为 64 位且内存为 64GB，所以 `nf_conntrack_max` 估算: `64 * 1024 * 1024 * 1024 / 16384 / (64 / 32) = 2097152`, 又因为`nf_conntrack_max = nf_conntrack_buckets value * 4`, 所以 `nf_conntrack_buckets` 为 524288.

**查看**:

* `sysctl net.netfilter.nf_conntrack_max`
* `cat /proc/sys/net/netfilter/nf_conntrack_max`

修改:

* 临时修改 `sysctl -w  <key>=<value>`
* 永久生效: 修改 `/etc/sysctl.conf`



### TCP 队列

服务端与客户端在 TCP 三次握手过程中会涉及两个 TCP 连接队列, 一个是 `syns queue` 另外一个是 `accept queue`:

1. `client`发送`SYN`到`server`了将状态修改为`SYN_SEND`, 如果`server`收到请求, 则将状态修改为`SYN_RCVD`, 并把该请求放到`syns queue`队列中
2. `server` 回复 `SYN+ACK` 给 `client`, 如果`client`收到请求, 则将状态修改为 `ESTABLISHED`, 并发送 `ACK` 给 `server`
3. `server` 收到 `ACK`, 将状态修改为 `ESTABLISHED`, 并把该请求从 `syns queue` 中放到 `accept queue`

其中限制等待队列的内核参数:

* `net.core.somaxconn`: `accept queue` 队列最大值, 高并发场景可适当调大该值
* `net.ipv4.tcp_max_syn_backlog`: `syns queue` 队列最大值

其他相关参数:

* `net.ipv4.tcp_abort_on_overflow`: `accept queue` 已满时新的 TCP 连接过来时的行为, 0(默认值) 代表丢弃 ACK, 等 `client` 发送数据过来时返回 RST; 1 代表队列满了马上返回 RST.
* `net.ipv4.tcp_syncookies`: 当出现 `sysns queue` 等待队列溢出时，启用 cookies 来处理，可防范少量 SYN 攻击，默认为 1，会少量消耗 CPU

**修改**:

* 临时修改 `sysctl -w  <key>=<value>`
* 永久生效: 修改 `/etc/sysctl.conf`

### 端口

`net.ipv4.ip_local_port_range`: 发起一个 TCP 请求需要占用一个端口(Http2 可以端口复用), 在高并发发起请求场景下(常见于客户端), 可以适当修改端口范围:

* 临时生效: `sysctl -w net.ipv4.ip_local_port_range="1024 65535"`
* 永久生效: `/etc/sysctl.conf`

### TPC 连接优化

下面参数大多适用于高并发发起请求方的调优:

```
# 复用 TIME_WAIT 的 tcp 连接
net.ipv4.tcp_tw_reuse = 1  # 默认 0 
 
# 启用时间戳, tcp_tw_reuse 必须依赖 tcp_timestamps
net.ipv4.tcp_timestamps = 1 # 一般默认 1

# 空闲 10 分钟开始探测
net.ipv4.tcp_keepalive_time = 600 # 默认 7200 秒

# 每 30 秒重试一次
net.ipv4.tcp_keepalive_intvl = 30 # 默认 75 秒

# 尝试 3 次失败后断开
net.ipv4.tcp_keepalive_probes = 3 # 默认 9 次
 
# 主动关闭 TCP 连接会累积 TIME_WAIT 状态的连接
net.ipv4.tcp_max_tw_buckets = 262144

# 缩短 FIN_WAIT2 状态时间(被动关闭方)
net.ipv4.tcp_fin_timeout = 30 # 默认 60 秒
```

## 其他可选优化参数

```
# 该参数决定了，网络设备接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。
# 默认值：net.core.netdev_max_backlog = 1000
net.core.netdev_max_backlog = 8192

# 每个套接字允许的最大辅助缓冲区大小。辅助数据是带有附加数据的结构cmsghdr结构的序列。
# 默认值：net.core.optmem_max = 20480
net.core.optmem_max = 81920

# 指定了接收套接字缓冲区大小的最大值（以字节为单位）。
# 默认值：net.core.rmem_default = 212992
net.core.rmem_default = 262144

# TCP接收/发送缓存最小值，默认值，最大值
# 默认值：net.ipv4.tcp_rmem = 4096  131072  6291456
net.ipv4.tcp_rmem = 4096  32768  262142
# 默认值：net.ipv4.tcp_wmem = 4096  16384   4194304
net.ipv4.tcp_wmem = 4096  32768  262142

# Socket接收/发送缓存最大值
# 默认值均为：212992
net.core.rmem_max = 4194304
net.core.wmem_max = 419430

# 整个系统所有 TCP 连接可以使用的内存总量, 单位: 页, 一页 4 KB
net.ipv4.tcp_mem = 561588  748785  1123176
```

