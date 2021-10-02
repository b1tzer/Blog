
# Linux 初始化设置

每次安装 Linux 都要设置一番，故把自己习惯的设置记录下来，防止每次设置都要百度一遍。


1. 设置电源永不休眠

2. 设置固定IP

3. 设置合盖不休眠

4. 更新软件并安装常用软件
   1. 根据自己的网络环境是否更换软件源
        ```ini
        # 配置文件为 /etc/apt/sources.ist

        # Ubuntu 20.04 使用阿里云的源
        deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

        deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

        deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

        deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

        deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
        deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse


        
        ```
   2. 更新软件
        ```ini
        sudo apt update
        sudo apt upgrade
        ```
   3. 安装常用软件
        - 安装 ssh server, git, vim, zsh
        ```ini
        sudo apt install openssh-server git vim zsh -y

        # 安装 oh-my-zsh 脚本
        sudo sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

        ```

        - 安装 Docker、Portainer-ce

        ```ini
        # 安装 Docker
        sudo snap install docker

        # 创建卷
        sudo docker volume create portainer_data

        # 安装 Portainer-ce，docker webUI 管理系统        
        sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
            --restart=always \
            -v /var/run/docker.sock:/var/run/docker.sock \
            -v portainer_data:/data \
            portainer/portainer-ce:latest
        
        # 在网页中输入 https://localhost:9443 即可开始初始化
        ```


    4. 个人需求服务
   
        - 安装 Bark iOS 自定义推送服务
        ```ini
        sudo docker run -dt --name bark -p 8080:8080 \
            --mount 'type=volume,src=bark_data,dst=/data,volume-driver=local' \
            finab/bark-server
        ```

        - 安装 Home Assistant
        ```


        ```






