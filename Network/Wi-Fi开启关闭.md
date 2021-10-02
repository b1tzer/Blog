# 网卡开启关闭


* 查看网卡

`ip a`


* 启动关闭网卡
    ```
    sudo ifconfig wlp3s0 down
    sudo iwconfig wlp3s0 mode monitor
    sudo ifconfig wlp3s0 up
    ```

* 编辑Wi-Fi默认配置文件
    
    ```bash
    sudo vim /etc/netplan/00-installer-config-wifi.yaml

    # This is the network config written by 'subiquity'
    network:
    version: 2
    wifis:
        wlp1s0:
        #dhcp4: true
        access-points:
            "whats_up_guys":
            password: "Aa983843"
        addresses: [192.168.3.4/24]
        gateway4: 192.168.3.1
        nameservers:
            addresses: [144.144.144.144]
    ```
    
* 生成配置
  
    `sudo netplan generate`

* 生效
    `sudo netpaln apply`

