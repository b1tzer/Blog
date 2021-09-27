# Linux 部署 OpenWRT 步骤

## 打开网卡混杂模式
`wlp1s0` 替换为自己的网卡名
`sudo ip link set wlp1s0 promisc on`

## 创建网络
(须结合实际网络情况，不能照抄命令)

`docker network create -d macvlan --subnet=192.168.3.0/24 --gateway=192.168.3.1 -o parent=wlp1s0 macnet`

## 使用 docker network ls命令可以看到网络macnet已建立成功：

`docker network ls`

## 拉取镜像
若身处国内，为提高拉取速度，请拉取阿里云仓库中的镜像：

`docker pull sulinggg/openwrt:x86_64`


## 创建并启动容器
`docker run --restart always --name openwrt -d --network macnet --privileged sulinggg/openwrt:x86_64 /sbin/init`

`--restart always`参数表示容器退出时始终重启，使服务尽量保持始终可用；

`--name openwrt`参数定义了容器的名称；

`-d`参数定义使容器运行在 Daemon 模式；

`--network macnet`参数定义将容器加入 maxnet网络；

`--privileged` 参数定义容器运行在特权模式下；

`registry.cn-shanghai.aliyuncs.com/suling/openwrt:latest` 为 Docker 镜像名，因容器托管在阿里云 Docker 镜像仓库内，所以在镜像名中含有阿里云仓库信息；

`/sbin/init`定义容器启动后执行的命令。

启动容器后，我们可以使用 `docker ps -a` 命令查看当前运行的容器：

```bash
$ docker ps -a
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
a26cee7cade6 openwrt:latest "/sbin/init" 3 hours ago Up 3 hours openwrt
```

若容器运行信息STATUS列为 UP状态，则说明容器运行正常。

## 进入容器并修改相关参数
(须结合实际网络情况，不能照抄配置)

`docker exec -it openwrt bash`
其中：

`openwrt` 为容器名称；

`bash` 为进入容器后执行的命令。

执行此命令后我们便进入 OpenWrt 的命令行界面，首先，我们需要编辑 OpenWrt 的网络配置文件：

`vim /etc/config/network`
我们需要更改 Lan 口设置：

```bash
config interface 'lan'
option type 'bridge'
option ifname 'eth0'
option proto 'static'
option ipaddr '192.168.123.100'
option netmask '255.255.255.0'
option ip6assign '60'
option gateway '192.168.123.1'
option broadcast '192.168.123.255'
option dns '192.168.123.1'
```
其中：

所有的 `192.168.123.x` 需要根据树莓派所处网段修改，`option gateway` 和 `option dns` 填写路由器的 `IP`，若树莓派获得的 `IP` 为 `192.168.2.154`，路由器 `IP` 为 `192.168.2.1`，则需要这样修改：

```
config interface 'lan'
option type 'bridge'
option ifname 'eth0'
option proto 'static'
option ipaddr '192.168.2.100'
option netmask '255.255.255.0'
option ip6assign '60'
option gateway '192.168.2.1'
option broadcast '192.168.2.255'
option dns '192.168.2.1'
```
`option ipaddr` 项目定义了 OpenWrt 的 `IP` 地址，在完成网段设置后，IP最后一段可根据自己的爱好修改（前提是符合规则且不和现有已分配 IP 冲突）。

## 重启网络
`/etc/init.d/network restart`

## 进入控制面板
在浏览器中输入第 5 步 `option ipaddr` 项目中的 `IP` 进入 Luci 控制面板，若 `option ipaddr` 的参数为 `192.168.123.100`，则可以在浏览器输入 `http://192.168.123.100` 进入控制面板。

用户名：`root`

密码：`password`