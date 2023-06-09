# Sensor Repo section

ssh sensor
`sudo yum list zeek` this should be an error, want it to pull locally
`mkdir ~/archive` begin archiving old repo files
`ll /etc/yum.repos.d` default repos live here
`sudo mv /etc/yum.repos.d/* ~/archive/` moving default repos to archive
`ll /etc/yum.repos.d/` checking if the dir is empty

verify local.repo is on fileshare  
`cd /etc/yum.repos.d/`
`sudo curl -LO http://repo:8000/local.repo` grab the file
`sudo vi /etc/yum.repos.d/local.repo`

```
[local-base]
name=local-base
baseurl=http://repo:8008/local-base/
enabled=1
gpgcheck=0

[local-rocknsm-2.5]
name=local-rocknsm-2.5
baseurl=http://repo:8008/local-rocknsm-2.5/
enabled=1
gpgcheck=0

[local-elasticsearch-7.x]
name=local-elasticsearch-7.x
baseurl=http://repo:8008/local-elastic-7.x/
enabled=1
gpgcheck=0

[local-epel]
name=local-epel
baseurl=http://repo:8008/local-epel/
enabled=1
gpgcheck=0

[local-extras]
name=local-extras
baseurl=http://repo:8008/local-extras/
enabled=1
gpgcheck=0

[local-updates]
name=local-updates
baseurl=http://repo:8008/local-updates/
enabled=1
gpgcheck=0
```
`sudo yum makecache fast` making a local cache fast...
`sudo yum list zeek` to verify that the local repo is being referenced

---
Do these steps after configuring secure conenctions on repo

`sudo vi /etc/yum.repos.d/local.repo`

```
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
```

`for host in elastic{0..2} pipeline{0..2} kibana; do scp ~/archive/* $host:/home/elastic ; done`

mkdir ~/archive  
sudo mv /etc/yum.repos.d/* ~/archive  
sudo vi /etc/yum.repos.d/local.repo

put all the repos in each box

rm *.repo

ll /etc/yum.repos.d

mssh name1 name2 name3

> this whole section is messed up, fix for loop will be presented in the future.

---

## Takeaways
- Pulled all repo files and modified repo pointer to look for updates locally


## copy local cert from repo container to all other containers
`for host in sensor elastic{0..2} pipeline{0..2} kibana; do scp ~/certs/localCA.crt elastic@$host:/home/elastic/localCA.crt ; done`

1. `sudo mv ~/localCA.crt /etc/pki/ca-trust/source/anchors/localCA.crt`
2. `sudo update-ca-trust`


`sudo vi /etc/nginx/conf.d/proxy.conf` on repo
`location /packages/` this needed to be changed, dev notified
`sudo yum makecache fast` verify your local.repo file, things were duplicated

> figure out how to organize this before the capstone

## Student laptop again
`sudo scp elastic@repo:/home/elastic/certs/localCA.crt ~/localCA.crt`

add in this cert to chrome web browser
https://repo/fileshare/
successfully implemented tls on fileshare and repo











