### Configuring Suricata
---

SSH into sensor  
`ssh admin@172.16.30.100`  
First things first, install Suricata.  
`sudo yum list suricata`   
You should see local-rocknsm-2.5  
`sudo yum install suricata`

Now we can begin configuration. Must be root to configure Suricata.  
`sudo -s`  
`cd /etc/suricata`  
```
[root@SG-30 suricata]# ll
total 92
-rw-r-----. 1 suricata suricata  4258 Dec 18  2019 classification.config
-rw-r-----. 1 suricata suricata  1375 Dec 18  2019 reference.config
drwxr-x---. 2 suricata suricata  4096 Mar 29 13:39 rules
-rw-r-----. 1 suricata suricata 70174 Dec 18  2019 suricata.yaml
-rw-r-----. 1 suricata suricata  1644 Dec 18  2019 threshold.config
```
`vi suricata.yaml`  
This file is MASSIVE, so I will first be referencing line number, followed by changes to make.
To display line numbers in vi, hit `:set nu` within vi.   
```
56 default-log-dir: /data/suricata/
57
58 # global stats configuration
59 stats:
60  enabled: no
```
```
75 - fast:
76     enabled: no
```
```
403   - stats:
404       enabled: no
```
```
556   - console:
557       enabled: no
```
```
579 af-packet:
580   - interface: enp5s0
581     # Number of receive threads. "auto" uses the number of cores
582     threads: auto

```
Line 582 uncomment "threads: auto"

Now we change suricata to use afpacket    
```
[root@SG-30 sysconfig]# sudo vi /etc/sysconfig/suricata
# The following parameters are the most commonly needed to configure
# suricata. A full list can be seen by running /sbin/suricata --help
# -i <network interface device>
# --user <acct name>
# --group <group name>

# Add options to be passed to the daemon
OPTIONS="--af-packet=enp5s0 --user suricata "
```
Ensure the OPTIONS line equals what is typed above.  

Now we add an 'upstream' for our emerging threat rules, which is what Suricata will look for in traffic. We will pull these from the instructor NUC.  
`sudo suricata-update add-source local-emerging-threats http://192.168.2.20:8080/emerging.rules.tar`  
`sudo suricata-update`  
`sudo suricata-update list-sources`  
to view the rules  
`cat /var/lib/suricata/rules/suricata.rules`  

We still need to apply the user group information to the suricata files.  
`cd /data`  
`sudo chown -R suricata: /data/suricata`

We are now ready to start suricata.

`sudo systemctl start suricata`  
`sudo systemctl enable suricata`  
`sudo systemctl status suricata`
```
[root@SG-30 sysconfig]# sudo systemctl status suricata
● suricata.service - Suricata Intrusion Detection Service
   Loaded: loaded (/usr/lib/systemd/system/suricata.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-03-29 14:26:29 UTC; 44s ago
     Docs: man:suricata(1)
 Main PID: 5234 (Suricata-Main)
   CGroup: /system.slice/suricata.service
           └─5234 /sbin/suricata -c /etc/suricata/suricata.yaml --pidfile /var/run/suricata.pid --af-packet=enp5...
                                      + + +
```

Now we'll try to generate alerts to verify suricata is working properly.  
`curl -LO 192.168.2.20:8080/all-class-files.zip`  
`cd /data/suricata`  
```
[root@SG-30 suricata]# ll
total 64
-rw-r--r--. 1 suricata suricata  2650 Mar 29 14:34 eve.json
-rw-r--r--. 1 suricata suricata 55022 Mar 29 14:26 flowbits.json
-rw-r--r--. 1 root     root      2370 Mar 29 14:26 suricata.log
```
`cat eve.json`  
```
[root@SG-30 suricata]# cat eve.json
{"timestamp":"2023-03-29T14:28:15.000416+0000","flow_id":2174137321576044,"in_iface":"enp5s0","event_type":"flow","src_ip":"192.168.2.78","src_port":48100,"dest_ip":"172.16.30.100","dest_port":22,"proto":"TCP","flow":{"pkts_toserver":67,"pkts_toclient":39,"bytes_toserver":5538,"bytes_toclient":5426,"start":"2023-03-29T14:27:10.775788+0000","end":"2023-03-29T14:27:14.034028+0000","age":4,"state":"new","reason":"timeout","alerted":false},"tcp":{"tcp_flags":"00","tcp_flags_ts":"00","tcp_flags_tc":"00"}}
{"timestamp":"2023-03-29T14:28:17.000450+0000","flow_id":1765338037873938,"in_iface":"enp5s0","event_type":"flow","src_ip":"192.168.2.66","src_port":40696,"dest_ip":"172.16.30.100","dest_port":22,"proto":"TCP","flow":{"pkts_toserver":73,"pkts_toclient":42,"bytes_toserver":5934,"bytes_toclient":6008,"start":"2023-03-29T14:26:47.650514+0000","end":"2023-03-29T14:27:16.047329+0000","age":29,"state":"new","reason":"timeout","alerted":false},"tcp":{"tcp_flags":"00","tcp_flags_ts":"00","tcp_flags_tc":"00"}}
```  

As you can see, alerts were generated into eve.json.  
`tail -f eve.json`  
this will update to console in real time  
