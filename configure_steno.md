# Install steno (on sensor)
`sudo yum install stenographer`

`cd /etc/stenographer`
`sudo vi /etc/stenographer/config`
Packets dir is where files are being written to - change this

```
{
  "Threads": [
    { "PacketsDirectory": "/data/stenographer/packets"
    , "IndexDirectory": "/data/stenographer/index"
    , "MaxDirectoryFiles": 30000
    , "DiskFreePercentage": 30
    }
  ]
  , "StenotypePath": "/usr/bin/stenotype"
  , "Interface": "eth1"
  , "Port": 1234
  , "Host": "127.0.0.1"
  , "Flags": []
  , "CertPath": "/etc/stenographer/certs"
}

```
`sudo mkdir -p /data/stenographer/{packets,index}`

## user and group changes for security

`sudo chown -R stenographer:stenographer /data/stenographer`
`sudo stenokeys.sh stenographer stenographer`
`ll /etc/stenographer/certs/` - client/server certificate keys

`sudo systemctl enable stenographer --now` - start and enable persists
`sudo systemctl status stenographer` - to check the status of steno
`sudo stenoread 'host 8.8.8.8'` - shows the traffic, not direction specific for ip. After steno filter you can use tcpdump filtering -nn to not resolve ip.

`watch ls -al /data/stenographer/packets` - live look at packets coming in

`df -h` - disk space. normally would make a seperate partition for stenodata

---

# Configure Suricata

`sudo yum install suricata`
`sudo -s ` - sets you to root
`ll /etc/suricata`
`vi /etc/suricata/suricata.yaml` - spaces will mess this file up
`:set nu`

line 56 -> default log dir /data/suricata 
line 60 -> no stats enabled
line 76 -> no output fast
line 404 -> no
line 557 -> console enabled no
line 580 -> interface eth1
line 582 -> uncomment and 3 threads
line 981,2,3 -> uncomment user/group suricata
1434 -> cpu affinity yes
1452 -> worker cpu set 0-2
1459 -> prio medium 1
1460 -> prio high 2
1461 -> default to high
1500 -> no
1515, 16 -> no
1521 -> prefilter no
1527 -> no
1536 -> no

`sudo vi /etc/sysconfig/suricata`

OPTIONS="--af-packet=eth1 --user suricata --group suricata"

sudo suricata-update add-source emergingthreats https://repo/fileshare/emerging.rules.tar.gz

> suricata update to point to fileshare to get emerging rules

`sudo suricata-update`
sudo mkdir -p /data/suricata
sudo chown -R suricata:suricata /data/suricata
ll /data/suricata/
ll /data
sudo systemctl enable suricata --now

> Troubleshooting `journalctl -xeu suricata`






