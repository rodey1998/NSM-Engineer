# Configure Filebeat - Lightweight Log Shipper
 > Install on sensor

 1. `sudo yum install filebeat -y` - install packages
 2. `sudo mv /etc/filebeat/filebeat{.yml,.yml.bk}` - making backup
 3. `cd /etc/filebeat/`
 4. `curl -LO https://repo/fileshare/filebeat.yml` - grab filebeat.yml from repo
 5. `sudo curl -LO https://repo/fileshare/filebeat/filebeat.yml`
 6. `cat filebeat.yml` - should see stuff in here
 7. `sudo vi filebeat.yml`

 line 34 - `hosts: ["pipeline0:9092","pipeline1:9092","pipeline2:9092"]`

 8. `sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic fsf-raw`

 9. `sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic suricata-raw`

 10. `sudo /usr/share/kafka/bin/kafka-topics.sh --list --zookeeper pipeline0:2181`

 > Kafka's ability to create topics is turned off in this environment

 > Back on the sensor

 11. `sudo systemctl start filebeat`
 12. `sudo systemctl status filebeat`

 > Done in pipeline0

 Verify messages are added to suricata-raw
`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic suricata-raw --from-beginning`

Verify messages are added to fsf-raw
`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic fsf-raw --from-beginning`

 > File beat installed and pushing to kafka
 

