### Filebeat
---
`sudo yum install filebeat-7.8.1`  
`sudo mv /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.bk`  
`sudo curl -LO 192.168.2.20:8080/filebeat.yml`  
`sudo vi /etc/filebeat/filebeat.yml`  
```
1 #=========================== Filebeat prospectors =============================
2 filebeat.inputs:
3   - type: log
4     enabled: true
5     paths:
6       - /data/suricata/eve.json
7     json.keys_under_root: true
8     fields:
9       kafka_topic: suricata-raw
10     fields_under_root: true
11   - type: log
12     enabled: true
13     paths:
14       - /data/fsf/logs/rockout.log
15     json.keys_under_root: true
16     fields:
17       kafka_topic: fsf-raw
18     fields_under_root: true
19
20 #================================ Outputs =====================================
21 output.kafka:
22   hosts: ["172.16.30.100:9092"]

                                          + + +
```

`sudo systemctl start filebeat`  
`sudo systemctl enable filebeat`  
`sudo systemctl status filebeat`  

```
[root@SG-30 filebeat]# sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.30.100:9092 --list
__consumer_offsets
fsf-raw
suricata-raw
zeek-raw
```
`sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.30.100:9092 --topic suricata-raw --from-beginning`  

`rm -rm /usr/var/filebeat/registry`  
