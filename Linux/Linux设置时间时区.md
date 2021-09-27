# Linux 查看设置时间以及时区

## 小知识

**CST**：中国标准时间（China Standard Time），这个解释可能是针对RedHat Linux。

**UTC**：协调世界时，又称世界标准时间，简称UTC，从英文国际时间/法文协调时间”Universal Time/Temps Cordonné”而来。中国大陆、香港、澳门、台湾、蒙古国、新加坡、马来西亚、菲律宾、澳洲西部的时间与UTC的时差均为+8，也就是UTC+8。

**GMT**：格林尼治标准时间（旧译格林威治平均时间或格林威治标准时间；英语：Greenwich Mean Time，GMT）是指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。

## 查看时间时区
```
~$ date -R
Mon, 27 Sep 2021 05:31:49 +0000
```

北京时间为东八区+0800，使用 `tzselect` 交互命令***查看时区名称***，东八区可以写为 `Asia/Hong_Kong`.

Example：

```
~$ tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
1) Africa							     7) Europe
2) Americas							     8) Indian Ocean
3) Antarctica							 9) Pacific Ocean
4) Asia								    10) coord - I want to use geographical coordinates.
5) Atlantic Ocean						11) TZ - I want to specify the timezone using the Posix TZ format.
6) Australia
#? 4
Please select a country whose clocks agree with yours.
 1) Afghanistan	  11) East Timor	    21) Kazakhstan	      31) Myanmar (Burma)	41) Sri Lanka
 2) Armenia		  12) Georgia		    22) Korea (North)	  32) Nepal			    42) Syria
 3) Azerbaijan	  13) Hong Kong		    23) Korea (South)	  33) Oman			    43) Taiwan
 4) Bahrain		  14) India		        24) Kuwait		      34) Pakistan		    44) Tajikistan
 5) Bangladesh	  15) Indonesia		    25) Kyrgyzstan	      35) Palestine		    45) Thailand
 6) Bhutan		  16) Iran		        26) Laos		      36) Philippines		46) Turkmenistan
 7) Brunei		  17) Iraq		        27) Lebanon		      37) Qatar			    47) United Arab Emirates
 8) Cambodia	  18) Israel		    28) Macau		      38) Russia		    48) Uzbekistan
 9) China		  19) Japan		        29) Malaysia	      39) Saudi Arabia		49) Vietnam
10) Cyprus		  20) Jordan		    30) Mongolia	      40) Singapore		    50) Yemen
#? 13

The following information has been given:

	Hong Kong

Therefore TZ='Asia/Hong_Kong' will be used.
Selected time is now:	Mon Sep 27 13:35:33 HKT 2021.
Universal Time is now:	Mon Sep 27 05:35:33 UTC 2021.
Is the above information OK?
1) Yes
2) No
#? 1

You can make this change permanent for yourself by appending the line
	TZ='Asia/Hong_Kong'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Hong_Kong
```

## 修改时区

### 为单独一个用户设置时区
根据提示键入 `TZ='Asia/Hong_Kong'; export TZ >> .xxxprofile`，根据自己的控制台类型选择配置文件
Example:
```
TZ='Asia/Hong_Kong'; export TZ >> .bashrc
sudo source .bashrc
```

### 修改系统时区

```bash
~$ ls /etc/localtime -al
lrwxrwxrwx 1 root root 27 Aug 24 08:45 /etc/localtime -> /usr/share/zoneinfo/Etc/UTC
~$ sudo rm /etc/localtime # 删除原有文件
~$ sudo ln /usr/share/zoneinfo/Asia/Shanghai /etc/localtime # 创建新的硬连接
``` 

## 修改时间

不能联网的设备手动修改时间

`date -s "20211001 10:00:00"`

允许联网设备可以使用使用 NTP 来与网络中的 NTP server 时间同步：

```
sudo apt install ntpdate
sudo ntpdate ip
```

常用 NTP Serve 地址
```
1. 常用服务器
NTP 授时快速域名服务: cn.ntp.org.cn
中国科学院国家授时中心: ntp.ntsc.ac.cn
清华大学(ipv4/ipv6): ntp.tuna.tsinghua.edu.cn
美国：time.nist.gov
微软公司授时主机(美国): time.windows.com 
开源NTP服务器: cn.pool.ntp.org

2. 腾讯云授时服务器
time1.cloud.tencent.com 
time2.cloud.tencent.com 
time3.cloud.tencent.com
time4.cloud.tencent.com
time5.cloud.tencent.com

3. 阿里云授时服务器
ntp.aliyun.com


4. 苹果授时服务器
time.apple.com
time.asia.apple.com
time1.apple.com
time2.apple.com
time3.apple.com
time4.apple.com
time5.apple.com
time6.apple.com
time7.apple.com

5. Google提供的授时服务器   
time1.google.com
time2.google.com
time3.google.com
time4.google.com
```



