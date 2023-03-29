### Configuring Stenographer
---
SSH into sensor  
`ssh admin@172.16.30.100`

Stenotype -> Disk + Packets <- Stenoread  

Check to see where stenographer is located  
`sudo yum list stenographer`  

We should see local-rocknsm-2.5  

Now that we've verified, we can install stenographer.  
`sudo yum install stenographer`

And we can now begin configuration:  
`cd /etc/stenographer`  
```
[admin@SG-30 stenographer]$ ll
total 4
drwxr-x---. 2 stenographer stenographer   6 Apr  1  2019 certs
-rw-r--r--. 1 root         root         379 Jan 30  2019 config
```
`sudo vi /etc/stenographer/config`  
```
{
  "Threads": [
    { "PacketsDirectory": "/path/to/thread0/packets/directory"
    , "IndexDirectory": "/path/to/thread0/index/directory"
    , "MaxDirectoryFiles": 30000
    , "DiskFreePercentage": 10
    }
  ]
  , "StenotypePath": "/usr/bin/stenotype"
  , "Interface": "em1"
  , "Port": 1234
  , "Host": "127.0.0.1"
  , "Flags": []
  , "CertPath": "/etc/stenographer/certs"
}
```
This file shows where Stenographer will write to disk. We need to update "**Packets Directory**", line 28, to "**/data/stenographer/packets**".
Next, we need to update "**IndexDirectory**", line 29, to "**/data/stenographer/index**".
For StenotypePath, you can run `sudo which stenotype` to locate the binary path for stenographer. Ensure this value matches! Change "interface" to our promiscuous interface, **"enp5s0"**. Since stenographer is running on the sensor, host will remain localhost.

After all changes, the file should look like this:  
```
{
  "Threads": [
    { "PacketsDirectory": "/data/stenographer/packets"
    , "IndexDirectory": "/data/stenographer/index"
    , "MaxDirectoryFiles": 30000
    , "DiskFreePercentage": 10
    }
  ]
  , "StenotypePath": "/bin/stenotype"
  , "Interface": "enp5s0"
  , "Port": 1234
  , "Host": "127.0.0.1"
  , "Flags": []
  , "CertPath": "/etc/stenographer/certs"
}
```

We now need to create the directories referenced in the config file.  
`sudo mkdir /data/stenographer/packets`  
`sudo mkdir /data/stenographer/index`  

Verify file creation
`cd /data/stenographer`  
```
[admin@SG-30 stenographer]$ ll
total 0
drwxr-xr-x. 2 root root 6 Mar 29 12:44 index
drwxr-xr-x. 2 root root 6 Mar 29 12:43 packets
```

Each service we install will create their own user and group, so that they have all the accesses required and will not conflict with SELinux. However, they do not have these set by default.

`sudo chown -R stenographer:stenographer /data/stenographer`  
```
[admin@SG-30 stenographer]$ ll
total 0
drwxr-xr-x. 2 stenographer stenographer 6 Mar 29 12:44 index
drwxr-xr-x. 2 stenographer stenographer 6 Mar 29 12:43 packets
```

Now we will create stenographer keys. This script will generate certificates needed for the service to communicate over the socket.  
`sudo stenokeys.sh stenographer stenographer`

We are now ready to start stenographer.  
`sudo systemctl start stenographer`  
`sudo systemctl enable stenographer`  
`sudo systemctl status stenographer`
```
[admin@SG-30 stenographer]$ sudo systemctl status stenographer
● stenographer.service - packet capture to disk
   Loaded: loaded (/usr/lib/systemd/system/stenographer.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-03-29 12:54:30 UTC; 2min 45s ago
 Main PID: 4450 (stenographer)
   CGroup: /system.slice/stenographer.service
           ├─4450 /usr/bin/stenographer
           └─4484 /bin/stenotype --threads=1 --iface=enp5s0 --dir=/tmp/stenog...
                                      + + +

```
Troubleshooting pro tip to read the service logs  
`sudo journalctl -xeu stenographer`
```
-- Logs begin at Tue 2023-03-28 19:17:18 UTC, end at Wed 2023-03-29 13:00:25 UTC. --
Mar 29 12:54:30 SG-30.local.lan systemd[1]: Started packet capture to disk.
-- Subject: Unit stenographer.service has finished start-up
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit stenographer.service has finished starting up.
--
-- The start-up result is done.
Mar 29 12:56:50 SG-30.local.lan stenographer[4450]: 2023-03-29T12:56:50.880548Z T:f0e957 [util.cc:117] WATCHDOG FAI
                                                    ABORTABORTABORT
Mar 29 12:56:50 SG-30.local.lan stenographer[4450]: /bin/stenotype() [0x4063fc]
                                                    /lib64/libstdc++.so.6(+0xb5330) [0x7f4473034330]
                                                    /lib64/libpthread.so.0(+0x7ea5) [0x7f44736d5ea5]
                                                    /lib64/libc.so.6(clone+0x6d) [0x7f447279796d]
Mar 29 12:56:50 SG-30.local.lan stenographer[4450]: 2023/03/29 12:56:50 Stenotype stopped after 2m20.294107024s: st
Mar 29 12:56:50 SG-30.local.lan stenographer[4450]: 2023/03/29 12:56:50 Deleted stale output file "/data/stenograph
```

If there are any issues with launching the service, it will be written into this log.  
Now, we need to verify packets are being written to disk.  
`ping 192.168.2.20`  
`sudo stenoread 'host 192.168.2.20' -nn`  
```
[admin@SG-30 stenographer]$ sudo stenoread 'host 192.168.2.20' -nn
Running stenographer query 'host 192.168.2.20', piping to 'tcpdump -nn'
reading from file /dev/stdin, link-type EN10MB (Ethernet)
13:03:33.924937 IP 172.16.30.100 > 192.168.2.20: ICMP echo request, id 4560, seq 1, length 64
13:03:33.925598 IP 192.168.2.20 > 172.16.30.100: ICMP echo reply, id 4560, seq 1, length 64
```
If you want to specify directionality, you need to "pipe" the stenoread command to tcpdump.  
`sudo stenoread 'host 192.168.2.20' -nn 'src host 192.168.2.20'`  
```
[admin@SG-30 stenographer]$ sudo stenoread 'host 192.168.2.20' -nn 'src host 192.168.2.20'
Running stenographer query 'host 192.168.2.20', piping to 'tcpdump -nn src host 192.168.2.20'
reading from file /dev/stdin, link-type EN10MB (Ethernet)
13:03:33.925598 IP 192.168.2.20 > 172.16.30.100: ICMP echo reply, id 4560, seq 1, length 64
13:03:34.925730 IP 192.168.2.20 > 172.16.30.100: ICMP echo reply, id 4560, seq 2, length 64
13:03:35.925686 IP 192.168.2.20 > 172.16.30.100: ICMP echo reply, id 4560, seq 3, length 64
13:03:36.925755 IP 192.168.2.20 > 172.16.30.100: ICMP echo reply, id 4560, seq 4, length 64
```
