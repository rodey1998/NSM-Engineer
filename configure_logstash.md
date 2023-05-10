# Information

- Transforms events into common formats (linxus,windows logs)
- Mutates based on mutate filter
- Translate filter. 1 to 1 transfer
- Enriches information

# Setup - Install & Configure Logstash

- Begin on pipeline 0

#/bin/bash

sudo yum install logstash -y  
sudo curl -LO https://repo/fileshare/logstash/logstash.tar.gz  
sudo tar -zxvf logstash.tar.gz -C /etc/logstash  
sudo chown logstash: /etc/logstash/conf.d/*  
sudo chmod -R 744 /etc/logstash/conf.d/ruby  
cd /etc/logstash/conf.d/

sudo sed -i s/127.0.0.1:9092/pipeline0:9092,pipeline1:9092,pipeline2:9092/g logstash-100-input-kafka-{suricata,fsf,zeek}.conf

sudo sed -i 's/"127.0.0.1"/"elastic0", "elastic1", "elastic2"/g' logstash-9999-output-elasticsearch.conf

sudo -u logstash /usr/share/logstash/bin/logstash -t --path.settings /etc/logstash

---

cp command here - do this later

chmod +xs filename

./filename

> TESTING LOGSTASH - Takes a little bit - 6 passed 0 failed

> Once done do the other pipelines

---

sudo systemctl enable logstash --now
sudo systemctl status logstash
