### Configuring the Interface to Promiscuous Mode
---
Interface 'enp5s0' on sensor will be our promiscuous port.

Disable NIC offloading to optimize zeek  
`sudo ethtool -k enp5s0`
```
Features for enp5s0:
rx-checksumming: on
tx-checksumming: on
	tx-checksum-ipv4: off [fixed]
	tx-checksum-ip-generic: on
	tx-checksum-ipv6: off [fixed]
	tx-checksum-fcoe-crc: off [fixed]
	tx-checksum-sctp: on
                                              + + +
```
We need to disable checksumming  
`curl -LO http://192.168.2.20:8080/interface_prep.sh`
```
#!/bin/bash
  for i in rx tx sg tso ufo gso gro lro rxvlan txvlan
    do
      /usr/sbin/ethtool -K $1 $i off
  done

  /usr/sbin/ethtool -N $1 rx-flow-hash udp4 sdfn
  /usr/sbin/ethtool -N $1 rx-flow-hash udp6 sdfn
  /usr/sbin/ethtool -n $1 rx-flow-hash udp6
  /usr/sbin/ethtool -n $1 rx-flow-hash udp4
  /usr/sbin/ethtool -C $1 rx-usecs 10
  /usr/sbin/ethtool -C $1 adaptive-rx off
  /usr/sbin/ethtool -G $1 rx 4096

  /usr/sbin/ip link set dev $1 promisc on
```
Ensure script is executable  
`sudo chmod +x interface_prep.sh`  
Run the script against the interface  
`sudo ./interface_prep.sh enp5s0`  
Verify changes took by running  
`sudo ethtool -k enp5s0`

---

Acquire ifup-local script to fix boot issues within CentOS

`cd /sbin`  
`sudo curl -LO http://192.168.2.20:8080/ifup-local`

```
#!/bin/bash
if [[ "$1" == "enp5s0" ]]
then
  for i in rx tx sg tso ufo gso gro lro rxvlan txvlan
    do
      /usr/sbin/ethtool -K $1 $i off
  done

  /usr/sbin/ethtool -N $1 rx-flow-hash udp4 sdfn
  /usr/sbin/ethtool -N $1 rx-flow-hash udp6 sdfn
  /usr/sbin/ethtool -n $1 rx-flow-hash udp6
  /usr/sbin/ethtool -n $1 rx-flow-hash udp4
  /usr/sbin/ethtool -C $1 rx-usecs 10
  /usr/sbin/ethtool -C $1 adaptive-rx off
  /usr/sbin/ethtool -G $1 rx 4096

  /usr/sbin/ip link set dev $1 promisc on

fi
```

Ensure script is executable  
`sudo chmod 755 ifup-local`  

Configure within /etc/sysconfig/network-scripts  
`cd /etc/sysconfig/network-scripts`  
`sudo vi ifup`

```
                                              + + +
if [ -x /sbin/ifup-local ]; then
        /sbin/ifup-local ${DEVICE}
fi
```
Add  entry to the bottom of the file (94-96)

Create new network script  
`sudo vi ifcfg-enp5s0`  

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=none
IPV4_FAILURE_FATAL=no
IPV6INIT=no
IPV6_AUTOCONF=no
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp5s0
UUID=5b61f301-2620-487f-8573-2614f63b1d43
DEVICE=enp5s0
ONBOOT=yes
NM_CONTROLLED=no
```
Set BOOTPROTO to **none**  
Set DEFROUTE to **none**  
Set all IPv6 options to **no**  
Set ONBOOT to **yes**  
Add **NM_CONTROLLED=no**  
Ensure NAME and DEVICE = **enp5s0**

Reboot the sensor!  
`sudo systemctl reboot`  

Verify checksum changes persisted through boot  
`sudo ethtool -k enp5s0`  
`ip a`

---

### Configuring the In-Line TAP
---
We will be inserting the TAP between the Sensor and the PFSense.  

Insert Ethernet from MONITOR port on TAP to **PROMISC (LEFT)** port on IntelNUC.  

In order to install TAP, we will need to have a network outage.
1. Unplug MGMT (LAN2) on PFSense and insert into FAR LEFT port on TAP.
2. Insert Ethernet Cable into Port 2 on TAP into LAN2 on PFSense.
3. Validate traffic ingest  
  `sudo yum install tcpdump`  
  `sudo tcpdump -nn -i enp5s0`  
  `sudo tcpdump -nn -i enp5s0 '!port 22'`

  Should see large amounts of packets!
