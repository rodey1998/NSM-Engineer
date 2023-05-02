# Student Laptop

## Commands used

`:%d` deletes contents of file  
`lxc list` and `lxc start all` list all containers

## Credentials
Username : elastic  
Password : training

## Modify IPs to Static  
ssh into each container  
`ssh elastic@x.x.x.x`  
```
disable ipv6 command "sudo vi /etc/sysctl.conf"
look for ipv6 info 1 = disabled
```
vi insert mode i  
vi command :wq  
will need to edit network script  
eth 0 is management interface  
`ip a` to see interfaces  

# Building the containers  
## Accessing interfaces configs

`sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0`

```
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=elastic0
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.30
GATEWAY=10.81.139.1
PREFIX=24
```
## Edit Hostfile
`sudo vi /etc/hosts`

```
127.0.0.1 localhost
10.81.139.10 repo
10.81.139.20 sensor
10.81.139.30 elastic0
10.81.139.31 elastic1
10.81.139.32 elastic2
10.81.139.40 pipeline0
10.81.139.41 pipeline1
10.81.139.42 pipeline2
10.81.139.50 kibana

```
> restart the network  
`sudo systemctl restart network`

## Takeaways
- Verify all containers are up and running 
- Verify label, IP, and trusted hosts