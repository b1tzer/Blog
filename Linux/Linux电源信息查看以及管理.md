# Linux 电源管理


## 查看电源信息

使用 UPower

    upower -i `upower -e | grep 'BAT'`

    
使用 TLP

    sudo apt install tlp

    # 查看电池相关信息
    sudo tlp-stat –b

    # 查看更多信息
    sudo tlp-stat –s

安装TLP 后，其配置文件为/etc/default/tlp，您将可以使用以下命令：

- tlp – 应用笔记本电脑省电设置
- tlp-stat – 显示所有省电设置
- tlp-pcilist – 显示 PCI(e) 设备数据
- tlp-usblist – 用于查看 USB 设备数据

## 使用 TLP 更改笔记本电源策略

    # 查看 TLP 服务状态
    sudo systemctl status tlp

配置文件路径，由上至下读取文件，最后的设置生效

- 内在默认值
- `/etc/tlp.d/*.conf`：插入自定义片段，按词法（字母）顺序阅读
- `/etc/tlp.conf` : 用户配置


配置如下

```ini
# 超过多少秒开启音频节能 单位秒 0禁用节能
SOUND_POWER_SAVE_ON_AC=1
SOUND_POWER_SAVE_ON_BAT=1

# 关闭声卡以及控制器电源
SOUND_POWER_SAVE_CONTROLLER=Y

# 电池保养 类似苹果的电池充电计划
# 分别设置不同电池 可以通过前面的命令查看电池信息 BAT0使用所有电池
# 电磁掉到75
START_CHARGE_THRESH_BAT0=75
STOP_CHARGE_THRESH_BAT0=80

START_CHARGE_THRESH_BAT1=75
STOP_CHARGE_THRESH_BAT1=80

# 配置生效
sudo tlp start
```
