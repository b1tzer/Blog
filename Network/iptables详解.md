# iptables 详解

## 1. 基本概念

### 1.1 什么是防火墙
在计算中，防火墙是基于预定安全规则来监视和控制传入和传出网络流量的网络安全系统。该计算机流入流出的所有网络通信均要经过此防火墙。防火墙对流经它的网络通信进行扫描，这样能够过滤掉一些攻击，以免其在目标计算机上被执行。防火墙还可以关闭不使用的端口。而且它还能禁止特定端口的流出通信，封锁特洛伊木马。最后，它可以禁止来自特殊站点的访问，从而防止来自不明入侵者的所有通信。

防火墙分为软件防火墙和硬件防火墙，他们的优缺点
:
- 硬件防火墙：拥有经过特别设计的硬件及芯片，性能高、成本高 (当然硬件防火墙也是有软件的，只不过有部分功能由硬件实现，所以硬件防火墙其实是硬件 + 软件的方式)；

- 软件防火墙: 应用软件处理逻辑运行于通用硬件平台之上的防火墙，性能比硬件防火墙低、成本低。

### 1.2 iptables 是什么？Netfilter 与 iptables 的关系

`iptables` 一般指**配置 Linux 内核防火墙的命令行工具**，是 Netfilter 项目中的一部分，是内核防火墙与维护人员的**窗口**。术语 [*iptables*](https://wiki.archlinux.org/title/iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 也经常用于指代内核集防火墙。

Netfilter 是由 Rusty Russell 提出的 Linux 2.4 内核防火墙框架，该框架既简洁又灵活，可实现安全策略应用中的许多功能，如数据包过滤、数据包处理、地址伪装、透明代理、动态网络地址转换 (Network Address Translation，NAT)，以及基于用户及媒体访问控制 (Media Access Control，MAC) 地址的过滤和基于状态的过滤、包速率限制等。iptables/Netfilter 的这些规则可以通过灵活组合，形成非常多的功能、涵盖各个方面，这一切都得益于它的优秀设计思想。

Netfilter 是 Linux 操作系统核心层内部的一个数据包处理模块，它具有如下功能: 

- 网络地址转换 (Network Address Translate)
- 数据包内容修改
- 以及数据包过滤的防火墙功能

Netfilter 平台中制定了数据包的五个挂载点 (Hook Point，我们可以理解为回调函数点，数据包到达这些位置的时候会主动调用我们的函数，使我们有机会能在数据包路由的时候改变它们的方向、内容)，这 5 个挂载点分别是 `PRE_ROUTING`、`INPUT`、`OUTPUT`、`FORWARD`、`POST_ROUTING`。

Netfilter 所设置的规则是存放在内核空间中的，而 iptables 是一个应用层的应用程序，它通过 Netfilter 放出的接口来对存放在内核空间中的 XXtables(Netfilter 的配置表) 进行修改。这个 XXtables 由表 tables、链 chains、规则 rules 组成，iptables 在应用层负责修改这个规则文件，类似的应用程序还有 firewalld (CentOS7 默认防火墙)。

所以 Linux 中真正的防火墙是 Netfilter，但由于都是通过应用层程序如 iptables 或 firewalld 进行操作，所以我们一般把 iptables 或 firewalld 叫做 Linux 的防火墙。

注意
: 以上说的 `iptables` 都是针对 `IPv4` 的，如果 `IPv6`，则要用 `ip6tables`，至于用法应该是跟 `iptables` 是一样的。

注
: Linux 系统运行时，内存分内核空间和用户空间，内核空间是 Linux 内核代码运行的空间，它能直接调用系统资源，用户空间是指运行用户程序的空间，用户空间的程序不能直接调用系统资源，必须使用内核提供的接口 `system call`。

### 1.3 链

`iptables` 开启后，数据报文从进入服务器到出来会经过 5 道关卡，分别为 `Prerouting` (路由前)、`Input` (输入)、`Output` (输出)、`Forward` (转发)、`Postrouting` (路由后): 

![linux_iptables](https://i.loli.net/2021/09/28/a79FYdQBNyz15Tb.png)

每一道关卡中有多个规则，数据报文必须按顺序一个一个匹配这些规则，这些规则串起来就像一条链，所以我们把这些关卡都叫 “链”: 

![linux_chains](https://i.loli.net/2021/09/28/6NS32o9rWlUcYd5.png)

- `INPUT`
: 当接收到防火墙本机地址的数据包 (入站) 时，应用此链中的规则；

- `OUTPUT`
: 当防火墙本机向外发送数据包 (出站) 时，应用此链中的规则；

- `FORWARD`
: 当接收到需要通过防火墙发送给其他地址的数据包 (转发) 时，应用此链中的规则；

- `PREROUTING`
: 在对数据包作路由选择之前，应用此链中的规则，如 DNAT；

- `POSTROUTING`
: 在对数据包作路由选择之后，应用此链中的规则，如 SNAT。

其中 `INPUT`、`OUTPUT` 更多的应用在 “主机防火墙” 中，即主要针对服务器本机进出数据的安全控制；而 `FORWARD`、`PREROUTING`、`POSTROUTING` 更多的应用在 “网络防火墙” 中，特别是防火墙服务器作为网关使用时的情况。

### 1.4 表
虽然每一条链上有多条规则，但有些规则的作用 (功能) 很相似，多条具有相同功能的规则合在一起就组成了一个“表”，`iptables` 提供了四种“表”: 
- Filter 表
: 主要用于对数据包进行过滤，根据具体的规则决定是否放行该数据包 (如 DROP、ACCEPT、REJECT、LOG)，所谓的防火墙其实基本上是指这张表上的过滤规则，对应内核模块 iptables_filter；

- NAT 表
: Network Address Translation，网络地址转换功能，主要用于修改数据包的 IP 地址、端口号等信息 (网络地址转换，如 SNAT、DNAT、MASQUERADE、REDIRECT)。属于一个流的包(因为包的大小限制导致数据可能会被分成多个数据包) 只会经过这个表一次，如果第一个包被允许做 NAT 或 Masqueraded，那么余下的包都会自动地被做相同的操作，也就是说，余下的包不会再通过这个表。对应内核模块 iptables_nat；

- Mangle 表
: 拆解报文，做出修改，并重新封装，主要用于修改数据包的 ToS(Type Of Service，服务类型)、TTL(Time To Live，生存周期) 指以及为数据包设置 Mark 标记，以实现 QoS(Quality Of Service，服务质量) 调整以及策略路由等应用，由于需要相应的路由设备支持，因此应用并不广泛。对应内核模块 iptables_mangle；

- Raw 表
: 是自 1.2.9 以后版本的 iptables 新增的表，主要用于决定数据包是否被状态跟踪机制处理，在匹配数据包时，raw 表的规则要优先于其他表，对应内核模块 iptables_raw。
我们最终定义的防火墙规则，都会添加到这四张表中的其中一张表中。

### 1.5 表链关系
这5 条链 (即 5 个关卡) 中，并不是每条链都能应用所有类型的表，事实上除了 `Ouptput` 链能同时有四种表，其他链都只有两种或三种表: 

![linux_iptables_table_chain_relation](https://i.loli.net/2021/09/28/X5ntkg82TbxE1vo.png)




![linux_iptables_flow](https://i.loli.net/2021/09/30/6lHgTKDpMjzshb8.png)



从图中可以看出，所有链路遵循一个规则
: 匹配的顺序 **raw→mangle→nat→filter**

| Table  | PREROUTING | INPUT | FORWORD | OUTPUT | POSTROUTING |
| ------ | :--------: | :---: | :-----: | :----: | :---------: |
| Raw    |     ✅      |   ❌   |    ❌    |   ✅    |      ❌      |
| Mangle |     ✅      |   ✅   |    ✅    |   ✅    |      ✅      |
| NAT    |     ✅      |   ✅   |    ❌    |   ✅    |      ✅      |
| Filter |     ❌      |   ✅   |    ✅    |   ✅    |      ❌      |

**数据包流向总结如下图:**

![linux_firmwall_flow](https://i.loli.net/2021/09/28/bMDnz2ApNw31SFl.png)

## 2. iptables 的规则与操作

iptables 规则主要包含 “条件 & 动作”，即匹配出符合什么条件(规则) 后，对它采取怎样的动作。

### 2.1 条件

- `S_IP`: source ip
  
- `S_PORT`: source port
  
- `D_IP`: destination ip
  
- `D_PORT`: destination port
  
- `TCP/UDP`: 第四层 (传输层) 协议

### 2.2 动作

- `ACCEPT`: 允许数据包通过；
  
- `DROP`: 直接丢弃数据包，不回应任何信息，客户端只有当该链接超时后才会有反应；
  
- `REJECT`: 拒绝数据包，会给客户端发送一个数据包被丢弃的响应的信息；
  
- `SNAT`: S 指 Source，源 `NAT`(源地址转换)。在进入路由层面的 route 之后，出本地的网络栈之前，改写源地址，目标地址不变，并在本机建立 `NAT` 表项，当数据返回时，根据 `NAT` 表将目的地址数据改写为数据发送出去时候的源地址，并发送给主机。解决私网用户用同一个公网 IP 上网的问题；
  
- `MASQUERADE`: 是 `SNAT` 的一种特殊形式，适用于动态的、临时会变的 IP 上；
  
-  `DNAT` 指 `Destination NAT`，解决私网服务端，接收公网请求的问题。和 `SNAT` 相反，IP 包经过 route 之前，重新修改目标地址，源地址不变，在本机建立 `NAT` 表项，当数据返回时，根据 `NAT` 表将源地址修改为数据发送过来时的目标地址，并发给远程主机。可以隐藏后端服务器的真实地址；

- `REDIRECT`: 在本机做端口映射；

- `LOG`: 在 `/var/log/messages` 文件中记录日志信息，然后将数据包传递给下一条规则。
  
除去最后一个 LOG，前 3 条规则匹配数据包后，该数据包不会再往下继续匹配了，**所以编写的规则顺序极其关键。**
其中 `REJECT` 和 `DROP` 有点类似，以下是服务器设置 `REJECT` 和 `DROP` 后，客户端 ping 服务器的响应的区别: 

* `REJECT` 动作: 

    ```ini
    PING 10.37.129.9 (10.37.129.9): 56 data bytes
    92 bytes from centos-linux-6.5.host-only (10.37.129.9): Destination Port Unreachable
    Vr HL TOS  Len   ID Flg  off TTL Pro  cks      Src      Dst
    4  5  00 5400 29a3   0 0000  40  01 3ab1 10.37.129.2  10.37.129.9

    Request timeout for icmp_seq 0
    ```

* `DROP` 动作: 
  
    ```ini
    PING 10.37.129.9 (10.37.129.9): 56 data bytes
    Request timeout for icmp_seq 0
    Request timeout for icmp_seq 1
    ```

## 3. iptables 的基本使用

对 iptables 进行操作，其实就是对它的四种 “表” 进行 “增删改查” 操作。

### 3.1 启动 iptables

CentOS6 及之前都默认使用 iptables 防火墙，但 CentOS7 改为 firewalld，不过你还是可以自己安装 iptables。

* CentOS6 and before: 

    ```ini
    #启动 iptables
    `service iptables start`

    # 查看启动状态
    service iptables status

    # 停止 iptables
    service iptables stop

    # 重启 iptables(重启其实就是先 stop 再 start)
    service iptables restart

    # 重载就是重新加载配置的规则，在这里貌似跟重启一样
    service iptables reload
    ```

* CentOS7: 

    手动安装 iptables。

    ```ini
    #CentOS7 关闭 firewalld 防火墙:
    systemctl stop firewalld

    #CentOS7 关闭 firewalld 防火墙开机启动
    systemctl disable firewalld

    #CentOS7 安装 iptables 防火墙
    sudo yum -y install iptables

    #CentOS7 安装 iptables 的 service 启动工具
    sudo yum -y install iptables-services
    
    # 启动 iptables
    systemctl start iptables

    # 查看 iptables 状态
    systemctl status iptables

    # 停止 iptables
    systemctl stop iptables

    # 重启 iptables
    systemctl restart iptables

    # 重载 iptables
    systemctl reload iptables
    ```

稍微了解 Linux 的童鞋都知道，service 命令是用于启动 Linux 的进程的，但 `service iptables start` 并没有启动一个进程，因此查看它的状态用该命令 `service iptables status` 。

所以 `iptables` 其实不能叫 *Service*，因为它并没有一个“守护进程”，由于 *netfilter* 是内核功能，用户无法直接操作，`iptables` 这个工具是提供给用户设置过滤规则的，但最终这个过滤规则是由 *netfilter* 来执行的。

### 3.2 查询规则
命令格式: `iptables -L [参数]`

常用选项: 

* `-L`: list iptables，查看所有防火墙规则

    `iptables -L` 
    ```ini
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     all  --  anywhere             anywhere            state RELATED,ESTABLISHED
    ACCEPT     icmp --  anywhere             anywhere
    ACCEPT     all  --  anywhere             anywhere
    ACCEPT     tcp  --  anywhere             anywhere            state NEW tcp dpt:ssh
    REJECT     all  --  anywhere             anywhere            reject-with icmp-host-prohibited

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination
    REJECT     all  --  anywhere             anywhere            reject-with icmp-host-prohibited

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination
    ```

    * -L <chain_name>: 查看具体 chain 规则，**区分大小写**，如查看 “INPUT” 上的规则:

        `iptables -L INPUT`
        ```ini
        Chain INPUT (policy ACCEPT)
        target     prot opt source               destination
        ACCEPT     all  --  anywhere             anywhere            state RELATED,ESTABLISHED
        ACCEPT     icmp --  anywhere             anywhere
        ACCEPT     all  --  anywhere             anywhere
        ACCEPT     tcp  --  anywhere             anywhere            state NEW tcp dpt:ssh
        REJECT     all  --  anywhere             anywhere            reject-with icmp-host-prohibited
        ```

  * `Chain XXX`: XXX 链上的规则，`policy ACCEPT`: 默认策略是接受，显性设置，清晰且方便修改；

  * `target`: 目标动作或链，比如 ACCEPT(接受)、REJECT(拒绝)，以及自定义链；
    
  * `prot`: 协议；
    
  * `opt`: 选项；
    
  * `source`: 源地址(ip(可以是网段)/ 域名 / 主机名)；
    
  * `destination`: 目标地址(ip(可以是网段)/ 域名 / 主机名)；
    
  * 末列: 一些额外的信息


* `-t`: table to manipulate (default: `filter`)

* `-n`: 源和目标地址、端口以数字/数值的方式显示，否则默认会以域名/主机名/程序名等显示，该选项一般与 `-L` 合用

    ```ini
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED
    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0
    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22
    REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited
    ```

    其中 dpt:22 中的 dpt 是指 destination port(目标端口)，同理，spt 就是 source port(源端口)。

* `-v`: “verbose”，输出更加详细的信息
  
    ```ini
    iptables -v -L

    Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
    pkts bytes target     prot opt in     out     source               destination
    13627 1033K ACCEPT     all  --  any    any     anywhere             anywhere            state RELATED,ESTABLISHED
        0     0 ACCEPT     icmp --  any    any     anywhere             anywhere
        0     0 ACCEPT     all  --  lo     any     anywhere             anywhere
        2   128 ACCEPT     tcp  --  any    any     anywhere             anywhere            state NEW tcp dpt:ssh
    275 53284 REJECT     all  --  any    any     anywhere             anywhere            reject-with icmp-host-prohibited

    Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
    pkts bytes target     prot opt in     out     source               destination
        0     0 REJECT     all  --  any    any     anywhere             anywhere            reject-with icmp-host-prohibited

    Chain OUTPUT (policy ACCEPT 12506 packets, 1485K bytes)
    pkts bytes target     prot opt in     out     source               destination
    ```

    * pkts: packets，包的数量
    * bytes: 流过的数据包的字节数
    * in: 入站网卡
    * out: 出站网卡

    在很多情况下，Linux 的选项是可以合并的，若参数后面可添加参数则必须放在最后一位或单独写出

    正确示范：

    `iptables -nvL` 

    错误示范：
        
    `iptables -Lnv`


* `-x`: 以 bytes 为单位显示数据量，配合 `- v` 使用

* `--line-numbers`: 显示行号
  
    ```
    iptables -nvL --line-numbers 

    Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
    num   pkts bytes target     prot opt in     out     source               destination
    1    14739 1123K ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED
    2        0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0
    3        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
    4        2   128 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22
    5      305 55948 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited
    6        0     0 July_filter  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0
    ```

### 3.3 添加规则
命令格式: `iptables -I/A [参数]`

iptables 执行规则是根据文本顺序由上至下查找，找到第一条符合的规则则结束查找，因此规则放在文件开头与结尾执行效果会有差异。

  * `-I`: 在规则记录开头插入规则；

  * `-A`: 在规则记录结尾追加规则；
    
  * `-t`: 指定插入到哪个表中，默认为 “filter” 表；
    
  * `-s`: 匹配源 ip；
    
  * `-j`: 跳转指定跳转的目标，如自定义的链， action，如 ACCEPT、DROP、REJECT 等。

向 INPUT 的 filter 表中添加一条规则: 

`iptables -t filter -I INPUT -s 10.37.129.2 -j DROP`

filter 表中的 `INPUT` 链开头插入一条记录，将来自 `10.39.129.2` 的数据包丢弃

### 3.4 删除 iptables 中的规则记录

* `-D chain [rulenum]`: 删除符合条件的某链或某链中的某一条

    ```ini
    # 删除 filter 表中 INPUT 链的所有规则
    iptables -t filter -D INPUT

    # 删除 filter 表中 INPUT 链的第 2 条规则
    iptables -t filter -D INPUT 2

    # 删除 filter 表中 INPUT 链里符合该规则的记录
    iptables -t filter -D INPUT -s 10.37.129.2 -j DROP 
    ```


* `-F [chain]`: 清空某链或清空所有链路
  
   ```ini
   # 删除 filter 表中 INPUT 链的所有规则
   iptables -t filter -F INPUT

   # 删除 filter 表的所有规则
   iptables -t filter -F
   ```

### 3.5 修改规则

* `-R chain rulenum`: 修改指定链路的指定规则

    ```ini
    # 修改 filter 表中 INPUT 链的第 1 条规则
    iptables -t filter -R INPUT 1 -s 10.37.129.3 -j ACCEPT
    ```

* `-P chain target`: 修改指定链路的Policy

    ```ini
    # 修改 filter 表中 FORWARD 链的 Policy 为丢弃
    iptables -P FORWARD DROP
    ```

    尽管 man iptables 中有 iptables [-t table] -P chain target 这个说明，但我认为这是错的，不信你可以执行以下两条命令试试，不管先执行哪行，后面执行的总会替换前面执行的(即使它们指定的表不一样)，这就说明指定表跟不指定表是没区别的，因为默认规则是作用于整条链的，无法单独对表作用: 

    注: 这个结论还有待证明，我目前认为是这样。

### 3.6 保存规则

* 使用命令 `service iptables save`，保存时它会输出: 

`iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]`

保存到 `/etc/sysconfig/iptables` 文件中的。

* 直接编辑 `/etc/sysconfig/iptables`


### 3.7 更详细的命令
* `-d`: destination，用于匹配报文的目标地址，可以同时指定多个 ip(逗号隔开，逗号两侧都不允许有空格)，也可指定 ip 段: 
  
    ```
    iptables -t filter -I OUTPUT -d 192.168.1.111,192.168.1.118 -j DROP
    iptables -t filter -I INPUT -d 192.168.1.0/24 -j ACCEPT
    iptables -t filter -I INPUT ! -d 192.168.1.0/24 -j ACCEPT
    ```

* `-p`: 用于匹配报文的协议类型, 可以匹配的协议类型 tcp、udp、udplite、icmp、esp、ah、sctp 等
  

    ```ini
    iptables -t filter -I INPUT -p tcp -s 192.168.1.146 -j ACCEPT

    # 感叹号表示“非”，即除了匹配这个条件的都 ACCEPT，但匹配这个条件不一定就是 REJECT 或 DROP？这要看是否有为它特别写一条规则，如果没有写就会用默认策略: 
    iptables -t filter -I INPUT ! -p udp -s 192.168.1.146 -j ACCEPT
    ```
* `-i`: 用于匹配报文是从哪个网卡接口流入本机的，由于匹配条件只是用于匹配报文流入的网卡，所以在 OUTPUT 链与 POSTROUTING 链中不能使用此选项: 

    ```
    iptables -t filter -I INPUT -p icmp -i eth0 -j DROP
    iptables -t filter -I INPUT -p icmp ! -i eth0 -j DROP
    ```

* `-o`: 用于匹配报文将要从哪个网卡接口流出本机，于匹配条件只是用于匹配报文流出的网卡，所以在 INPUT 链与 PREROUTING 链中不能使用此选项。

    ```
    iptables -t filter -I OUTPUT -p icmp -o eth0 -j DROP
    iptables -t filter -I OUTPUT -p icmp ! -o eth0 -j DROP
    ```

## 4. iptables 扩展模块

### 4.1 tcp 扩展模块
* `-p tcp -m tcp --sport` 用于匹配 tcp 协议报文的源端口，可以使用冒号指定一个连续的端口范围(-p:stuck_out_tongue_winking_eye:rotocol，-m:match, 指匹配的模块，很多人可能以为是 module 的缩写，其实是 match 的缩写，--sport: source port)；

* `-p tcp -m tcp --dport` 用于匹配 tcp 协议报文的目标端口，可以使用冒号指定一个连续的端口范围(--dport:grinning:estination port): 

    ```ini
    iptables -t filter -I OUTPUT -d 192.168.1.146 -p tcp -m tcp --sport 22 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m tcp --dport 22:25 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m tcp --dport :22 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m tcp --dport 80: -j REJECT
    iptables -t filter -I OUTPUT -d 192.168.1.146 -p tcp -m tcp ! --sport 22 -j ACCEPT
    ```

此外，tcp 扩展模块还有–tcp-flags 选项，它可以根据 TCP 头部的 “标识位” 来匹配，具体直接点进去看吧。

### 4.2 multiport 扩展模块
  
* `-p tcp -m multiport --sports` 用于匹配报文的源端口，可以指定离散的多个端口号, 端口之间用”逗号”隔开;
* `-p udp -m multiport --dports` 用于匹配报文的目标端口，可以指定离散的多个端口号，端口之间用”逗号”隔开: 
  
    ```ini
    # 示例如下
    iptables -t filter -I OUTPUT -d 192.168.1.146 -p udp -m multiport --sports 137,138 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport --dports 22,80 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport ! --dports 22,80 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport --dports 80:88 -j REJECT
    iptables -t filter -I INPUT -s 192.168.1.146 -p tcp -m multiport --dports 22,80:88 -j REJECT
    ```

### 4.3 iprange 扩展模块
使用 iprange 扩展模块可以指定”一段连续的 IP 地址范围”，用于匹配报文的源地址或者目标地址。iprange 扩展模块中有两个扩展匹配条件可以使用: 
* `--src-range`(匹配源地址范围)
* `--dst-range`(匹配目标地址范围)

`iptables -t filter -I INPUT -m iprange --src-range 192.168.1.127-192.168.1.146 -j DROP`


### 4.4 string 扩展模块
假设我们访问的是 “http://192.168.1.146/index.html”，当“index.html” 中包括 “XXOO” 字符时，就会被以下规则匹配上: 

`iptables -t filter -I INPUT -m string --algo bm --string "XXOO" -j REJECT`

`-m string`: 表示使用 string 模块

`--algo bm`: 表示使用 bm 算法来匹配 index.html 中的字符串，“algo”是 “algorithm” 的缩写，另外还有一种算法叫“kmp”，所以 --algo 可以指定两种值，bm 或 kmp，貌似是 bm 算法速度比较快。

### 4.5 time 扩展模块
我们可以通过 time 扩展模块，根据时间段区匹配报文，如果报文到达的时间在指定的时间范围以内，则符合匹配条件。

* 设定时间区间

    ```
    iptables -t filter -I INPUT -p tcp --dport 80 -m time --timestart 09:00:00 --timestop 18:00:00 -j REJECT
    iptables -t filter -I INPUT -p tcp --dport 443 -m time --timestart 09:00:00 --timestop 18:00:00 -j REJECT
    ```

* 设定日期区间
    ```
    iptables -t filter -I INPUT -p tcp --dport 80 -m time --daystart 2019-07-20 --daystop 2019-07-25 -j REJECT
    ```

* 设定星期

    ```
    iptables -t filter -I INPUT -p tcp --dport 80 -m time --weekdays 6,7 -j REJECT
    iptables -t filter -I INPUT -p tcp --dport 443 -m time --weekdays 6,7 -j REJECT
    ```
    `--weekdays` 可用 1-7 表示一周的 7 天，还能用星期的缩写来指定匹配: Mon、Tue、Wed、Thu、Fri、Sat、Sun。

* 设定每月几号

    ```
    iptables -t filter -I INPUT -p tcp --dport 80 -m time --monthdays 22,23 -j REJECT
    ```

* 多条件相与或取反  

    ```
    iptables -t filter -I INPUT -p tcp --dport 80 -m time --weekdays 5 --monthdays 22,23,24,25,26,27,28 -j REJECT
    ```
    
    `--monthdays` 和 `--weekdays` 可以用感叹号! 取反，其他的不行。


### 4.6 connlimit 模块

使用 connlimit 扩展模块，可以限制每个 IP 地址同时链接到 server 端的链接数量，注意: 我们不用指定 IP，其默认就是针对”每个客户端 IP”，即对单 IP 的并发连接数限制。

* 限制 22 端口 (ssh 默认端口) 连接数量上限不能超过 2 个；

    ```
    iptables -t filter -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 2 -j REJECT
    ```

* 配合 --connlimit-mask 来限制网段: 
    ```
    iptables -t filter -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 2 --connlimit-mask 24 -j REJECT
    ```
    网址由 32 位二进制组成，最大可写成: 255.255.255.255，而 mask 就是掩码(网络知识，请自行了解)，24 表示 24 个 1，即 255.255.255.0。


### 4.7 limit 扩展模块
limit 模块是限速用的，用于限制“单位时间内流入的数据包的数量”。

每 6 位秒放行一下 ping 包(因为 1 分钟是 60 秒，所以 1 分钟 10 个包，就相当于每 6 秒 1 个包): 
```
iptables -t filter -I INPUT -p icmp -m limit --limit 10/minite -j ACCEPT
```
* `--limit`: 后面的单位除了 minite，还可以是 second、hour、day

* `--limit-burst`:  burst 是爆发、迸发的意思，在这里是指最多允许一次性有几个包通过，要理解 burst，先看以下的“令牌桶算法”。

    令牌桶算法: 
        有一个木桶，木桶里面放了 5 块令牌，而且这个木桶最多也只能放下 5 块令牌，所有报文如果想要出关入关，都必须要持有木桶中的令牌才行，这个木桶有一个神奇的功能，就是每隔 6 秒钟会生成一块新的令牌，如果此时，木桶中的令牌不足 5 块，那么新生成的令牌就存放在木桶中，如果木桶中已经存在 5 块令牌，新生成的令牌就无处安放了，只能溢出木桶（令牌被丢弃），如果此时有 5 个报文想要入关，那么这 5 个报文就去木桶里找令牌，正好一人一个，于是他们 5 个手持令牌，快乐的入关了，此时木桶空了，再有报文想要入关，已经没有对应的令牌可以使用了，但是，过了 6 秒钟，新的令牌生成了，此刻，正好来了一个报文想要入关，于是，这个报文拿起这个令牌，就入关了，在这个报文之后，如果很长一段时间内没有新的报文想要入关，木桶中的令牌又会慢慢的积攒了起来，直到达到 5 个令牌，并且一直保持着 5 个令牌，直到有人需要使用这些令牌，这就是令牌桶算法的大致逻辑。

    看完了“令牌桶算法”，其实 --limit 就相当于指定“多长时间生成一个新令牌”，而 --limit-burst 则用于指定木桶中最多存放多少块令牌。

### 4.8 udp 扩展模块
udp 扩展模块中能用的匹配条件比较少，只有两个，就是 `--sport` 与 `--dport`，即匹配报文的源端口与目标端口。

* 放行 `samba` 服务的 137 和 138 端口: 

    ```
    iptables -t filter -I INPUT -p udp -m udp --dport 137 -j ACCEPT
    iptables -t filter -I INPUT -p udp -m udp --dport 138 -j ACCEPT
    ```

    当使用扩展匹配条件时，如果未指定扩展模块，iptables 会默认调用与 -p 对应的协议名称相同的模块，所以，当使用 -p udp 时，可以省略 -m udp: 

    ```
    iptables -t filter -I INPUT -p udp --dport 137 -j ACCEPT
    iptables -t filter -I INPUT -p udp --dport 138 -j ACCEPT
    ```

* udp 扩展中的 `--sport` 与 `--dport` 与 tcp 一样，同样支持指定一个连续的端口范围: 

    ```
    iptables -t filter -I INPUT -p udp --dport 137:157 -j ACCEPT
    ```
    `--dport 137:157` 表示匹配 137-157 之间的所有端口。

    另外与 tcp 一样，udp 也能使用 `--multiport` 指定多个不连续的端口。

### 4.9 icmp 扩展模块
* ping 是使用 icmp 协议的，假设要禁止所有 icmp 协议的报文进入本机(根据前面所说，我们可以省略用 -m icmp 来指定使用 icmp 模块，因为不指定它会默认使用 -p 指定的协议对应的模块): 

    ```
    iptables -t filter -I INPUT -p icmp -j REJECT
    ```

    上述命令能产生两个效果: 
    – 1、别人 ping 本机时，无法 ping 通，因为数据报文无法进入；
    – 2、本机 ping 别人时，虽然数据包可以出去，但别人的响应包也是 icmp 协议，无法进来(即“有去无回”)。
    所以这样设置会导致，不止别人 ping 不通本机，本机也 ping 不通别人。

    很明显上边的规则不是我们想要的，我们想要的一般都是允许本机 ping 别人，不允许别人 ping 本机: 

    ```
    iptables -t filter -I INPUT -p icmp --icmp-type 8/0 -j REJECT
    ```

* `--icmp-type 8/0` 用于匹配报文 type 为 8，code 为 0 时才会被匹配到，至于什么是 type 和 code，这是 icmp 协议的知识

    其实上边的命令还可以省略 code(即把 “8/0” 写成 “8” 即可，省略掉“/0”，原因是 type=8 的报文中只有 code=0 一种，所以我们不写默认就是 code=0，不会有其它值):

    ```
    iptables -t filter -I INPUT -p icmp --icmp-type 8 -j REJECT
    ```

    除了能用 type/code 来匹配 icmp 报文，还可以使用 icmp 的描述名称来匹配: 

    ```
    iptables -t filter -I INPUT -p icmp --icmp-type "echo-request" -j REJECT
    ```

    `--icmp-type "echo-request"` 的效果与 `icmp --icmp-type 8/0` 或 `icmp --icmp-type 8` 的效果完全一样 (你可能发现了，icmp 协议的描述“echo-request” 其实是“echo request”，只不过我们用于作为匹配条件时，要把空格换成横杠)。

### 4.10 state 扩展模块
在 TCP/IP 协议簇中，UDP 和 ICMP 是没有所谓的连接的，但是对于 state 模块来说，tcp 报文、udp 报文、icmp 报文都是有连接状态的，我们可以这样认为，对于 state 模块而言，只要两台机器在”你来我往”的通信，就算建立起了连接。

而”连接”中的报文可以分为 5 种状态，报文状态可以为 NEW、ESTABLISHED、RELATED、INVALID、UNTRACKED。

* 放行 RELATED 和 ESTABLISHED 状态的报文: 

    ```
    iptables -t filter -I INPUT -m state --state RELATED, ESTABLISHED -j ACCEPT
    ```
