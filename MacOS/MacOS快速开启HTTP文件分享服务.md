# Mac OS 快速开启 HTTP 文件共享服务

Mac OS 自带 Apache Python 组建，可以通过 Simple HTTP Server 模版快速启动

1. 启动 apache 服务

    `sudo apachectl start`

2. cd 到要共享的目录下，启动 Simple HTTP Server

   `python -m SimpleHTTPServer`
