# 智能家居系统 Home Assistant 安装

## 简介
[官网介绍](https://www.home-assistant.io/)：将本地控制和隐私放在首位的开源家庭自动化系统。由世界各地的工程师和DIY爱好者社区提供动力。完美运行在树莓派或本地服务器。

该系统支持多种智能家居平台，目前实测小米和HomeKit是完美支持的。非常适合家里买了好几个平台的或者手持iPhone但买了小米家设备的情况。

## 安装方式选择
官网提供了三种形式的HASS安装
- 独立的 `Linux` 操作系统
- `docker` 容器
- `python` 软件包

其中 `docker` 容器还有一个分支叫 `Container Supervised` 版，基础三个版本安装官网都有详细说明，基本照着步骤不会有问题，[点此直接访问官网安装流程](https://www.home-assistant.io/installation/)

我自己捣鼓了很久，最后还是选择了 Supervised 版，因为我想将我自己的电脑加入智能家居系统，希望可以随时查看电脑运行状态，尤其是这个系统主机的运行情况，并实现远程开关机局域网内的电脑，发现基础款都不支持添加 add-on 组件，只能强行被迫安装这个版本。安装过程记录一下。

## 安装
先放上[官网的安装说明链接](https://github.com/home-assistant/supervised-installer)，一开始自己也是找了半天。我是安装在我的闲置笔记本，CPU为AMD 瑞龙5 2400U，内存8G。官网推荐使用树莓派安装。

### 确认安装环境是否都符合
  - `Docker CE` >= 19.03
  - `Systemd` >= 239
  - `NetworkManager` >= 1.14.6
  - `AppArmor` == 2.13.x (built into the kernel)
  - Debian Linux Debian 11 aka Bullseye (no derivatives)
  - Home Assistant OS-Agent (Only the latest release is supported)

网络环境可能需要注意一下，Github 时好时坏，可以裸连，当然你有工具那更是稳定了。

只要安装的是新系统就基本没问题，Linux对硬件要求不高，升级也没什么压力。注意系统为 Debian Linux，实测使用 Ubuntu 20.04 也可以。刚安装的 Ubuntu 20.04 Server 实测仅缺少 `jq` 安装包，使用 `sudo apt install jq` 安装。
```
sudo apt install jq
sudo curl -Lo installer.sh https://raw.githubusercontent.com/home-assistant/supervised-installer/master/installer.sh
bash installer.sh
```

我在安装中最主要的是遇到网络问题。查看 installer.sh 文件，将其中的链接复制下来，直接下载。
当前脚本有个 10s 思考反悔时间，可以进文件直接注释掉。
```ini
# 注释掉等待，每次失败都要再次等待10s运行
#sleep 10
```

网络问题还有一个解决方案，使用国内 Gitee 克隆该项目，该脚本非常贴心的将主 url 设置了单独变量，直接设置 `URL_RAW_BASE` 设置为自己的仓库即可

## 使用

安装完成后直接在浏览器访问 `http://server_ip:8123`，我的主机 IP 为 `192.168.3.2`，则直接访问 `http://192.168.3.2:8123`，登录进去就是设置界面。

### 下面就是连接小米
我们需要先安装 HACS，HASS 的社区商店，有很多爱好者会上传一些插件控制设备，可以[点此处访问官网](https://hacs.xyz/)，安装插件（HASS 里称为集成）的方式是在**配置-->集成-->添加集成**，HACS 充当一个扩充官方插件库的角色，在 HACS 中下载的自定义插件，就可以在官方的插件库中找到，**最终我们安装插件还是要到这个官方的这条路**。通过 HACS 安装小米插件，即可控制小米设备。

1. 确认环境
   - Home Assistant 版本 2021.2.0 及以后
   - Github 账号
   - A supported Home Assistant installation
   - Access to the filesystem where Home Assistant is located
   - You know how to access the Home Assistant log file
   - **A stable internet connection with sufficient available data or no data caps** 这个要注意，在2021.10月后发现，可能网络不稳定多次访问 Github 会被限流。

2. 进入 Home Assistant config 目录下，下载 HACS 文件
   可以通过 `portainer` 查看 hass 容器的配置，`/config` 路径对应的主机路径为 `/usr/share/hassio/homeassistant`
    ```
    sudo cd /usr/share/hassio/homeassistant
    wget -O - https://get.hacs.xyz | bash -
    ```

3. 在 HASS 页面中，点击 配置-->集成-->添加集成，搜索 HACS，找到添加即可，其中需要点击 Github 的授权页面，将授权码输入到 认证页面即可。如果搜索不到，可以尝试清除浏览器缓存，或者换一个浏览器，换一台设备也行。安装完成就可以在菜单栏看到 HACS 了。耐心等待 HACS 显示已完成，等待时间不等，看 Github 的网络连接情况

    ![hacs](https://i.loli.net/2021/10/08/ObVNJkei8IdaL7z.png)

    查看日志，可以看到我的网络就不太行。已经被限流

    ![github_limit](https://i.loli.net/2021/10/08/ZTpD8UHm3SLJncR.png)

4. 等待显示已完成，我们就在 HACS 中安装自定义集成，点击浏览并添加存储库，搜索 xiaomi miot，不带任何后缀。添加到存储裤后，再返回 HACS 界面等待安装完成，完成后会提示重启，然后重启 HASS 即可。
   
5. 到官方集成添加 xiaomi miot 集成，登陆小米的账号，选择设备，等待完成，即可在概览看到设备。

### 连接 iphone，使用 iPhone 控制米家设备
1. 在官方集成中搜索 HomeKit，安装后会有一个通知出现。
2. 打开 iPhone 手机的家庭，添加设备，扫一下通知里面出现的二维码，即可添加。
3. 过一会应该就可以了。
   
恭喜，到这一步，使用 iPhone 控制米家设备就已经基本完成了。有关控制局域网内的电脑开关机我会在后面的文章中介绍。
