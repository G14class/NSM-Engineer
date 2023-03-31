# NSM-Engineer
Notes
- Git account creation
- install vscode/atom
- github> code copy and run on terminal( sudo git clone "link")
- go to the directory (cd NSM-Engineer, ls)
- do 'Sudo Git pull' to update the repos.
-

PFSENSE.
em0- wan - LAN1
em1 -lan - LAN2

em1/LAN IP: 172.16.20.1

ip range: 172.16.20.100 - 172.16.20.254

http://172.16.20.1 - Pfsense



pfsense>services>Dns forwarder> uncheck DNS forwarder


edge router (192.168.1.1)> repo server(192.16.2.20) >
Troubleshooting steps

1.192.168.2.1 > Edge router
2.10.0.0.2 > downstream to cisco switch
3.10.0.0.1 > cisco SW port 24 to edge router
4.10.0.20.1 > physical port on switch.
5.10.16.20.1 > Pfsense
6.172.16.20.100 > sensor




intel NUC
en01-management
enp5
connect LAN2(pfsense) to intelnuc(management port)

configure en01 as (172.16.20.100 - sensor IP), gateway: 172.16.20.1

under /root > create


kafka - messaging queue
zeek - network metadata
suricata - ids
fsf- part of file extraction, uses yara rules
elasticsearch - database
Stenographer-pcap
___________________________________
sensor configurations, SSH in to sensor
(ssh admin@172.16.20.100) username: admin, password: pfsense

/etc/sysconfig/network-scripts/ifcfg-eno1 - disable ipv6(no)

/etc/sysctl.conf  (: add the below at the end of conf file and save)
(net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1)

