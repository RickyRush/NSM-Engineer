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
  `sudo /usr/share/kafka/bin/kafka-topics.sh --boostrap-server 172.16.30.100 --list`  
  `sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.30.100:9092 --topic zeek-raw`
7. Is zeek doing file extraction?  
8. Does fsf analysis/logging work?  
`sudo curl 192.168.2.20:8080/putty.exe`  
`ll /data/zeek/extract_files`  
`/opt/fsf/fsf-client/fsf_client.py --full ~/bababooey`  
`cat /data/fsf/logs/rockout.log | jq`  
9. Does Elasticsearch work?   
  `curl 172.16.30.100:9092`  
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
347  systemctl status kibana
348  systemctl status logstash
349  systemctl status filebeat
350  systemctl status fsf
351  systemctl status kafka
352  systemctl status zookeeper
353  systemctl status zeek
354  systemctl status suricata
355  systemctl status stenographer
356  stenoread 'host 172.16.30.100'
357  tail -f eve.json
358  tail -f /data/suricata/eve.json
361  cat /var/spool/zeek/logger/conn.log
362  /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.30.100:9092 --topic zeek-raw
363  ll /data/zeek/extract_files/
365  cat /data/fsf/logs/rockout.log
366  /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server 172.16.30.100:9092 --list
```
