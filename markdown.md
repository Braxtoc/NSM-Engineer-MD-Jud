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
