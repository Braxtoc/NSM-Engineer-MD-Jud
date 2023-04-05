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
  - configure partitionsyum install stenogapher
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
  - at the end of the file <shift g> enter
    - if [ -x /sbin/ifup-local ]; then
    - ...../sbin/ifup-local ${DEVICE}
    - fi
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
1. ``yum install stenographer``
1. ``cd /etc/stenographer``
1. ``vi config``
  - on line cd /etc/sysconfig/network-scripts
3 modify "packets directory": ``"/data/stenographer/packets"``
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
1. ``curl -LO http://192.168.2.20:8080/zeek_scripts/afpacket.zeek``
1. ``curl -LO http://192.168.2.20:8080/zeek_scripts/extension.zeek``
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
1. ``systemctl enable zeek``

---
## Kafka/Zookeeper
1. ``yum install kafka zookeeper``
1. 'situation awareness' ``vi /etc/zookeeper/zoo.cfg``
1. ``systemctl start zookeeper``
1. ``systemctl enable zookeeper``
1. ``vi /etc/kafka/server.properties``
  - line 31 uncomment: ``listeners=PLAINTEXT://172.16.60.100:9092``
  - line 36 uncomment: ``advertised.listeners=PLAINTEXT://172.16.60.100:9092``
  - line 60: ``log.dirs=/data/kafka``
  - line 65: ``num.partitions=3``
  - line 107: ``log.retention.bytes=90000000000``
  - line 123: ``zookeeper.connect=127.0.0.1:2181``
1. ``chown -R kafka: /data/kafka``
1. ``firewall-cmd --add-port=9092/tcp --permanent``
1. ``firewall-cmd --add-port=2181/tcp --permanent``
1. ``firewall-cmd --reload``
1. ``systemctl start kafka``
1. ``systemctl enable kafka``
1. ``ss -lnt``
1. ``cd /usr/share/zeek/site/scripts``
1. ``curl -LO http://192.168.2.20:8080/zeek_scripts/kafka.zeek``
1. `` vi /usr/share/zeek/site/scripts``
  - line 7: ``["metadata.broker.list"] = "172.16.60.100:9092");``
1. ``vi /usr/share/zeek/site/local.zeek``
  - end of file: ``@load ./scripts/kafka.zeek``
1. ``systemctl restart zeek``
1. ``/usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.60.100:9092 --list``
1. ``/usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.60.100:9092 --describe --topic zeek-raw``
1. ``/usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.60.100:9092 --topic zeek-raw``

