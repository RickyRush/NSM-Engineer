# NSM-Engineer

sudo git clone <url>

to update:
(from directory)
sudo git pull

to upload:
  (from directory)
  sudo git add .
  to commit changes to hub:
  sudo git commit -m "file name"
  sudo git push

---

### Steps to complete successful rebuild
---

1. Install and configure CentOS to the sensor.  (sensor.md)
2. Create the local repo  (repo.md)
3. Set enp5s0 to be promiscuous (sniffing traffic.md)
4. Install the services (steno, suricata, zeek, fsf, kafka, elasticsearch)
5.

If having issues connecting after wiping sensor -  
`[student@student37 ~]$ cd /home/student/.ssh/`
```
[student@student37 .ssh]$ ll
total 12
-rw-------. 1 student docker 1679 Mar 27 15:58 id_rsa
-rw-r--r--. 1 student docker  411 Mar 27 15:58 id_rsa.pub
-rw-r--r--. 1 student docker  351 Mar 28 14:53 known_hosts
[student@student37 .ssh]$ rm -rf known_hosts
```
This will wipe the SSH fingerprint on record for the sensor  

Test Questions
---
1. Are all services running?  
  `sudo systemctl status stenographer`  
  `sudo systemctl status suricata`  
  `sudo systemctl status zeek`  
  `sudo systemctl status fsf`  
  `sudo systemctl status kibana`  
  `sudo systemctl status kafka`  
  `sudo systemctl status zookeeper`  
  `sudo systemctl status logstash`  
  `sudo systemctl status elasticsearch`  
  `sudo systemctl status filebeat`
2. Will my monitoring interface persist on reboot?  
3. Can I pull pcap from steno?  
  `ll /data/stenographer/packets`  
  `sudo stenoread 'port 22' -nn`  
4. Do I have suricata logs?  
  `curl -LO 192.168.2.20:8080/all-class-files.zip`  
  `cat -f eve.json | jq`  
5. Do I have zeek logs?  
  `cat /data/zeek/logger/conn.log`
6. Is zeek streaming logs to kafka topic?  
  `sudo /usr/share/kafka/bin/kafka-topics.sh --boostrap-server 172.16.30.100:9092 --list`  
  `sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.30.100:9092 --topic zeek-raw`
7. Is zeek doing file extraction?  
8. Does fsf analysis/logging work?  
`sudo curl 192.168.2.20:8080/putty.exe`  
`ll /data/zeek/extract_files`  
`/opt/fsf/fsf-client/fsf_client.py --full ~/bababooey`  
`cat /data/fsf/logs/rockout.log | jq`  
9. Does Elasticsearch work?   
  `curl 172.16.30.100:9200`  
10. Does Kibana work?  
  `browse to 172.16.30.100:5601`  
  `verify traffic in discover tab`  
11. Does filebeat work?  
  `sudo /usr/share/kafka/bin/kafka-topics.sh --boostrap-server 172.16.30.100 --list`  
  Verify fsf-raw and suricata-raw exist  
12. Does logstash work?  
  `sudo tail -f /var/log/logstash/logstash-plain.log`  
  `watch -d curl 172.16.30.100:9200/_cat/indices?v`  
```
347  systemctl status kibana logstash filebeat fsf kafka zookeeper zeek suricata stenographer
356  stenoread 'host 172.16.30.100'
357  tail -f eve.json
358  tail -f /data/suricata/eve.json
361  cat /var/spool/zeek/logger/conn.log
362  /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.30.100:9092 --topic zeek-raw
363  ll /data/zeek/extract_files/
365  cat /data/fsf/logs/rockout.log
366  /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.30.100:9092 --list
```

FIREWALL change > added port 561 and removed 5601. changed back.    
```
firewall-cmd --remove-port=561/tcp --permanent
firewall-cmd --add-port=5601/tcp --permanent
```

kafka.zeek script > zeek-raww > changed port to 9029. change back to 9092.. changed IP to loopback. change back to sensor.     
/etc/kafka/server.properties > changed zookeeper port to 9092. change back to 2181.   
/etc/kafka/server.properties > changed logs output to /data/kakfa. change back to /data/kafka   

CHOWN all /data/ folders.   

```
sudo chmod 755 /usr/lib/systemd/system/elasticsearch.service.d
sudo chmod 644 /usr/lib/systemd/system/override.conf
sudo systemctl daemon-reload
```

Logstash-input zeek.conf > zeek-raww ; sensor ip change, port change (change back to 9092)  
Filebeat.yaml > suricaata-raww, evee.json, reoot = treat  

Test Day
---
Changed /data permissions back to root  
Changed firewall to port 561 instead of 5601  
`firewall-cmd --remove-port=561/tcp --permanent`  
`firewall-cmd --add-port=5601/tcp --permanent`  
Changed permissions of /usr/lib/systemd/system/elasticsearch.service.d.   
`sudo chmod 755 /usr/lib/systemd/system/elasticsearch.service.d`  
Changed kafka.zeek script. Edited port to 9029.  
