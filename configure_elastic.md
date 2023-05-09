# Configure Elasticsearch - Single node Config

> starting on elastic0
1. `sudo yum install elasticsearch -y`
2. `sudo mv /etc/elasticsearch/elasticsearch{.yml,.yml.bk}` - making backup yml
3. `sudo curl -LO https://repo/fileshare/elasticsearch/elasticsearch.yml` - yml from repo
4. `sudo vi elasticsearch.yml`

```
cluster.name:  nsm-cluster
node.name:  es-node-0
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
http.port:9200
network.host: _local:ipv4_
discovery.type: single-node

```

5. `sudo mv ~/elasticsearch.yml /etc/elasticsearch/`
6. `sudo chmod 640 /etc/elasticsearch/elasticsearch.yml`
7. `sudo mkdir /usr/lib/systemd/system/elasticsearch.service.d`
8. `sudo chmod 755 /usr/lib/systemd/system/elasticsearch.service.d`
9. `sudo vi /usr/lib/systemd/system/elasticsearch.service.d/override.conf` - new file

```
[Service]
LimitMEMLOCK=infinity

```
10. `sudo chmod 644 /usr/lib/systemd/system/elasticsearch.service.d/override.conf`
11. `sudo systemctl daemon-reload`
12. `sudo vi /etc/elasticsearch/jvm.options.d/jvm_override.conf` - half of available ram dont go over 31GB of RAM

```
-Xms2g
-Xmx2g
```
13. `sudo mkdir -p /data/elasticsearch`
14. `sudo chown elasticsearch: /data/elasticsearch/`
15. `sudo chmod 755 /data/elasticsearch/`
16. `sudo firewall-cmd --add-port={9200,9300}/tcp --permanent`
17. `sudo firewall-cmd --reload`

---

## Multi-node Setup - USE THIS SECTION FOR THE TEST

> Only difference is changes to the yml file.

1. `sudo vi /etc/elasticsearch/elasticsearch.yml`

>node.name will change per node 

```
cluster.name:  nsm-cluster
node.name:  es-node-0
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: _site:ipv4_
http.port: 9200
discovery.seed_hosts: ["elastic0","elastic1","elastic2"]
cluster.initial_master_nodes: ["es-node-0","es-node-1","es-node-2"]
```
> continue with step 5 from above

`sudo chown elasticsearch: /etc/elasticsearch/elasticsearch.yml`

`sudo systemctl start elasticsearch`