sudo vi /etc/hosts
( diable ::1 -it's for ipv6 by putting # infront of it)

sudo sysctl -p

___________________________________________


cd /etc/yum.repos.d

ls -l

sudo rm -r Centos-*(remove all repos to create a local repos for our kit)

to create a local repo: sudo curl -LO http://192.168.2.20:8080/local.repo

sudo yum makecache
sudo yum update
sudo systemctl reboot

after reboot the centos repo might comeback, use the below command to disable it.

sudo yum makecache repo="--disable*" --enablerepo=local*
sudo yum update --disablerepo=* --enablerepo=local* (updates)


cd /etc/yum.repos.d/

cat local.repo

name=local-base
baseurl=http://192.168.2.20:8008/local-base/
enabled=1
gpgcheck=0

[local-elastic]
name=local-elastic
baseurl=http://192.168.2.20:8008/local-elastic/
enabled=1
gpgcheck=0

[local-epel]
name=local-epel
baseurl=http://192.168.2.20:8008/local-epel/
enabled=1
gpgcheck=0

[local-extras]
name=local-extras
baseurl=http://192.168.2.20:8008/local-extras/
enabled=1
gpgcheck=0

[local-rocknsm-2.5]
name=local-rocknsm-2.5
baseurl=http://192.168.2.20:8008/local-rocknsm-2.5/
enabled=1
gpgcheck=0

[local-updates]
name=local-updates
baseurl=http://192.168.2.20:8008/local-updates/
enabled=1
gpgcheck=0

_-_______________________________________________________________-

Wednesday Topics
SELinux - uses least priviledge to access, uses Mandatory access control(MAC) vs Discretionary access control(DAC)
- MAC : adds labels to objects
- don't disable SELinux, we will install each program with its own permission and not the root level permission.
- status of SELinux - sestatus
_______________________________________________________________
SNIFFING Traffic 
-
NIC offloading - disable this so zeek can get the raw data.

sudo ethtool -k enp5s0 - to modify setting of nic,

 cd (change directory to home to curl the below file)
curl -LO http://192.168.2.20:8080/interface_prep.sh

sudo chmod +X interface_prep.sh (changing permission to execute)

sudo ./interface_prep.sh enp5s0


sudo curl -LO http://192.168.2.20:8080/ifup-local (use sudo or curl won't work)

cd /etc/sysconfig/network-scripts

sudo vi ifup

        add this at the end(calls if up during startup for every interface)

if [ -x /sbin/ifup-local ]; then
        /sbin/ifup-local ${DEVICE}
fi


sudo vi ifcfg-enp5s0

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=no
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp5s0
UUID=4cb042f9-b6d3-4c20-9a61-7ce07f4062b5
DEVICE=enp5s0
ONBOOT=yes
NM_CONTROLLED=no

sudo systemctl reboot (to make the changes)

configuring ILINE TAP

sudo yum install tcpdump
sudo yum list tcpdump (to view the tcpdum file)

sudo tcpdump -i enp5s0
sudo tcpdump -nn -i enp5s0 -c4

sudo tcpdump -nn -i enp5s0 '!port 22'
sudo tcpdump -nn -c5 -i enp5s0 '!port 22'


---------------------------------------------



______________________________________________-
Google Stenographer
-fast write and slow read

Stenographer

stenotype>  Disk  < Stenoread

as analyst we will interact with stenoread.


ssh admin@172.16.20.100
sudo yum install stenographer

cd /etc/stenographer (the /etc folder is where we will install all programs)

sudo vi config

  "Threads": [
    { "PacketsDirectory": "/data/stenographer/packets"
    , "IndexDirectory": "/data/stenographer/index"
    , "MaxDirectoryFiles": 30000
    , "DiskFreePercentage": 10
    }
  ]
  , "StenotypePath": "/bin/stenotype"
  , "Interface": "enp5s0"
  , "Port": 1234
  , "Host": "127.0.0.1"
  , "Flags": []
  , "CertPath": "/etc/stenographer/certs"


sudo which stenotype ( to see where the location )
sudo mkdir /data/stenographer/packets
sudo mkdir /data/stenographer/index

cd /data/stenographer ( and do ls, to view the created folders)

cd ..
in the Data directory
drwxr-xr-x. 2 root root  6 Mar 28 14:32 elasticsearch
drwxr-xr-x. 2 root root  6 Mar 28 14:32 fsf
drwxr-xr-x. 2 root root  6 Mar 28 14:32 kafka
drwxr-xr-x. 4 root root 34 Mar 29 12:33 stenographer
drwxr-xr-x. 2 root root  6 Mar 28 14:32 suricata
drwxr-xr-x. 2 root root  6 Mar 28 14:32 zeek

sudo chown -R stenographer:stenographer /data/stenographer (changes the root permissions to stenographer user only)

drwxr-xr-x. 2 root         root          6 Mar 28 14:32 elasticsearch
drwxr-xr-x. 2 root         root          6 Mar 28 14:32 fsf
drwxr-xr-x. 2 root         root          6 Mar 28 14:32 kafka
drwxr-xr-x. 4 stenographer stenographer 34 Mar 29 12:33 stenographer
drwxr-xr-x. 2 root         root          6 Mar 28 14:32 suricata
drwxr-xr-x. 2 root         root          6 Mar 28 14:32 zeek


sudo stenokeys.sh stenographer stenographer

Generating CA state
Generating key/cert for 'client'
Generating key/cert for 'server'

sudo systemctl start stenographer
sudo systemctl enable stenographer(^enable^status)
sudo journalctl -xeu stenographer( to find any error logs

ping 192.168.2.20
       to query stenoread
sudo stenoread 'host 192.168.2.20' -nn (we specified not to see the 
sudo stenoread 'host 192.168.2.20' -nn 'src 192.168.2.20'

cd /data/stenographer
cd packets
ll
watch ls -al


--------------------------------------------------------------

sudo yum install suricata

sudo cd /etc/suricata
ll
sudo vi suricata.yaml
:set nu (to insert number)

line 56  : data
line 60  : no
line 70  : no
line 404 : no
line 557 : no
line 580 : enp5s0
line 582 : threads : auto (take out the #)


cd /etc/sysconfig
sudo vi suricata

OPTIONS="--af-packet-enp5s0 --user suricata "


sudo suricata-update add-source local-emerging-threats http://192.168.2.20:8080/emerging.rules.tar

suricata-update

sudo suricata-update list-sources


cd /data

sudo chown -R suricata: /data/suricata

drwxr-xr-x. 2 root         root          6 Mar 28 14:32 elasticsearch
drwxr-xr-x. 2 root         root          6 Mar 28 14:32 fsf
drwxr-xr-x. 2 root         root          6 Mar 28 14:32 kafka
drwxr-xr-x. 4 stenographer stenographer 34 Mar 29 12:33 stenographer
drwxr-xr-x. 2 suricata     suricata      6 Mar 28 14:32 suricata
drwxr-xr-x. 2 root         root          6 Mar 28 14:32 zeek

systemctl start suricata
^enable^start
systemctl status suricata
  -active running and enabled

curl -LO 192.168.2.20:8080/all-class-files.zip

cd /data/suricata
ll
cat eve.json | jq (to view the eve json file in a readable format.line 83 in the yaml to turn on/off)

tail -f eve.json | jq


_______________________________________________________
Zeek

zeek configuraton
/usr/bin - binaries
/etc/zeek - configurations
/var/log - data
/usr/share/zeek - scripts


/etc/zeek/
 node.c
____________________________________________________________-
sudo yum install zeek zeek-plugin-kafka zeek-plugin-af_packet

sudo vi /etc/zeek/networks.cfg (nothing to change here)

# List of local networks in CIDR notation, optionally followed by a
# descriptive tag.
# For example, "10.0.0.0/8" or "fe80::/64" are valid prefixes.

10.0.0.0/8          Private IP space
172.16.0.0/12       Private IP space
192.168.0.0/16      Private IP space

sudo vi /etc/zeek/zeekctl.cfg

line 30 LogRotationInterval = 3600
line 67 LogDir = /var/log/zeek    to    /data/zeek
line 76 lb_custom.InterfacePrefix=af_packet::

sudo vi /etc/zeek/node.cfg  (update line 9 -37)
line 9  #type=standalone
     10 #host=localhost
     11 #interface=eth0
line 16 [logger]
     17 type=logger
     18 host=localhost
line 19 
     20 [manager]
     21 type=manager
     22 host=localhost
     23 pin_cpus=1
     24 
     25 [proxy-1]
     26 type=proxy
     27 host=localhost
     28 
     29 [worker-1]
     30 type=worker
     31 host=localhost
     32 interface=enp5s0
     33 lb_method=custom
     34 lb_procs=2
     35 pin_cpus=2,3
     36 env_vars=fanout_id=77
     37 #
     38 #[worker-2]
     39 #type=worker
line 40 #host=localhost
     41 #interface=eth0


sudo mkdir /usr/share/zeek/site/scripts
cd /usr/share/zeek/site/scripts/
pwd
curl -LO http://192.168.2.20:8080/zeek_scripts/afpacket.zeek
vi afpacket.zeek  (just to view, no changes)

vi /usr/share/zeek/site/scripts

curl -LO http://192.168.2.20:8080/zeek_scripts/extension.zeek

sudo vi /usr/share/zeek/site/local.zeek (:set nu, shift G)

line 104 @load ./scripts/af_packet.zeek
     105 @load ./scripts/extension.zeek

cd /data
                  chown -R zeek: /etc/zeek
                  chown -R zeek: /data/zeek
[root@SG20 data]# chown -R zeek: /usr/bin/zeek
[root@SG20 data]# chown -R zeek: /usr/bin/capstats
[root@SG20 data]# chown -R zeek: /var/spool/zeek
[root@SG20 data]# chown -R zeek: /usr/share/zeek



sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/zeek
sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/capstats

to verify persistence

sudo getcap /usr/bin/zeek

sudo getcap /usr/bin/capstats

sudo vi /etc/systemd/system/zeek.service (native command to start or stop service)

[Unit]
Description=Zeek Network Analysis Engine
After=network.target

[Service]
Type=forking
User=zeek
ExecStart=/usr/bin/zeekctl deploy
ExecStop=/usr/bin/zeekctl stop

[Install]
WantedBy=multi-user.target


[root@SG20 data]# sudo systemctl daemon-reload
[root@SG20 data]# sudo systemctl start zeek

Job for zeek.service failed because the control process exited with error code. See "systemctl status zeek.service" and "journalctl -xe" for details.

sudo journalctl -xeu zeek (since Zeek failed to start we will use the journalctl to investigate the error, the error shows 

sudo vi /etc/zeek/node.cfg
    
      line 8 #[zeek]

sudo vi /usr/share/zeek/site/local.zeek   (update location from af_packet.zeek to afpacket.zee)

@load ./scripts/afpacket.zeek
@load ./scripts/extension.zeek


sudo ls -l /data/zeek/current/
-rw-r--r--. 1 zeek zeek  1622 Mar 30 14:18 broker.log
-rw-r--r--. 1 zeek zeek   412 Mar 30 14:33 capture_loss.log
-rw-r--r--. 1 zeek zeek  2011 Mar 30 14:18 cluster.log
-rw-r--r--. 1 zeek zeek  1055 Mar 30 14:25 conn.log
-rw-r--r--. 1 zeek zeek   629 Mar 30 14:19 dhcp.log
-rw-r--r--. 1 zeek zeek   318 Mar 30 14:18 known_services.log
-rw-r--r--. 1 zeek zeek 37079 Mar 30 14:18 loaded_scripts.log
-rw-r--r--. 1 zeek zeek   230 Mar 30 14:18 packet_filter.log
-rw-r--r--. 1 zeek zeek  2505 Mar 30 14:33 stats.log
-rw-r--r--. 1 zeek zeek     0 Mar 30 14:18 stderr.log
-rw-r--r--. 1 zeek zeek   188 Mar 30 14:18 stdout.log


[root@SG20 data]# sudo systemctl enable zeek

Created symlink from /etc/systemd/system/multi-user.target.wants/zeek.service to /etc/systemd/system/zeek.service.
________________________________________________________________________________--

KAFKA

sudo yum install kafka zookeeper

sudo vi /etc/zookeeper/zoo.cfg

sudo systemctl start zookeeper

sudo systemctl status zookeeper

sudo vi /etc/kafka/server.properties
line  31 listeners=PLAINTEXT://172.16.20.100:9092
line  36 advertised.listeners=PLAINTEXT://172.16.20.100:9092
      60 log.dirs=/data/kafka
      65 num.partitions=3
      107 log.retention.bytes=90000000000
      123 zookeeper.connect=127.0.0.1:2181

sudo chown -R kafka: /data/kafka

sudo firewall-cmd --add-port=9092/tcp --permanent

sudo firewall-cmd --add-port=2181/tcp --permanent

sudo firewall-cmd --reload


[root@SG20 ~]# sudo systemctl start kafka
[root@SG20 ~]# sudo systemctl status kafka
[root@SG20 ~]# cd /data/kafka


sudo firewall-cmd --list-all (lists all ports assigned)

[root@SG20 kafka]# sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eno1 enp5s0
  sources: 
  services: dhcpv6-client ssh
  ports: 9092/tcp 2181/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:

cd /data/kafka
ll

ss -lnt (used to troubleshoot errors)
[root@SG20 kafka]# ss -lnt
State      Recv-Q Send-Q       Local Address:Port                      Peer Address:Port              
LISTEN     0      50           172.16.20.100:9092                                 *:*                  
LISTEN     0      50                       *:2181                                 *:*                  
LISTEN     0      128              127.0.0.1:1234                                 *:*                  
LISTEN     0      128                      *:22                                   *:*                  
LISTEN     0      50                       *:43479                                *:*                  
LISTEN     0      50                       *:46552                                *:*                  
LISTEN     0      100              127.0.0.1:25                                   *:*                  
LISTEN     0      128                   [::]:47761                             [::]:*                  
LISTEN     0      128                   [::]:47762                             [::]:*                  
LISTEN     0      128                   [::]:47763                             [::]:*                  
LISTEN     0      128                   [::]:47764                             [::]:*                  
LISTEN     0      128                   [::]:47765                             [::]:*                  
LISTEN     0      128                   [::]:22                                [::]:* 




cd /usr/share/zeek/site/scripts

sudo curl -LO http://192.168.2.20:8080/zeek_scripts/kafka.zeek
[root@SG20 scripts]# ll
total 12
-rw-r--r--. 1 zeek zeek 102 Mar 30 13:25 afpacket.zeek
-rw-r--r--. 1 zeek zeek 827 Mar 30 13:31 extension.zeek
-rw-r--r--. 1 root root 703 Mar 30 17:09 kafka.zeek

[root@SG20 scripts]# vi kafka.zeek

line 7     ["metadata.broker.list"] = "<172.16.20.100>:9092");


cd ..
vi local.zeek

@load ./scripts/afpacket.zeek
@load ./scripts/extension.zeek
@load ./scripts/kafka.zeek

systemctl restart zeek

sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.20.100:9092 --list zeek-raw
[root@SG20 scripts]# sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.20.100:9092 --list
zeek-raw

if zeek-raw doesn't show(virify the

sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.20.100:9092 --describe --topic zeek-raw



sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.20.100:9092 --topic zeek-raw


Kafka folders: /usr/share/kafka/bin
               /usr/share/zeek/site/scripts


__________________________________________________________________________________________________

FSF
- uses analytical tools to analyze the file.
uses YARA signatures

sudo yum install fsf

if we wanted to use specific repo (repo="--disable*" --enablerepo=local*)

sudo vi /opt/fsf/fsf-server/conf/config.py
line 9 SCANNER_CONFIG = { 'LOG_PATH' : '/data/fsf/logs',
     10                    'YARA_PATH' : '/var/lib/yara-rules/rules.yara',
     11                    'PID_PATH' : '/runfsf/fsf.pid',
     12                    'EXPORT_PATH' : '/data/fsf/archive',
     13                    'TIMEOUT' : 60,
     14                    'MAX_DEPTH' : 10,
     15                    'ACTIVE_LOGGING_MODULES' : ['scan_log', 'rockout'],
     16                    }
     17 
     18 SERVER_CONFIG = { 'IP_ADDRESS' : "localhost",

sudo mkdir -p /data/fsf/{logs,archive}
ll
root@SG20 fsf]# sudo mkdir -p /data/fsf/{logs,archive}
[root@SG20 fsf]# ll
total 0
drwxr-xr-x. 2 root root 6 Mar 30 18:11 archive
drwxr-xr-x. 2 root root 6 Mar 30 18:11 logs


sudo vi /opt/fsf/fsf-client/conf/config.py  (client configuration no changes)
-___________________________________________________________________-----------
[root@SG20 bin]# sudo vi /opt/fsf/fsf-server/conf/config.py
You have new mail in /var/spool/mail/root
[root@SG20 bin]# cd data/fsf/logs
bash: cd: data/fsf/logs: No such file or directory
[root@SG20 bin]# cd /data/fsf/logs
bash: cd: /data/fsf/logs: No such file or directory
[root@SG20 bin]# cd /data/fsf/logs/
bash: cd: /data/fsf/logs/: No such file or directory
[root@SG20 bin]# cd /data/
[root@SG20 data]# ll
total 5260
-rw-r--r--.  1 root         root         5378247 Mar 29 19:22 all-class-files.zip
drwxr-xr-x.  2 root         root               6 Mar 29 18:44 elasticsearch
drwxr-xr-x.  2 root         root               6 Mar 29 18:44 fsf
drwxr-xr-x. 55 kafka        kafka           4096 Mar 30 18:10 kafka
drwxr-xr-x.  4 stenographer stenographer      34 Mar 29 19:13 stenographer
drwxr-xr-x.  2 suricata     suricata          42 Mar 29 19:21 suricata
drwxr-xr-x.  3 zeek         zeek              39 Mar 30 17:34 zeek
[root@SG20 data]# cd fsf/
[root@SG20 fsf]# ll
total 0
[root@SG20 fsf]# sudo mkdir -p /data/fsf/{logs,archive}
[root@SG20 fsf]# ll
total 0
drwxr-xr-x. 2 root root 6 Mar 30 18:11 archive
drwxr-xr-x. 2 root root 6 Mar 30 18:11 logs
[root@SG20 fsf]# sudo chown -R fsf: /data/fsf
[root@SG20 fsf]# sudo chmod -R 0755 /data/fsf
[root@SG20 fsf]# ll
total 0
drwxr-xr-x. 2 fsf fsf 6 Mar 30 18:11 archive
drwxr-xr-x. 2 fsf fsf 6 Mar 30 18:11 logs
[root@SG20 fsf]# cd ..
[root@SG20 data]# ll
total 5260
-rw-r--r--.  1 root         root         5378247 Mar 29 19:22 all-class-files.zip
drwxr-xr-x.  2 root         root               6 Mar 29 18:44 elasticsearch
drwxr-xr-x.  4 fsf          fsf               33 Mar 30 18:11 fsf
drwxr-xr-x. 55 kafka        kafka           4096 Mar 30 18:13 kafka
drwxr-xr-x.  4 stenographer stenographer      34 Mar 29 19:13 stenographer
drwxr-xr-x.  2 suricata     suricata          42 Mar 29 19:21 suricata
drwxr-xr-x.  3 zeek         zeek              39 Mar 30 17:34 zeek
[root@SG20 data]# sudo vi /opt/fsf/fsf-client/conf/config.py
[root@SG20 data]# 

___________________________________

[root@SG20 data]# sudo vi /etc/systemd/system/fsf.service ( add the below script)

[Unit]
Description=File Scanning Framework (FSF-Server) Service
After=network.target
 
[Service]
Type=forking
User=fsf
Group=fsf
WorkingDirectory=/
PIDFile=/run/fsf/fsf.pid
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -P /run/fsf
ExecStartPre=/bin/chown -R fsf:fsf /run/fsf
ExecStart=/opt/fsf/fsf-server/main.py start
ExecStop=/opt/fsf/fsf-server/main.py stop
ExecReload=/opt/fsf/fsf-server/main.py restart
 
[Install]
WantedBy=multi-user.target


fix : /opt/fsf/fsf-server/conf/config.py

#!/usr/bin/env python
#
# Basic configuration attributes for scanner. Used as default
# unless the user overrides them.
#

import socket

SCANNER_CONFIG = { 'LOG_PATH' : '/data/fsf/logs',
                   'YARA_PATH' : '/var/lib/yara-rules/rules.yara',
                   'PID_PATH' : '/run/fsf/fsf.pid',
                   'EXPORT_PATH' : '/data/fsf/archive',
                   'TIMEOUT' : 60,
                   'MAX_DEPTH' : 10,
                   'ACTIVE_LOGGING_MODULES' : ['scan_log', 'rockout'],
                   }

SERVER_CONFIG = { 'IP_ADDRESS' : "localhost",
                  'PORT' : 5800 }


[root@SG20 fsf]# cd
[root@SG20 ~]# su admin
[admin@SG20 root]$ cd
[admin@SG20 ~]$ touch test
[admin@SG20 ~]$ vi test

vi test

/opt/fsf/fsf-client/fsf_client.py --full test

vi rockout.log

[admin@SG20 ~]$ sudo mkdir /data/zeek/extracted_files
[sudo] password for admin: 
[admin@SG20 ~]$ sudo chown -R zeek: /data/zeek/extracted_files
[admin@SG20 ~]$ sudo chmod 0755 /data/zeek/extracted_files

sudo vi /usr/share/zeek/site/scripts/extract_files.zeek"   (creating script for zeek to talk to fsf)

@load /usr/share/zeek/policy/frameworks/files/extract-all-files.zeek
redef FileExtract::prefix = "/data/zeek/extracted_files/";
redef FileExtract::default_limit = 1048576000;


sudo vi /usr/share/zeek/site/local.zeek

@load ./scripts/afpacket.zeek
@load ./scripts/extension.zeek
@load ./scripts/kafka.zeek
@load ./scripts/extract_files.zeek

curl -L -O http://192.168.2.20:8080/putty.exe

sudo /usr/share/zeek/site/scripts/fsf.zeek

[admin@SG20 ~]$ ls -l /data/zeek/extracted_files/
total 1612
-rw-r--r--. 1 zeek zeek 1647912 Mar 30 19:21 extract-1680204078.916322-HTTP-FMZ1CL1ZAx7PE8Spfb
[admin@SG20 ~]$ sudo /usr/share/zeek/site/scripts/fsf.zeek
[sudo] password for admin: 
sudo: /usr/share/zeek/site/scripts/fsf.zeek: command not found
[admin@SG20 ~]$ cd /usr/share/zeek/site/scripts/
[admin@SG20 scripts]$ sudo curl -L -O http://192.168.2.20:8080/zeek_scripts/fsf.zeek


vi fsf.zeek             (change the location to "extracted_files"

10                local file_path = "/data/zeek/extracted_files/";


[admin@SG20 scripts]$ sudo vi /usr/share/zeek/site/local.zeek
[admin@SG20 scripts]$ sudo curl -L -O http://192.168.2.20:8080/putty.exe
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1609k  100 1609k    0     0  29.1M      0 --:--:-- --:--:-- --:--:-- 29.6M
[admin@SG20 scripts]$ cd /data/fsf/logs
[admin@SG20 logs]$ ll
total 12
-rw-r--r--. 1 fsf fsf 2233 Mar 30 19:36 daemon.log
-rw-rw-rw-. 1 fsf fsf    0 Mar 30 18:50 dbg.lock
-rw-rw-rw-. 1 fsf fsf    0 Mar 30 18:50 dbg.log
-rw-rw-rw-. 1 fsf fsf    0 Mar 30 18:50 rockout.lock
-rw-rw-rw-. 1 fsf fsf 1004 Mar 30 18:50 rockout.log
-rw-rw-rw-. 1 fsf fsf    0 Mar 30 18:50 scan_log.lock
-rw-rw-rw-. 1 fsf fsf  697 Mar 30 18:50 scan_log.log
[admin@SG20 logs]$ vi rockout.log (to see 



































































