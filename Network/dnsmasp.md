# dnsmasq 






```
#!/bin/sh
#ip route add 8.8.8.8 dev $1
iptables -t mangle -A fwmark -m set --match-set gfwlist dst -j MARK --set-mark 0xffff
ip rule add fwmark 0xffff table gfwtable
ip route add default dev $1 table gfwtable
iptables -I FORWARD -o $1 -j ACCEPT
iptables -t nat -I POSTROUTING -o $1 -j MASQUERADE
```


```

 daemon.notice openvpn(myvpn)[24185]: SIGTERM[hard,] received, process exiting
 openvpn(myvpn): Multiple --up scripts defined.  The previously configured script is overridden.
 openvpn(myvpn): Multiple --down scripts defined.  The previously configured script is overridden.

 daemon.notice openvpn(myvpn): net_route_v4_best_gw query: dst 0.0.0.0
 daemon.notice openvpn(myvpn): net_route_v4_best_gw result: via 192.168.3.1 dev eth0
 daemon.notice openvpn(myvpn): TUN/TAP device tun0 opened
 daemon.notice openvpn(myvpn): net_iface_mtu_set: mtu 1500 for tun0
 daemon.notice openvpn(myvpn): net_iface_up: set tun0 up
 daemon.notice openvpn(myvpn): net_addr_ptp_v4_add: 10.37.1.10 peer 10.37.1.9 dev tun0
 daemon.notice openvpn(myvpn): /usr/libexec/openvpn-hotplug up myvpn tun0 1500 1557 10.37.1.10 10.37.1.9 init
 daemon.notice openvpn(myvpn): net_route_v4_add: 69.12.68.54/32 via 192.168.3.1 dev [NULL] table 0 metric -1
 daemon.notice openvpn(myvpn): net_route_v4_add: 0.0.0.0/1 via 10.37.1.9 dev [NULL] table 0 metric -1
 daemon.notice openvpn(myvpn): net_route_v4_add: 128.0.0.0/1 via 10.37.1.9 dev [NULL] table 0 metric -1
 daemon.notice openvpn(myvpn): net_route_v4_add: 10.37.0.1/32 via 10.37.1.9 dev [NULL] table 0 metric -1
 daemon.warn openvpn(myvpn): WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
 daemon.notice openvpn(myvpn): Initialization Sequence Completed
```