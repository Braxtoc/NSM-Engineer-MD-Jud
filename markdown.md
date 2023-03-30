# This is a heading
## heading
### heading

---
**Bold**

---

*italic*
___

***Bold and Italic***
___

~~strike-through~~

---
> this quote

---
- Bullet
 - Bullet
    - bullet
- Bullet
    - bullet

---
1. One
1. Two

---
1. online - [elastic](https://elastic.co)
1. local link - [Readme](/student/home/)

---
`systemctl start kibana`

```
suricata.yaml
```
---
| column1 | colum2 | cloumn3 | column4 |
| --- | --- | --- | --- |
| Data1 | Data2 | Data3 | Data4 |

---
<!--you cant see me -->

---

1. save file
1. git add .
1. git commit -m "message"
1. git push

# Day 1
---
 ## PfSense
 1. Insert the Bootable OS device into the PfSense
 1. Boot into the device
 1. and initiate the install of the OS
 1.`click accept`
 1. `default key map`
 1. `auto guided ... uefi`
 1. `option 1`
 1. `no vlan`
 1. `Wan interface, em0`
 1. `Lan interface, em1`
 1. `option 2`
 1. `2`
 1. `172.16.60.1`
 1. `24` bits
 1. `DHCP: yes`
 1. set range: `172.16.60.100`
 1. `172.16.60.254`
 1. web config : `yes`

 ## Change configurations in GUI
 1.  login `username: PfSense`
 `password: PfSense`
 1. set hostname
 1. override DNS
 1. uncheck stuff on WAN at bottom
 1. continue through to reload
 1. finish

 ## Firewall
 - select `firewall`
 - select `rules`
    - Action `Pass`
    - address family `IPV4`
    - Protocol `any`
    - Source `network`
        - 192.168.2.0/24

---
## Services
- DNS Fowarder
    - unceck `enable DNS fowarder`
- Diagnostic
  - Halt System `ok`     


---
# Day 2

- Edge router (facing workstations): 192.168.2.1
- Edge router (downstream to cisco): 10.0.0.2
- Cisco switch(port 24 to edge): 10.0.0.1
- Cisco switch(physical connect to cisco): 10.0.60.1
- PfSense: 172.16.60.1
- Sensor: 172.16.60.100

---
## Sensor install

1. install OS onto device w/ USB
  - ``uefi usb 0``
  - ``install centos 7``
  - ``English``
1. Network and hostname: eno1 (plugged into pfsense)
  - Configure ipv4 to static address of sensor
  - Configure ipv6 to ignore
  - Set hostname: sg60.local.Lan
1. Date and time: ETC & Coordinated universal time
  - Network time
1. Disable KDUMP
1. Installation Destination
  - clear thine partitions
  - configure partitions
    - /home : 50g
    - /boot : default
    - /boot/efi : default
    - / : **blank for after volume group creation**
    - swap : 4g
    - /var/log : 50g
    - /var : 50g
    - /tmp : 5g
    - /var/log/audit : 50g
    - /var/tmp : 10g
    - /data/stenographer : 500g
    - /data/suricata : 25g
    - /data/kafka : 100g
    - /data/elasticsearch : **blank for after volume group creation**
    - /data/fsf : 10g
    - /data/zeek : 25g
  - Configure volume groups
    - 500 G , vg_os : set all partitions that aren't /Data
    - 1 TB, vg_data : for /data partitions
  - begin Installation
  - Create a admin user : elastic123

1. within the network scripts make sure IPV6 is disabled for eno1
1. ``vi /etc/sysctl.conf``
  - add to end of file ``net.ipv6.conf.all.disable_ipv6=1`` & ``net.ipv6.conf.default.disable_ipv6=1``
1. ``/etc/hosts``
  - comment out the second line
1. ``/etc/sysconfig/network-scripts/ifcfg-eno1``
  - Change all IPV6 to "no"
  - BOOTPROTO=yes
1. ``systemctl restart network``
  - or ``sysctl -p``
---

## Sensor Configuration

1. Set up Repos
  - ``cd /etc/yum.repos.d``
  - `` sudo rm -rf CentOS-*``
  - Open browser and navigate to 192.168.2.20 to download local.repo
  - ``curl -LO http://192.168.2.20:8080/local.repo``
  - ``cat local.repo``
  - ``yum makecache``
  - ``yum list zeek`` test your repo
  - ``yum update``
  - ``systemctl reboot``
  - ``yum makecache --disablerepo="*" --enablerepo=local*``
  - ``yum update --disablerepo="*" --enablerepo=local*``
  - ``reboot``
1. SELinux
  - ``sestatus`` checks the state

---
# Day 3
## Sniffing Traffic
1. Interface setup: enp5s0
  - ``ethtool -k enp5s0`` shows if nic offloading is enabled
  - ``curl -LO http://192.168.2.20:8080/interface_prep.sh`` into the home directory
  - ``chmod +x interface_prep.sh`` turns off things
  - `` ./inteface_prep.sh enp5s0``
  - ``ethtool -k enp5s0``
  - ``cd /sbin``
  - ``curl -LO http://192.168.2.20:8080/ifup-local``
  - ``chmod +x ifup-local``
  - ``./ifup-local``
  - ``cd /etc/sysconfig/network-scripts``
  - ``vi ifup``
  - at the end of the file <shift g> enter ``if [ -x /sbin ifup-local ]; then
  /sbin/ifup-local ${DEVICE}
  fi``
  - ``vi ifcfg-enp5s0``
    - BOOTPROTO=none
    - IPV6_AUTOCONF=no
    - IPV6_DEFROUTE=no
    - IPV6_FAILURE_FATAL=no
    - DEFROUTE=none
    - ONBOOT=yes
    - NM_CONTROLLER=no
  - ``reboot``
  - ``ethtool -k enp5s0``

---
## Stenographer

1. ``cd /etc/yum.repos.d``
1. ``yum list stenographer``
1. ``yum install stenogapher``
1. ``cd /etc/stenographer``
1. ``vi config``
  - on line 3 modify "packets directory": ``"/data/stenographer/packets"``
  - "IndexDirectory": ``"/data/stenographer/index"``
  - "StenotypePath": ``"/bin/stenotype"``  Command to find ``which stenotype``
  - "Interface": ``"enp5s0"``
1. ``mkdir /data/stenographer/packets``
1. ``mkdir /data/stenographer/index``
1. ``cd /data/stenographer``
1. `` cd /data ``
1. ``chown -R stenographer:stenographer /data/stenographer``
1. ``stenokeys.sh stenographer stenographer``
1. ``systemctl start stenographer``
1. ``systemctl status stenographer``
1. ``systemctl enable stenographer``
1. ``stenoread 'host XXX.XXX.XXX.XXX' -nn 'src host xxx.xxx.xxx.xxx'``

---
## Suricata

1. ``yum install suricata``
1. ``cd /etc/suricata``
1. ``vi suricata.yaml``
  - line 56: ``default-log-dir: /data/suricata``
  - line 60: ``enabled: no``
  - line 76: ``enabled: no``
  - line 404: ``no``
  - line 557: ``enabled: no``
  - line 580: ``interface:enp5s0``
  - line 582: ``threads: auto`` uncomment
1. ``cd /etc/sysconfig``
1. ``vi suricata``
  - ``OPTIONS="--af-packet=enp5s0 --user suricata"``
1. ``suricata-update add-source local-emerging-threats http://192.168.2.20:8080/emerging.rules.tar``
1. ``suricata-update``
1. ``cd /data``
1. ``chown -R suricata: /data/suricata``
1. ``systemctl start suricata``
1. ``systemctl enable suricata``
1. ``systemctl status suricata``
1. test rules ``curl -LO 192.168.2.20:8080/all-class-files.zip``
1. ``cat eve.json | jq`` look at logs

---
# Day 4
## zeek
zeek -Cr [pcap file]

1. ``yum install zeek zeek-plugin-kafka zeek-plugin-af_packet``
1. ``vi /etc/zeek/networks.cfg``
1. ``vi /etc/zeek/zeekctl.cfg``
  - line 67: ``LogDir = /data/zeek``
  - line 76: ``lb_custom.InterfacePrefix=af_packet::``
1. ``vi /etc/zeek/node.cfg``
  - line 8-11 comment router
  - line 16-32 uncomment
  - line 20-23
    - add line under host=localhost ``pin_cpus=1``
  - line 32: ``interface=enp5s0``
  - after line 32 add ``lb_method=custom``
  - after line 33 add ``lb_procs=2``
  - after line 34 add ``pin_cpus=2,3``
  - after line 35 add ``env_vars=fanout_id=77``
1. ``mkdir /usr/share/zeek/site/scripts``
1. ``cd /usr/share/zeek/site/scripts``
1. ``curl -LO https://192.168.2.20:8080/zeek_scripts/afpacket.zeek``
1. ``curl -LO https://192.168.2.20:8080/zeek_scripts/extension.zeek``
1. `` vi /usr/share/zeek/site/local.zeek``
  - end of file `` @load ./scripts/afpacket.zeek``
  - end of file `` @load ./scripts/extension.zeek``
1. ``cd /data``
1. `` chown -R zeek: /data/zeek``
1. `` chown -R zeek: /etc/zeek``
1. `` chown -R zeek: /usr/share/zeek``
1. `` chown -R zeek: /usr/bin/zeek``
1. `` chown -R zeek: /usr/bin/capstats``
1. `` chown -R zeek: /var/spool/zeek ``
1. `` /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/zeek ``
1. `` /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/capstats ``
1. `` getcap /usr/bin/zeek``
1. `` getcap /usr/bin/capstats``
1. `` vi /etc/systemd/system/zeek.service``
  - [Unit]
  - Description=Zeek Network Analysis Engine
  - After=network.target
  -
  - [Service]
  - Type=forking
  - User=zeek
  - ExecStart=/usr/bin/zeekctl deploy
  - ExecStop=/usr/bin/zeekctl stop
  -
  - [Install]
  - WantedBy=multi-user.target
1. ``systemctl daemon-reload``
1. ``systemctl start zeek``

---
## Kafka
1. 
