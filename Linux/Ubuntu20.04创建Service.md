# Ubuntu 20.04 创建 Service

## 新建 service 文件
`sudo vim /lib/systemd/system/frps.service`

## 写入代码
```bash
[Unit]
Description=frps daemon
 
[Service]
Type=simple
#此处把/root/frp_linux_arm64替换成 你的frps的实际安装目录
ExecStart=/root/frp_linux_arm64/frps -c /root/frp_linux_arm64/frps.ini
 
[Install]
WantedBy=multi-user.target
```

## 启动

`frps sudo systemctl start frps`

## 自启动

`sudo systemctl enable frps`