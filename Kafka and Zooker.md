### Configuring Kafka & Zookeeper
---
Kafka has a dependency - zookeeper. We will need to install both.  
`sudo yum install kafka zookeeper`  

Now we validate our Zookeeper config. no need to make any changes, just ensure it looks like the below file.  
`sudo vi /etc/zookeeper/zoo.cfg`  
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/var/lib/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

If the config matches, we are ready to start zookeeper.  
`sudo systemctl start zookeeper`  
`sudo systemctl enable zookeeper`  
`sudo systemctl status zookeeper`  

---

Now we can begin configuring kafka. Make the following changes at the below lines.  
`sudo vi /etc/kafka/server.properties`  
```
                                        + + +
31 listeners=PLAINTEXT://172.16.30.100:9092
                                        + + +
36 advertised.listeners=PLAINTEXT://172.16.30.100:9092
                                        + + +
60 log.dirs=/data/kafka
                                        + + +
65 num.partitions=3
                                        + + +
107 log.retention.bytes=90000000000                                        
                                        + + +
123 zookeeper.connect=127.0.0.1:2181
                                        + + +
```

Now we need to ensure the proper ownerships are given.  
`sudo chown -R kafka: /data/kafka`  

We now need to update the firewall to allow this communication.  
`sudo firewall-cmd --add-port=9092/tcp --permanent`  
`sudo firewall-cmd --add-port=2181/tcp --permanent`  
`sudo firewall-cmd --reload`  

After this, the service should be ready to start :D  
`sudo systemctl start kafka`  
`sudo systemctl enable kafka`  
`sudo systemctl status kafka`  

Verify the service communication is working properly.  
```
[admin@SG-30 bin]$ ss -lnt
State      Recv-Q Send-Q                                                       Local Address:Port                                                                      Peer Address:Port              
LISTEN     0      128                                                                      *:22                                                                                   *:*                  
LISTEN     0      100                                                              127.0.0.1:25                                                                                   *:*                  
LISTEN     0      50                                                           172.16.30.100:9092                                                                                 *:*   
```

Now we need to add a script to Zeek to make it talk to kafka.  
`cd /usr/share/zeek/site/scripts/`  
`sudo curl -LO 192.168.2.20:8080/zeek_scripts/kafka.zeek`  
`sudo chown zeek: kafka.zeek`  
`sudo vi kafka.zeek`  
```
7     ["metadata.broker.list"] = "172.16.30.100:9092");
```
Now we need to add this script to the local.zeek file.  
`sudo vi /usr/share/zeek/site/local.zeek`  
```
104 @load /usr/share/zeek/site/scripts/afpacket.zeek
105 @load /usr/share/zeek/site/scripts/extension.zeek
106 @load /usr/share/zeek/site/scripts/kafka.zeek
```
`sudo systemctl restart zeek`  

Verify zeek-raw created the necessary topic.    
`sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.30.100:9092 --list`

```
[admin@SG-30 site]$ sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.30.100:9092 --describe --topic zeek-raw
Topic:zeek-raw	PartitionCount:3	ReplicationFactor:1	Configs:segment.bytes=1073741824,retention.bytes=90000000000
	Topic: zeek-raw	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: zeek-raw	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: zeek-raw	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
```

Verify zeek-raw is writing logs to the kafka topic.  
`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.30.100:9092 --topic zeek-raw`
