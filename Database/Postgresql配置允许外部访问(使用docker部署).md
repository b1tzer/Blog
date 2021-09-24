# Docker 部署 PostgreSQL 

直接命令上手，一键完成。

```bash
$ docker run -d --name postgres --restart always \
-e POSTGRES_USER='postgres' \
-e POSTGRES_PASSWORD='abc123' \
-e ALLOW_IP_RANGE=0.0.0.0/0 \
-v /home/postgres/data:/var/lib/postgresql \
-p 5432:5432 \
-d postgres
```
`--name` 自定义容器名称

`-e ALLOW_IP_RANGE` 允许所有ip访问，如果不加，则非本机ip访问不了

`-e POSTGRES_USER=abcuser` 用户名

`-e POSTGRES_PASS=‘abc123'` 指定密码

`-p 5432:5432` 指定端口映射，主机：容器

`-v /path` 映射路径，本地路径：容器路径


## 已经提前安装好 Postgresql 的，可以通过修改配置文件允许外部访问
1. 查看容器名:

    `docker container list`

2. 进入 PostgreSQL 容器控制台:

    `docker exec -it xxxx(这里为找到的容器名) bash` 可以使用 `Tab` 键自动补齐

3. 进入 pqsl 控制台，找到配置文件目录:

    ```shell
    root:/# pqsl
    postgresql=# show config_file;
               config_file
    -----------------------------------------
    var/lib/postgresql/data/postgresql.conf
    (1 row)
    postgresql=# \q   //quit
    ```

4. 修改该目录下的 `pg_hba.conf` 和 `postgresql.conf`

    `vim /var/lib/postgresql/data/pg_hba.conf`
    
    找到如下图所示 `IPv4 local connection` 下面的参数，根据自己需求修改允许的IP，局域网内则设置为`192.168.1.0/24`，全部允许则设置为 `0.0.0.0/0`

    ![pg_hba.conf](https://i.loli.net/2021/09/24/VLslibORMToE31h.png)

    `vim /var/lib/postgresql/data/postgresql.conf`

    修改 `listen_address = '*'` , 可以在 `vim控制台` 输入 `:/address` 快速搜索。
    这一步看起来不用做，当前时间下载的最新版docker镜像默认为 `*` 

    ![postgresql.conf](https://i.loli.net/2021/09/24/HhGcnSa3u1MPwId.png)

5. 重启服务
    `service postgresql force-reload`

## 现在就可以在非本机访问该服务了。详细咨询可以访问[官网](https://hub.docker.com/_/postgres)