---
## fsf
1. ``yum install fsf``
1. ``vi /opt/fsf/fsf-server/conf/config.py``
  - line 9: ``SCANNER_CONFIG = { 'LOG_PATH' : '/data/fsf/logs',``
  - line 10: ``'YARA_PATH' : '/var/lib/yara-rules/rules.yara',``
  - line 11: ``'PID_Path' : '/run/fsf/fsf.pid',``
  - line 12: ``'EXPORT_PATH' : '/data/fsf/archive',``
  - line 18: ``SERVER_CONFIG = [ 'IP_ADDRESS' : "172.16.60.100",``
1. ``vi /opt/fsf/fsf-client/conf/config.py``
  - line 9: change to sensor ip
1. ``cd /data``
1. ``mkdir -p /data/fsf/{logs,archive}``
1. ``chown -R fsf: /data/fsf``
1. ``chmod -R 0755 /data/fsf``
1. ``vi /opt/fsf/fsf-client/conf/config.py``
  - just for SA
1. ``vi /etc/systemd/system/fsf.service``
  - [Unit]
  - Description=File Scanning Framework (FSF-Server) Services
  - After=network.target
  -
  - [Service]
  - Type=forking
  - User=fsf
  - Group=fsf
  - WorkingDirectory=/
  - PIDFile=/run/fsf/fsf.pid
  - PermissionsStartOnly=true
  - ExecStartPre=/bin/mkdir -p /run/fsf
  - ExecStartPre=/bin/chown -R fsf:fsf /run/fsf
  - ExecStart=/opt/fsf/fsf-server/main.py start
  - ExecStop=/opt/fsf/fsf-server/main.py stop
  - ExecReload=/opt/fsf/fsf-server/main.py restart
  -
  - [Install]
  - WantedBy=multi-user.target
1. ``systemctl start fsf``
1. ``systemctl status fsf``
1. ``/opt/fsf/fsf-client/fsf_client.py --full <filename>``
1. ``mkdir /data/zeek/extracted_files``
1. `` chown -R zeek: /data/zeek/extracted_files``
1. `` chmod -R 0755 /data/zeek/extracted_files``
1. `` vi /usr/share/zeek/site/scripts/extract_files.zeek``
  - @load /usr/share/zeek/policy/frameworks/files/extract-all-files.zeek
  - redef FileExtract::prefix = "/data/zeek/extracted_files/";
  - redef FileExtract::default_limit = 1048576000;
1. ``vi /usr/share/zeek/site/local.zeek``
  - end of file: ``@load ./scripts/extract_files.zeek``
1. ``systemctl restart zeek``
1. `` cd /usr/share/zeek/site/scripts``
1. ``curl -L -O http://192.168.2.20:8080/zeek_scripts/fsf.zeek``
1. ``vi /usr/share/zeek/site/fsf.zeek``
  - line 10 : changed to extracted_files
1. `` vi /usr/share/zeek/site/local.zeek``
  - end of file: ``@load ./scripts/fsf.zeek``
1. ``systemctl restart zeek``

---
## Elasticscearch

1. `` yum install elasticsearch-7.8.1``
1. ``chown elasticsearch: /data/elasticsearch``
1.  `` vi /etc/elasticsearch/elasticsearch.yml``
  - line 17 (uncomment): ``cluster.name: sg-60-cluster``
  - line 23 (uncomment): ``node.name: sg-60-node``
  - line 33: `` path.data: /data/elasticsearch ``
  - line 43: uncomment
  - line 55: ``network.host: _eno1_``
  - line 59: uncomment
  - line 68 (uncomment): ``discovery.seed_hosts: [172.16.60.100]``
  - line 72 (uncomment): ``cluster.initial_master_nodes: [172.16.60.100]``
  - > OR
  - line 68 add: ``discovery.type: single-node``
  - line 71 & line 75: comment out
1. `` mkdir -p /usr/lib/systemd/system/elasticsearch.service.d``
1. `` vi /usr/lib/systemd/system/elasticsearch.service.d/override.conf``
  - [Service]
  - LimitMEMLOCK=infinity
1. ``chmod 755 /usr/lib/systemd/system/elasticsearch.service.d``
1. `` chmod 644 /usr/lib/systemd/system/elasticsearch.service.d/override.conf``
1. ``systemctl daemon-reload``
1. ``systemctl start elasticsearch``
1. ``systemctl enable elasticsearch``
1. `` vi /etc/elasticsearch/jvm.options``
 - line 22: ``-Xms4g``
 - line 23: ``-Xms4g``
1. ``systemctl restart elasticsearch``
1. ``ss -lnt`` (check for binding)
1. ``firewall-cmd --add-port={9200,9300}/tcp --permanent``
1. ``firewall-cmd --reload``
1. ``curl 172.16.60.100:9200``
1. ``curl 172.16.60.100:9200/_cat/nodes``

---
## Kibana
1. `` yum install kibana-7.8.1``
1. `` vi /etc/kibana/kibana.yml``
  - line 7 (uncomment): ``server.host: "172.16.60.100"``
  - line 28: uncomment change to sensor ip
1. ``systemctl start kibana``
1. ``systemctl enable kibana``
1. ``firewall-cmd --add-port=5601/tcp --permanent``
1. ``firewall-cmd --reload``
1. in browser ``http://172.16.60.100:5601``
1. ``cd /admin/home``
1. ``curl -LO http://192.168.2.20:8080/ecskibana.tar.gz``
1. `` tar -zxvf ecskibana.tar.gz``
1. `` cd ecskibana``
1. ``./import-index-templates.sh``
  - change from local host to sensor ip
1. Dev tools in Kibana
  - ``GET _cat/templates``
  - ``GET _cat/templates?v&s=name``
  - ``GET _cat/templates/ecs*``
  - ``GET _template/default``
---
## Filebeats
1. ``yum install filebeat-7.8.1``
1. ``cd /etc/filbeat``
1. ``mv /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.bk``
1. ``curl -LO 192.168.2.20:8080/filebeat.yml``
1. ``vi /etc/filebeat/filebeat.yml``
  - after line 10:
    - "- type: log"
    - enabled: true
    - paths:
    -   - /data/fsf/logs/rockout.log
    - json.keys_under_root: true
    - fields:
    -    kafka_topic: fsf-raw
    - fields_under_root: true
  - line 22: ``hosts: ["172.16.60.100:9092"]``
1. ``systemctl start filebeat``
1. ``/usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.60.100:9092 --topic fsf-raw --from-beginning``
1. ``systemctl enable filebeat``

---
## Logstash
1. ``yum install logstash-7.8.1``
1. ``cd /etc/logstash``    
1. ``curl -LO 192.168.2.20:8080/logstash.tar``
1. ``tar -xvf logstash.tar``
1. ``cd conf.d``
1. ``vi logstash-100-input-kafka-zeek.conf``
    - line 8: ``bootstrap_servers => "172.16.60.100:9092"``
1. ``vi logstash-100-input-kafka-suricata.conf``
    - line 8: ``bootstrap_servers => "172.16.60.100:9092"``
1. ``vi logstash-100-input-kafka-fsf.conf``
    - line 8: ``bootstrap_servers => "172.16.60.100:9092"``
1. ``vi logstash-9999-output-elasticsearch.conf``
    - replace all 127.0.0.1 with senor ip: 172.16.60.100:9092
      - in vi command mode ``%s/127.0.0.1/172.16.60.100``
    - Line 2: comment out
1. ``systemctl start logstash``
1. ``systemctl enable logstash``
1. ``cat /var/log/logstash/logstash-plain.log | tail -10``
1. ``vi /etc/kibana/kibana.yaml``
  - line 28: ``elasticsearch.hosts: ["http://172.16.60.100:9200"]``
1. ``systemctl restart {logstash,kibana}``
1. on kibana go to index patterns
    - ``create index pattern``
       - name ``ecs-suricata-*``
       - time filter: ``@timestamp``
       - ``create``
    - ``create index pattern``
       - name ``ecs-zeek-*``
       - time filter: ``@timestamp``
       - ``create``
    - ``create index pattern``
       - name ``fsf-*``
       - time filter: ``@timestamp``
       - ``create``
---
## Logstash pipes
1. ``systemctl stop logstash``
1. ``cd /etc/logstash``
1. ``vi pipeline.yml``
1. ``mkdir my-pipeline``
1. ``chown logstash: my-pipeline/``
1. ``cd /my-pipeline``
1. ``vi input.conf``
  - input {
  - generator {
  - message => 'paste http example'
  - count => 1
  - }  
  - }  
1. ``vi filter.conf``
  - filter {
  - mutate {
  - copy => { "[message]" => "[event][original]"}
  - }
  - json {
  - source => "[message]"
  - target => "[zeek][http]"
  - remove_field => "[message]"
  -   }
  - mutate {
  - copy => { "[@timestamp]" => "[event][created]"}
  - }
  - date {
  - match => ["[zeek][http][ts]", "UNIX"  ]
  - target => "[@timestamp]"
  - remove_field => "[zeek][http][ts]"
  - }
  - mutate {
  - rename => {
  -  "[zeek][http][id.orig_h]" => "[source][address]"
  -  "[zeek][http][id.orig_p]" => "[source][port]"
  -  "[zeek][http][id.resp_h]" => "[destination][address]"
  -  "[zeek][http][id.rest_p]" => "[destination][port]"
  - }
  - }
  - mutate {
  - copy => {
  - "[source][address]"      => "[source][ip]"
  - "[destination][address]" => "[destination][ip]"
  - "[source][address]"      => "[client][ip]"
  -  
  - }
  - copy => {
  - "[source][destination]" => "[client][address]"
  - "[destination][address]" => "[server][address]"
  - "[source][port]" => "[client][port]"
  - "[server][port]" => "[server][port]"
  - }
  - copy => {
  - "[client][address]" => "[client][ip]"
  - "[server][address]" => "[server][ip]"
  - }  
  - }
  - }
1. ``vi output.conf``
  - output {
  - stdout {}
  - }
1. ``vi pipelines.yml``
  - comment out content
  - add to end of file
  -  "- pipeline.id:test" no quotes, do one space in
  -  ``path.config: "/etc/logstash/my-pipeline/*.conf"``
1. ``/usr/share/logstash/bin/logstash --path.settings /etc/logstash/``
