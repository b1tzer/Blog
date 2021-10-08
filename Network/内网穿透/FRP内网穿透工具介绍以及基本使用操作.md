# frp 内网穿透工具

## 设备环境要求

* 局域网内有低功耗24小时待机设备（当然不在乎电量的拿主机也不是不可以）
* 有公网服务器及IP

## 目的（可玩方向）

* 将局域网内网站挂载到公网上。数据由于各种原因不愿意放在云上的。或者网站需要与局域网内设备通信
* 通过 SSH 访问内网机器
* 转发 DNS 查询请求
* 转发 Unix 域套接字
* 对外提供简单的文件访问服务
* 为本地 HTTP 服务启用 HTTPS
* 安全的暴露内网服务
* 点对点内网穿透 （流量无需经过服务器）

## [工具下载](https://github.com/fatedier/frp/releases)

下载地址 https://github.com/fatedier/frp/releases ，目前可以在 Github 的 Release 页面下载到最新版本的客户端和服务器端二进制文件。

## 运行
1. 解压缩下载的压缩包，将其中的 frpc 拷贝到内网服务所在的机器上，将 frps 拷贝到具有公网 IP 的机器上，放置在任意目录。***文件命名规则 c 为 client端，s 则为 server端。***

2. 在具有公网 IP 的机器上部署 frps，修改 frps.ini 文件，这里使用了最简化的配置，设置了 frp 服务器用户接收客户端连接的端口，同时勿忘开放7000端口的防火墙
  ```ini
    [common]
    bind_port = 7000
  ```
![云服务器防火墙配置](https://i.loli.net/2021/09/23/K56xDOXzJlSrWs7.png)

3. 在需要被访问的内网机器上部署 frpc，修改 frpc.ini 文件（这里以开放 SSH 功能为例，SSH 服务通常监听在 22 端口），假设 frps 所在服务器的公网 IP 为 x.x.x.x：
    ```ini
    [common]
    server_addr = x.x.x.x
    server_port = 7000

    [ssh]
    type = tcp
    local_ip = 127.0.0.1
    local_port = 22
    remote_port = 6000
    ```

    `local_ip` 和 `local_port` 配置为本地需要暴露到公网的服务地址和端口。`remote_port` 表示在 frp 服务端监听的端口，访问此端口的流量将会被转发到本地服务对应的端口。

4. 先通过 ./frps -c ./frps.ini 启动服务端，再通过 ./frpc -c ./frpc.ini 启动客户端。如果需要在后台长期运行，建议结合其他工具使用，例如 systemd 和 supervisor。
   1. frp v0.37 自带服务文件,稍加配置即可使用
   2. 将 frpc、frps 拷贝到 /usr/bin
   3. 将 frpc.ini、frps.ini 拷贝到 /etc/frp/
   4. 将 systemd/frpc.service、frps.service 拷贝到 /lib/systemd/system/
   5. sudo systemctl start frps.service

5. 此时我们就可以通过公网 IP 访问 SSH 服务。假设用户名为test：

    `ssh -oPort=6000 test@x.x.x.x`

    frp 会将请求 `x.x.x.x:6000` 的流量转发到内网机器的 22 端口。

## 常见服务配置及讲解

### 格式

目前仅支持 ini 格式的配置，如下的示例配置将本地 SSH 服务穿透到公网。

frps 配置：

```ini
[common]
bind_port = 7000
```

frpc 配置：

```ini
[common]
server_addr = x.x.x.x
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

`[common]` 是固定名称的段落，用于配置通用参数。

`[ssh]` 仅在 frpc 中使用，用于配置单个代理的参数。代理名称必须唯一，不能重复。

同一个客户端可以配置多个代理。

### 身份认证
目前 frpc 和 frps 之间支持两种身份验证方式，`token` 和 `oidc`，默认为 `token`。

通过 `frpc.ini` 和 `frps.ini` 的 `[common]` 段落中配置 `authentication_method` 来指定要使用的身份验证方式。

只有通过身份验证的客户端(frpc)才能成功连接 frps。

1. Token: 基于 Token 的身份验证方式比较简单，需要在 frpc 和 frps 的 `[common]` 段落中配置上相同的 `token` 参数即可。

2. OIDC: OIDC 是 `OpenID Connect` 的简称，验证流程参考 [Client Credentials Grant](https://tools.ietf.org/html/rfc6749#section-4.4)。

### web界面管理
目前 frpc 和 frps 分别内置了相应的 Web 界面方便用户使用。

1. 服务端 Dashboard 使用户可以通过浏览器查看 frp 的状态以及代理统计信息。

**注：Dashboard 尚未针对大量的 proxy 数据展示做优化，如果出现 Dashboard 访问较慢的情况，请不要启用此功能。**

需要在 frps.ini 中指定 dashboard 服务使用的端口，即可开启此功能：

```ini
# frps.ini
[common]
dashboard_port = 7500
# dashboard 用户名密码，可选，默认为空
dashboard_user = admin
dashboard_pwd = admin
```

打开浏览器通过 `http://[server_addr]:7500` 访问 Dashboard 界面，输入用户名密码 `admin`。

2. 客户端管理界面

frpc 内置的 Admin UI 可以帮助用户通过浏览器来查询和管理客户端的 proxy 状态和配置。

需要在 frpc.ini 中指定 admin 服务使用的端口，即可开启此功能：

```ini
# frpc.ini
[common]
admin_addr = 127.0.0.1
admin_port = 7400
admin_user = admin
admin_pwd = admin
```

打开浏览器通过 `http://127.0.0.1:7400` 访问 Admin UI。

如果想要在外网环境访问 Admin UI，可以将 7400 端口通过 frp 映射出去即可，但需要重视安全风险。

```ini
# frpc.ini
[admin_ui]
type = tcp
local_port = 7400
remote_port = 7400
```


## 更详细的功能及参数可以参考该项目[官网](https://gofrp.org/docs/)