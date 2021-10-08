
# Linux 初始化设置

每次安装 Linux 都要设置一番，故把自己习惯的设置记录下来，防止每次设置都要百度一遍。


1. 设置电源永不休眠

2. 设置固定IP

3. 设置合盖不休眠

4. 设置 SSH 密钥登录
   1. 
   
5. 更新软件并安装常用软件
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

        # 云服务器安装 Docker
        # 卸载老版本
        sudo apt remove docker docker-engine docker.io containerd runc
        # 添加软件源仓库
        sudo apt update
        sudo apt install
        install \
            apt-transport-https \
            ca-certificates \
            curl \
            gnupg \
            lsb-release
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
        echo \
        "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        # 安装
        sudo apt-get update
        sudo apt-get install docker-ce docker-ce-cli containerd.io

        # 创建卷
        sudo docker volume create portainer_data

        # 安装 Portainer-ce，docker webUI 管理系统        
        sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
            --restart=always \
            -v /var/run/docker.sock:/var/run/docker.sock \
            -v portainer_data:/data \
            portainer/portainer-ce:latest
        
        # 在网页中输入 https://localhost:9443 即可开始初始化

        # 如果需要远程管理 Docker， 可以安装 Portainer Agent
        # 如果使用 snap 安装 docker，下面的路径就要更换实际 volume 所在路径
        docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent:latest
        ```

    1. 个人需求服务
   
        - 安装 Bark iOS 自定义推送服务
        ```ini
        sudo docker run -dt --name bark -p 8080:8080 \
            --mount 'type=volume,src=bark_data,dst=/data,volume-driver=local' \
            finab/bark-server
        ```
        
        - 远程开关机 windows

        ```
        # 安装wakeonlan
        wakeonlan -i 192.168.3.255 -p 9 40:8D:5C:57:3B:19

        # 安装 Samba-common 套件
        sudo apt install samba-common
        net rpc shutdown -I client_ip -U username%password
        ```
        - Blog

        ```
        docker pull halohub/halo:1.4.12
        docker run -it -d --name halo -p 8090:8090 -v ~/.halo:/root/.halo --restart=unless-stopped halohub/halo:1.4.12
        ```




