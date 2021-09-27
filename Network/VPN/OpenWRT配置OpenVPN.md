# OpenWRT 配置 OpenVPN client

1. 从 VPN 服务提供商处下载 .ovpn 配置文件
    ```ini
    $ cat /etc/openvpn/my.ovpn
    dev tun
    fast-io
    persist-key
    persist-tun
    nobind
    remote usa-losangeles5-ca-version-2.expressnetw.com 1195

    remote-random
    pull
    comp-lzo no
    tls-client
    verify-x509-name Server name-prefix
    ns-cert-type server
    key-direction 1
    route-method exe
    route-delay 2
    tun-mtu 1500
    fragment 1300
    mssfix 1200
    verb 3
    cipher AES-256-CBC
    keysize 256
    auth SHA512
    sndbuf 524288
    rcvbuf 524288
    # 此处指定密码文件，第一行为用户名，第二行为密码
    # 或者在总配置文件配置 /etc/config/openvpn 添加此参数，则不用为每一个配置文件指定密码文件路径
    auth-user-pass '/etc/openvpn/passwd' 

    <cert>
    -----BEGIN CERTIFICATE-----
    XXXXXXXXXXXXXXXXXXXX
    -----END CERTIFICATE-----
    </cert>
    <key>
    -----BEGIN RSA PRIVATE KEY-----
    XXXXXXXXXXXXXXXXXXXX
    -----END RSA PRIVATE KEY-----
    </key>
    <tls-auth>
    #
    # 2048 bit OpenVPN static key
    #
    -----BEGIN OpenVPN Static key V1-----
    XXXXXXXXXXXXXXXXXXXX
    -----END OpenVPN Static key V1-----
    </tls-auth>
    <ca>
    -----BEGIN CERTIFICATE-----
    XXXXXXXXXXXXXXXXXXXX
    -----END CERTIFICATE-----
    </ca>
    ```

2. 配置 OpenVPN

    ```ini
    $ cat /etc/config/openvpn
    config openvpn 'myvpn'
        option status '/var/log/openvpn_status.log'
        option log '/tmp/openvpn.log'
        # 此处文件为第一步配置好的文件
        option config '/etc/openvpn/my2.ovpn'

        #option auth-user-pass '/etc/openvpn/passwd'
        #option auth-nocache
        #option up '/etc/openvpn/openvpn-up.sh'
        #option down '/etc/openvpn/openvpn-down.sh'
    ```

3. 开启 OpenVPN 服务

    ```ini
    $ /etc/init.d/openvpn enable
    $ /etc/init.d/openvpn start
    ```

4. 查看日志

    `logread -e openvpn; netstat -lnp | grep openvpn`

    查看日志，是否有报错信息

    * Auth failed
    : 服务器授权失败，确认账户是否正常，密码是否正确，参数配置是否正常

    * TLS connection out of time
    : 连接服务器超时，再次尝试或者更换服务器

    * can't ask for 'Enter Auth Username:'
    : 查看配置文件是否有 `auth-user-pass` 参数未指定密码文件，作为服务不允许接收控制台输入 
     





