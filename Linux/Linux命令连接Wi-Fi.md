# Connect to Wi-Fi network through Ubuntu terminal

## Connect Wi-Fi network using ***WEP*** as encryption.
* Open the terminal.

* sudo apt install net-tools

* Type `sudo ifconfig wlan0 up` and press `Enter`. You will not see any output in the terminal, as this command just turns your wireless card on. Most wireless cards are designated `wlan0`. If yours has a different designation, use that instead.

* Type `sudo iwconfig wlan0 essid name key password` and press `Enter`. Replace `name` with the actual network name, and replace `password` with the actual security key for the network. If your wireless network does not require a security key, do not enter key `password`.

* The password in the command `iwconfig wlan0 essid name key password` should be in ***hexadecimal***. If you want to type the ASCII password, you would use `iwconfig wlan0 essid name key s:password`.

* The command `iwconfig wlan0 essid name key password` only works with access points that use ***WEP*** as encryption. 

* Type `dhclient wlan0` and press `Enter` to obtain an IP address and connect to the Wi-Fi network.

## Connect Wi-Fi Network using ***WPA/WPA2*** as encryption.

* You need the `wpasupplicant` package which provides the `wpa_supplicant` command, install if necessary through `sudo apt-get install wpasupplicant`

* You put your SSID and password into `/etc/wpa_supplicant.conf` (requires sudo).

Example:
```ini
network={
    ssid="ssid_name"
    psk="password"
}
```

* Assuming your interface is wlan0 you can connect with:

```
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf -D wext
sudo dhclient wlan0
```

"wext" is a driver and that will be specific for each card; refer to `wpa_supplicant -h`. Examples:
```ini
drivers:
  nl80211 = Linux nl80211/cfg80211
  wext = Linux wireless extensions (generic)
  wired = Wired Ethernet driver
  macsec_linux = MACsec Ethernet driver for Linux
  none = no driver (RADIUS server/WPS ER)
```


* Or use `sudo netplan apply` to apply change.