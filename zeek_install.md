# Zeek Install
> Stores metadata in logs, examples include conn, files, and http.log.
Can you use the UID to coorelate logs.

> Zeek located on sensor.

1. `sudo yum install zeek` - should be pulling locally
2. `sudo yum install zeek-plugin-af_packet` - af packet on nic to send traffic to zeek
3. `sudo yum install zeek-plugin-kafka` - zeek sent over network to kafka

---

## Zeek Config Files

> Zeek config files - `/etc/zeek` - there should be three of them

4. `sudo vi /etc/zeek/zeekctl.cfg` - modify config file
5. `set nu` - lines `set nu!` - no lines
6. Line 67 - `LogDir = /data/zeek` - where logs will be stored for zeek
7. Line 68 - `lb_custom.InterfacePrefix=af_packet::` - tells zeek to use af_packet

8. `sudo vi /etc/zeek/node.cfg` - controls nodes, standalone or node
9. comment out lines 8-11 - removes standalone config
10. uncomment lines 16-31 - configures a cluster
11. line 20 manager add `pin_cpus=1`
12. line 29 worker-1 `interface=eth1`

> should look like this  
29 [worker-1]  
30 type=worker  
31 host=localhost  
32 interface=eth1  
33 lb_method=custom  
34 lb_procs=2  
35 pin_cpus=2,3  
36 env_vars=fanout_id=77

line 33-36 were added after interface change

---

# Extra Packages

> `lscpu -e` - shows how many cpus you have, just a helpful command

13. `sudo mkdir /usr/share/zeek/site/scripts` - make scripts dir  
14. `cd /usr/share/zeek/site/scripts` - custom scripts below exist here  
15. download these files from fileshare server  
`sudo curl -LO https://repo/fileshare/zeek/afpacket.zeek`   
`sudo curl -LO https://repo/fileshare/zeek/extension.zeek`   
`sudo curl -LO https://repo/fileshare/zeek/extract-files.zeek`      
`sudo curl -LO https://repo/fileshare/zeek/fsf.zeek`  
`sudo curl -LO https://repo/fileshare/zeek/json.zeek`  
`sudo curl -LO https://repo/fileshare/zeek/kafka.zeek`

 > `/usr/share/zeek/site` - local.zeek lives here

16. `sudo vi /usr/share/zeek/site/local.zeek` - what zeek is using for scripts
17. `shift g` - bottom of page
18. add the following lines   
`@load ./scripts/afpacket.zeek`  
`@load ./scripts/extension.zeek` - space after this line **MAYBE ERROR**
`redef ignore_checksums = T;`
19. `sudo mkdir -p /data/zeek`
20. change ownership on following dirs  
`sudo chown -R zeek: /data/zeek`  
`sudo chown -R zeek: /etc/zeek`  
`sudo chown -R zeek: /usr/share/zeek`  
`sudo chown -R zeek: /usr/bin/zeek`   
`sudo chown -R zeek: /usr/bin/capstats`  
`sudo chown -R zeek: /var/spool/zeek`
21. set capabilities of user to read network traffic
`sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/zeek`
`sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/capstats`
22. check that capabilites were modified  
`sudo getcap /usr/bin/zeek`  
`sudo getcap /usr/bin/capstats`  

> should see /usr/bin/x_dir = cap_net_admin,cap_net_raw+eip

23. starting the zeek service using zeekctl - SEE zeek_cheatsheet for more useful info  
`sudo -u zeek zeekctl deploy` - deploy zeek as zeek user  
`sudo -u zeek zeekctl status` - check the status of zeek  
24. checking if files have been generated
`ll /data/zeek/current/` - cat any of the files in here, if no files you did something wrong

## Extra zeek scripts

Download Zeek Scripts to sensor from repo (be on sensor)
`sudo yum install wget`
`sudo mkdir ~/downloads/zeek`
`cd ~/downloads/zeek`
`sudo wget -r --no-parent -nd -l1 https://repo/fileshare/zeek`

---
# FSF - File Scanning Framework, plan to replace as its depreciated

1. `sudo yum install fsf`
2. `sudo vi /opt/fsf/fsf-server/conf/config.py` - server config file for fsf

```
1 #!/usr/bin/env python
2 #
3 # Basic configuration attributes for scanner. Used as default
4 # unless the user overrides them.
5 #
6 
7 import socket
8 
9 SCANNER_CONFIG = { 'LOG_PATH' : '/data/fsf',
10                    'YARA_PATH' : '/var/lib/yara-rules/rules.yara',
11                    'PID_PATH' : '/run/fsf/fsf.pid',
12                    'EXPORT_PATH' : '/data/fsf/archive',
13                    'TIMEOUT' : 60,
14                    'MAX_DEPTH' : 10,
15                    'ACTIVE_LOGGING_MODULES' : ['rockout'], 
16                    }
17 
18 SERVER_CONFIG = { 'IP_ADDRESS' : "localhost",
19                   'PORT' : 5800 }

```
3. `sudo mkdir -p /data/fsf/archive`
4. `sudo chown -R fsf: /data/fsf`
5. `sudo vi /opt/fsf/fsf-client/conf/config.py` - client config file for fsf
6. `localhost` instead of loopback address
7. `sudo vi /usr/lib/systemd/system/fsf.service` - nothing to change in here
8. `sudo systemctl enable fsf --now` - enable and start fsf service
9. `sudo systemctl status fsf` - check if service is good GREENAGEEEE

> `journalctl -xeu <service>` - troubleshooting a service, extra info

10. `/opt/fsf/fsf-client/fsf_client.py --full interface.sh` - scan file to ensure fsf is working make sure you run as elastic not root. also be in home directory if not full path

---

# Adding fsf scripts into zeek

1. `sudo vi /usr/share/zeek/site/local.zeek`
2. shift g
3. add to local.zeek  
`@load ./scripts/extract-files.zeek`  
`@load ./scripts/fsf.zeek`  
`@load ./scripts/json.zeek` 

> could comment out the stuff if you want to put it in all at once

## Verify sfs status

4. `sudo -u zeek zeekctl stop` changed zeek config and have to stop
5. `sudo -u zeek zeekctl deploy` start it back up
6. `sudo -u zeek zeekctl status` check status, look for running
7. run commands to see that fsf extracted the request successfully
`cd /data/fsf/` - holds fsf logs
`cat rockout.log | jq` - this stuff will go to filebeat
`curl google.com` - 
`cat rockout.log | jq`

> FSF converts logs from zeek (tab delimited) to json that filebeat can understand.

---

Zeek does not need filebeat because it has a a plugin



































