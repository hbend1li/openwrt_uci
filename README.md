# OpenWrt UCI

The abbreviation of **UCI** stands for **U**nified **C**onfiguration **I**nterface and is intended to centralize the configuration of OpenWrt.

## Configuration Files

### File Syntax
The UCI configuration files usually consist of one or more config statements, so called sections with one or more option statements defining the actual values.

Below is an example of a simple configuration file:

```
package 'example'

config 'example' 'test'
        option   'string'      'some value'
        option   'boolean'     '1'
        list     'collection'  'first item'
        list     'collection'  'second item'
```

## Command Line Utility
For adjusting settings, one normally changes the UCI config files directly. However, for scripting purposes, all of UCI configuration can also be read and changed using the uci command line utility. For developers requiring automatic parsing of the UCI configuration, it is therefore redundant, unwise, and inefficient to use awk and grep to parse OpenWrt's config files. The uci utility offers all functionality with respect to modifying and parsing UCI.

Below is the usage, as well as some useful examples of how to use this powerful utility.

### Usage
```
Usage: uci [<options>] <command> [<arguments>]

Commands:
	batch
	export     [<config>]
	import     [<config>]
	changes    [<config>]
	commit     [<config>]
	add        <config> <section-type>
	add_list   <config>.<section>.<option>=<string>
	del_list   <config>.<section>.<option>=<string>
	show       [<config>[.<section>[.<option>]]]
	get        <config>.<section>[.<option>]
	set        <config>.<section>[.<option>]=<value>
	delete     <config>[.<section[.<option>]]
	rename     <config>.<section>[.<option>]=<name>
	revert     <config>[.<section>[.<option>]]
	reorder    <config>.<section>=<position>

Options:
	-c <path>  set the search path for config files (default: /etc/config)
	-d <str>   set the delimiter for list values in uci show
	-f <file>  use <file> as input instead of stdin
	-m         when importing, merge data into an existing package
	-n         name unnamed sections on export (default)
	-N         don't name unnamed sections
	-p <path>  add a search path for config change files
	-P <path>  add a search path for config change files and use as default
	-q         quiet mode (don't print error messages)
	-s         force strict mode (stop on parser errors, default)
	-S         disable strict mode
	-X         do not use extended syntax on 'show'
```

### Setting a value
If we want to change the listening port of the uHTTPd Web Server from 80 to 8080 we change the configuration in /etc/config/uhttpd :
```
# uci set uhttpd.main.listen_http=8080
# uci commit uhttpd
# /etc/init.d/uhttpd restart
```
Done, now the configuration file is updated and uHTTPd listens on port 8080.

### Export an entire configuration
```
# uci export httpd
package 'httpd'

config 'httpd'
	option 'port' '80'
	option 'home' '/www'
```
### Show the configuration "tree" for a given config
```
# uci show httpd
httpd.@httpd[0]=httpd
httpd.@httpd[0].port=80
httpd.@httpd[0].home=/www
```

## Wi-Fi /etc/config/wireless

