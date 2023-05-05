# First rebuild attempt

1. list out containers
2. login, edit interface file, host file, restart network
---
3. can skip the ssh shortcut
4. make archive copy default repo and create local.repo on each device
5. `for host in sensor elastic{0..2} pipeline{0..2} kibana; do scp ~/certs/localCA.crt elastic@$host:/home/elastic/localCA.crt ; done` - on repo

`sudo scp elastic@repo:/home/elastic/certs/localCA.crt ~/localCA.crt` - on student laptop

`mkdir ~/archive`
`sudo mv /etc/yum.repos.d/* ~/archive`  
`sudo vi /etc/yum.repos.d/local.repo`  
`sudo mv ~/localCA.crt /etc/pki/ca-trust/source/anchors/localCA.crt`  
`sudo update-ca-trust`  
`sudo yum makecache fast`  
`sudo yum list zeek`

[local-base]
name=local-base
baseurl=https://repo/packages/local-base/
enabled=1
gpgcheck=0

[local-rocknsm-2.5]
name=local-rocknsm-2.5
baseurl=https://repo/packages/local-rocknsm-2.5/
enabled=1
gpgcheck=0

[local-elasticsearch-7.x]
name=local-elasticsearch-7.x
baseurl=https://repo/packages/local-elastic-7.x/
enabled=1
gpgcheck=0

[local-epel]
name=local-epel
baseurl=https://repo/packages/local-epel/
senabled=1
gpgcheck=0

[local-extras]
name=local-extras
baseurl=https://repo/packages/local-extras/
enabled=1
gpgcheck=0

[local-updates]
name=local-updates
baseurl=https://repo/packages/local-updates/
enabled=1
gpgcheck=0

---

 follow Configure Capture Interface on config_sensor

 follow configure steno

 follow zeek_install








