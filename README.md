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
4. Install the services (steno, suricata, zeek, fsf, kafka)

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