[Refer to officiel web site OpenWrt](https://openwrt.org/docs/guide-user/network/wifi/basic)

The wireless radio UCI configuration is located in **/etc/config/wireless**.

If the device has ethernet ports, the wireless is turned **OFF** by default. You can turn it on in **/etc/config/wireless** by changing **option disabled '1'** to **option disabled '0'** (commenting out the line or removing it is sufficient).

### Wi-Fi interfaces
A minimal example for a wifi-iface declaration is given below.
```
config 'wifi-iface'
	option 'device'     'radio0'
	option 'network'    'lan'
	option 'mode'       'ap'
	option 'ssid'       'MyWifiAP'
	option 'encryption' 'psk2'
	option 'key'        'secret passphrase'
```


### Start/Stop wireless
Wireless interfaces are brought up and down with the **wifi** command. To (re)start the wireless after a configuration change, use **wifi**, to disable the wireless, run **wifi down**. In case your platform carries multiple wireless devices it is possible to start or run down each of them individually by making the **wifi** command be followed by the device name as a second parameter.  
Note: The **wifi** command has an optional first parameter that defaults to **up** , i.e. start the device. To make the second parameter indeed a second parameter it is mandatory to give a first parameter which can be anything except **down**. E.g. to start the interface **wlan2** issue: **wifi up wlan2**; to stop that interface: **wifi down wlan2**. If the platform has also e.g. wlan0 and wlan1 these will not be touched by stopping or starting wlan2 selectively.

### Regenerate configuration
To rebuild the configuration file, e.g. after installing a new wireless driver, remove the existing wireless configuration (if any) and use the wifi config command:
```
rm -f /etc/config/wireless
wifi config
```

## DHCP Server
This example will provide instructions on how configure RUT routers' DHCP Server using only UCI commands. For the sake of the example lets say that you want to change the dhcp range to 192.168.1.2 - 192.168.1.200 and the lease time to 30 minutes

To achieve such a task, the first relevant piece of required information is the config name, dhcp, where all the necessary configuration settings are stored. Another important thing to know is that when changing the lease time, three options are relevant - the time (option time), the unit of time measurement (option letter) and lease time(option leasetime), which is basically time + letter, e.g., 12 hour lease time is 12h. Other options in question are start address (option start) and address limit (option limit). Lets start:

Setting start address and limit:
```
# uci set dhcp.lan.start=2
# uci set dhcp.lan.limit=199
```
Setting lease time
```
# uci set dhcp.lan.letter=m
# uci set dhcp.lan.time=30
# uci set dhcp.lan.leasetime=30m
```
Final steps:
```
# uci commit dhcp
# luci-reload
```
The first step sets the start address to 2 and the limit of addresses to 199. The value of the start option is associated with the last section of an IP address (if start value is 2 then the starting IP address is 192.168.1.2(provided that the router's LAN IP is in the 192.168.1.0/24 network)), the value of the limit option denotes how many IP addresses can be leased out starting from and including the the start address. Then the second step is used to set the lease time. The letter option specifies the unit of time measurement (either m for minutes or h for hours). The time option specifies number of minutes (or hours in other cases) and the leasetime option is just the representation (nonetheless, it's still mandatory) of the previous two values, i.e., 30m - thirty minutes.



## Firewall
### Add a firewall rule
This is a good example of both adding a firewall rule to forward the TCP SSH port, and of the negative (-1) syntax used with uci.
```
# uci add firewall rule
# uci set firewall.@rule[-1].src=wan
# uci set firewall.@rule[-1].target=ACCEPT
# uci set firewall.@rule[-1].proto=tcp
# uci set firewall.@rule[-1].dest_port=22
# uci commit firewall
# /etc/init.d/firewall restart
```

### Site Blocking

[Refer to fw3 IPv4 configuration examples](https://openwrt.org/docs/guide-user/firewall/fw3_configurations/fw3_config_examples)

This example will provide instructions on how to enable RUT routers' Site Blocking feature and how to add hostnames to the Blacklist or Whitelist using only UCI commands. For the sake of our example lets say that you want to create a Blacklist that excludes access to all sites contained within the list. The sites in question are **www.facebook.com**, **www.youtube.com** and **9gag.com**.

To achieve such a task, the first relevant piece of required information is the config name, hostblock, where all the necessary configuration settings are stored. The next important thing to know is that each different website must be stored in a separate section of the type block. So we'll need to create a new section and enable each added element. Lets start:

First element:
```
# uci add hostblock block
# uci set hostblock.@block[0].host=www.facebook.com
# uci set hostblock.@block[0].enabled=1
```
Second element:
```
# uci add hostblock block
# uci set hostblock.@block[1].host=www.youtube.com
# uci set hostblock.@block[1].enabled=1
```
Third element:
```
# uci add hostblock block
# uci set hostblock.@block[2].host=9gag.com
# uci set hostblock.@block[2].enabled=1
```
Enabling Site Blocking:
```
# uci set hostblock.config.enabled=1
```
Final steps:
```
# uci commit hostblock
# luci-reload
```
The first-third steps add hostnames of the websites to be blocked, which are saved under the option host. Each of the first three elements also need to be enabled, therefore, the option enabled is set to 1 next to each host. The fourth step is for enabling the Site Blocking service (by setting the option enabled in section config to 1).

Another approach would be to block the **YouTube** IP range, based on a  custom firewall rules can be generated:  
**/etc/config/firewall**  
```
config rule
	option name		Block-YouTube-187.189.89.77/16
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		187.189.89.77/16
	option target		REJECT

config rule
	option name		Block-YouTube-189.203.0.0/16
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		189.203.0.0/16
	option target		REJECT

config rule
	option name		Block-YouTube-64.18.0.0/20
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		64.18.0.0/20
	option target		REJECT

config rule
	option name		Block-YouTube-64.233.160.0/19
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		64.233.160.0/19
	option target		REJECT

config rule
	option name		Block-YouTube-66.102.0.0/20
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		66.102.0.0/20
	option target		REJECT

config rule
	option name		Block-YouTube-66.249.80.0/20
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		66.249.80.0/20
	option target		REJECT

config rule
	option name		Block-YouTube-72.14.192.0/18
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		72.14.192.0/18
	option target		REJECT

config rule
	option name		Block-YouTube-74.125.0.0/16
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		74.125.0.0/16
	option target		REJECT

config rule
	option name		Block-YouTube-173.194.0.0/16
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		173.194.0.0/16
	option target		REJECT

config rule
	option name		Block-YouTube-207.126.144.0/20
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		207.126.144.0/20
	option target		REJECT

config rule
	option name		Block-YouTube-209.85.128.0/17
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		209.85.128.0/17
	option target		REJECT

config rule
	option name		Block-YouTube-216.58.208.0/20
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		216.58.208.0/20
	option target		REJECT

config rule
	option name		Block-YouTube-216.239.32.0/19
	option src		lan
	option family		ipv4
	option proto		all
	option dest		wan
	option dest_ip		216.239.32.0/19
	option target		REJECT
```
And so on for all IP ranges, be sure to restart the firewall service to apply the changes.
```shell
# /etc/init.d/firewall restart
```

## Setup mirror of Openwrt package repository
Suppose we need a local package repository, 19.07 branch of newif

```shell
#!/bin/bash
BASEURL="rsync://downloads.openwrt.org/downloads/releases"
VERSION="19.07.0-rc2"
TARGET="ar71xx"
ARCH="mipsel_24kc"

mkdir -p ~/mirror/openwrt/$VERSION/ && cd ~/mirror/openwrt/$VERSION/
rsync -avz --delete --progress $BASEURL/$VERSION/targets/$TARGET/generic/packages/ ./core
rsync -avz --delete --progress $BASEURL/$VERSION/packages/$ARCH/base ./
rsync -avz --delete --progress $BASEURL/$VERSION/packages/$ARCH/luci ./
rsync -avz --delete --progress $BASEURL/$VERSION/packages/$ARCH/packages ./
rsync -avz --delete --progress $BASEURL/$VERSION/packages/$ARCH/routing ./
rsync -avz --delete --progress $BASEURL/$VERSION/packages/$ARCH/telephony ./
```

Install web server ( darkhttpd )  
```shell
git clone https://unix4lyfe.org/git/darkhttpd ~/mirror/darkhttpd
cd ~/mirror/darkhttpd
make
~/mirror/darkhttpd/darkhttpd ~/mirror/openwrt
```

Modify OPKG-Configuration  **/etc/opkg/customfeeds.conf**  
replace **IP_ADDRESS:PORT** by your web server parameter.
```shell
echo "src/gz openwrt_core http://IP_ADDRESS:PORT/19.07.0-rc2/core">/etc/opkg/customfeeds.conf
echo "src/gz openwrt_base http://IP_ADDRESS:PORT/19.07.0-rc2/base">>/etc/opkg/customfeeds.conf
echo "src/gz openwrt_luci http://IP_ADDRESS:PORT/19.07.0-rc2/luci">>/etc/opkg/customfeeds.conf
echo "src/gz openwrt_packages http://IP_ADDRESS:PORT/19.07.0-rc2/packages">>/etc/opkg/customfeeds.conf
echo "src/gz openwrt_routing http://IP_ADDRESS:PORT/19.07.0-rc2/routing">>/etc/opkg/customfeeds.conf
echo "src/gz openwrt_telephony http://IP_ADDRESS:PORT/19.07.0-rc2/telephony">>/etc/opkg/customfeeds.conf

```
