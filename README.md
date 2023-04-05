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
  `sudo systemctl status filebeat`
2. Will my monitoring interface persist on reboot?  
3. Can I pull pcap from steno?  
  `ping 192.168.2.20`  
  `sudo stenoread 'host 192.168.2.20' -nn`  
4. Do I have suricata logs?  
  `tail -f eve.json`
5. Do I have zeek logs?  
  `ll /data/zeek/current`
6. Is zeek streaming logs to kafka topic?  
  `sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server 172.16.30.100:9092 --topic zeek-raw`
7. Is zeek doing file extraction?  
8. Does fsf analysis/logging work?  
`sudo curl 192.168.2.20:8080/putty.exe`  
`ll /data/zeek/extracted_files`
