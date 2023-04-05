### Logstash
---
`sudo yum install logstash-7.8.1`  
`cd /etc/logstash/`  
`sudo rm -rf conf.d`  
`curl -LO 192.168.2.20:8080/logstash.tar`  
`tar -xf logstash.tar`  

`[root@SG-30 logstash]# sudo vi conf.d/logstash-100-input-kafka-zeek.conf`
```
1 input {
2   kafka {
3     topics => ["zeek-raw"]
4     add_field => { "[@metadata][stage]" => "zeek_json" }
5     # Set this to one per kafka partition to scale up
6     #consumer_threads => 4
7     group_id => "zeek_logstash"
8     bootstrap_servers => "172.16.30.100:9092"
9     codec => json
10     auto_offset_reset => "earliest"
11     id => "input-kafka-zeek"
12   }
13 }
```
`[root@SG-30 logstash]# sudo vi conf.d/logstash-100-input-kafka-suricata.conf`
```
1 input {
2   kafka {
3     topics => ["suricata-raw"]
4     add_field => { "[@metadata][stage]" => "suricata_json" }
5     # Set this to one per kafka partition to scale up
6     #consumer_threads => 4
7     group_id => "suricata_logstash"
8     bootstrap_servers => "172.16.30.100:9092"
9     codec => json
10     auto_offset_reset => "earliest"
11     id => "input-kafka-suricata"
12   }
13 }
```
`[root@SG-30 logstash]# sudo vi conf.d/logstash-100-input-kafka-fsf.conf`
```
1 input {
2   kafka {
3     topics => ["fsf-raw"]
4     add_field => { "[@metadata][stage]" => "fsfraw_kafka" }
5     # Set this to one per kafka partition to scale up
6     #consumer_threads => 4
7     group_id => "fsf_logstash"
8     bootstrap_servers => "172.16.30.100:9092"
9     codec => json
10     auto_offset_reset => "earliest"
11     id => "input-kafka-fsf"
12   }
13 }
```
`[root@SG-30 logstash]# sudo vi conf.d/logstash-9999-output-elasticsearch.conf`
```
1 output {
2   #stdout { codec => json }
3   # Requires event module and category
4   if [event][module] and [event][category] {
5
6     # Requires event dataset
7     if [event][dataset] {
8       elasticsearch {
9           hosts => [ "172.16.30.100" ]
10           index => "ecs-%{[event][module]}-%{[event][category]}-%{+YYYY.MM.dd}"
11           manage_template => false
12       }
13     }
14
15     else {
16       # Suricata or Zeek JSON error possibly, ie: Suricata without a event.dataset seen with filebeat error, but doesn't have a tag
17       if [event][module] == "suricata" or [event][module] == "zeek" {
18         elasticsearch {
19             hosts => [ "172.16.30.100" ]
20             index => "parse-failures-%{+YYYY.MM.dd}"
21             manage_template => false
22         }
23       }
24       else {
25         elasticsearch {
26             hosts => [ "172.16.30.100" ]
27             index => "ecs-%{[event][module]}-%{[event][category]}-%{+YYYY.MM.dd}"
28             manage_template => false
29         }
30       }
31     }
32   }
33
34   else if [@metadata][stage] == "fsfraw_kafka" {
35     elasticsearch {
36         hosts => [ "172.16.30.100" ]
37         index => "fsf-%{+YYYY.MM.dd}"
38         manage_template => false
39     }
40
41   }
42
43   else if [@metadata][stage] == "_parsefailure" {
44     elasticsearch {
45         hosts => [ "172.16.30.100" ]
46         index => "parse-failures-%{+YYYY.MM.dd}"
47         manage_template => false
49   }
50
51   # Catch all index that is not RockNSM or ECS or parse failures
52   else {
53     elasticsearch {
54         hosts => [ "172.16.30.100" ]
55         index => "indexme-%{+YYYY.MM.dd}"
56         manage_template => false
57     }
58   }
59
60 }
```

`:%s/127.0.0.1/172.16.30.100`  

Using the string at the bottom we can replace all loopback addresses with our static sensor IP. Comment out line 2.  

`sudo systemctl start logstash`  
`sudo systemctl enable logstash`  
`sudo systemctl status logstash`  

`tail -f /var/log/logstash/logstash-plain.log`
