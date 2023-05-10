> Kafka organizes data into topics

# Kafka Install - on Pipelines

1. `ssh pipeline0`
2. `sudo yum install kafka zookeeper -y` - install kafka and zookeeper packages
3. `sudo mkdir -p /data/zookeeper`
4. `sudo chown -R zookeeper: /data/zookeeper/`
5. `sudo vi /etc/zookeeper/zoo.cfg` - should see content

```
# where zookeeper will store its data
 dataDir=/data/zookeeper

 # what port should clients like kafka connect on
 clientPort=2181

 # how many clients should be allowed to connect, 0 = unlimited
 maxClientCnxns=0

 # list of zookeeper nodes to make up the cluster
 # First port is how followers and leaders communicate
 # Second port is used during the election process to determine a leader
 server.1=pipeline0:2888:3888
 server.2=pipeline1:2888:3888
 server.3=pipeline2:2888:3888

 # more than one zookeeper node will have a unique server id.
 # Ex.) server.1, server.2, etc..

 # milliseconds in which zookeeper should consider a single tick
 tickTime=2000

 # amount of ticks a follow has to connect and sync with the leader
 initLimit=5

 # amount of ticks a follower has to sync with a leader before being dropped
 syncLimit=2

```
6. `sudo touch /data/zookeeper/myid` - new file
7. `sudo chown zookeeper: /data/zookeeper/myid`
8. `echo '3' | sudo tee /data/zookeeper/myid`
9. `cat /data/zookeeper/myid` - verify content - each pipeline will have its own id
10. `sudo firewall-cmd --add-port={2181,2888,3888}/tcp --permanent` - 2181 zookeeper default
11. `sudo firewall-cmd --reload`

> switch to pipeline1 and do steps 1-11. Ensure myid is 1,2,3
pipeline0 = 1
pipeline1 = 2
pipeline2 = 3

> start zookeeper on all pipelines
12. `sudo systemctl enable zookeeper --now` - start zookeeper
13. `sudo systemctl status zookeeper` - status

14. `for host in pipeline{0..2}; do (echo "stats" | nc $host 2181 -q 2); done` - from student laptop to verify cluster

> check mode, 2 followers, 1 leader

## Configure Kafka - on each pipeline

1. `sudo mkdir -p /data/kafka`
2. `sudo chown kafka: /data/kafka/`
3. `sudo cp /etc/kafka/server{.properties,.properties.bk}` - backup of properties
4. `sudo vi /etc/kafka/server.properties` - should see stuff %d blow it away use below

```
# The unique id of this broker should be different for each kafka node. Good practice is to match the kafka broker id to the zookeeper server id.
broker.id=0

# the port in wich kafka should use to communicate with other kafka clients
port=9092
# the hostname or IP address in which the server listens on
listeners=PLAINTEXT://pipeline0:9092

# hostname that will be advertised to producers and consumers
advertised.listeners=PLAINTEXT://pipeline0:9092

# number of threads used to send network responses
num.network.threads=3

# number of threads used to make I/O requests
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# where kafka should write its data to
log.dirs=/data/kafka

# how many partitions and replicas should be generated for topics that are created by other software
num.partitions=3
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=2
default.replication.factor = 3
min.insync.replicas = 2

# how many threads should be used for shutdown and start up
num.recovery.threads.per.data.dir=3

# how long should we retain logs in kafka
log.retention.hours=12
log.retention.bytes=90000000000

# max size of a single log file
log.segment.bytes=1073741824

# frequency in miliseconds to check if a log needs to be deleted
log.retention.check.interval.ms=300000
log.cleaner.enable=false

# will not allow a node to be elected leader if it is not in sync with other nodes. Prevents possible missing messages
unclean.leader.election.enable=false

# automatically create topics from external software
auto.create.topics.enable=false


# how to connect kafka to zookeeper
zookeeper.connect=pipeline0:2181,pipeline1:2181,pipeline2:2181
zookeeper.connection.timeout.ms=30000

```
> change these to reflect box you're on  
line 3 `broker.id = 0`  
line 13 - `listeners`  
line 19 - `advertised`  

> /etc/kafka/server.properties
pipeline0 - broker.id =0
pipeline1 - broker.id =1
pipeline2 - broker.id =2

5. `sudo firewall-cmd --add-port=9092/tcp --permanent`
6. `sudo firewall-cmd --reload`
7. `sudo systemctl start kafka`

> At this point zookeeper and kafka should be up and running! GREENAGE!

---

## Creating Kafka Topics

> Only do this on 0. To validate its working properly

1. `sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --create --topic test --partitions 3 --replication-factor 3`

>Commands from Dustin in Slack

```
Lists The Topic:
`sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --list` 

Lists Out The Topic & Shows Replication/Partition
`sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --describe --topic test` 

Creates The Topic:
`sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --create --topic test --partitions 3 --replication-factor 3`

```
2. `sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --delete --topic test` - deletes the test just created

> list topics again, should not have test

3. `cd /usr/share/zeek/site/scripts/` - check if you have kafka.zeek, on sensor
4. `sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic zeek-raw` - makes zeek-raw
5. `sudo /usr/share/kafka/bin/kafka-topics.sh --describe --zookeeper pipeline0:2181 --topic zeek-raw` - validate
6. `sudo vi /usr/share/zeek/site/local.zeek` - on sensor
7. shift g `@load ./scripts/kafka.zeek` add at bottom
8. `sudo -u zeek zeekctl deploy` - restart zeek and check status
9. `sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic zeek-raw` - back on pipeline0

`cd /usr/share/zeek/site/scripts/` - sensor
`sudo vi kafka.zeek` - plugin file

line 7 - `["metadata.broker.list"] = "pipeline0:9092,pipeline1:9092,pipeline2:9092");`
deploy zeek
run step 9, this shows that kafka is getting the traffic from zeek















