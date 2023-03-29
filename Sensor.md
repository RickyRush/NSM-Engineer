### Setting up the Sensor
---
#### Initial Install & Configuration
1. Insert "CentOS" USB into IntelNUC
2. Power on and spam F10 to enter the boot menu
3. Select 'UEFI: USB : General : Part 1 : OS Bootloader'
4. Select 'Test this media & Install CentOS 7'
5. Select 'English > US English'
6. Select 'Network and Host Name'
  1.  Configure eno1 as Management Port
    1. Insert Ethernet to **RIGHT port** on the back of IntelNUC > LAN1 on PFSense
    2. Turn the Ethernet of eno1 'ON'
    3. Click 'Configure'
    4. Select IPv4 Settings
    5. Method > Manual (static)
    6. Type in sensor information under 'addresses' (172.16.30.100 / 255.255.255.0 / 172.16.30.1)
    7. Select IPv6 Settings
    8. Method > Ignore
    9. Select SAVE
  2. Configure Host Name
    1. In bottom left, change HN to 'SG-30.local.lan'
    2. Apply
    3. Select 'done' in top left
7. Select 'Date and Time'
  1. Configure Time
    1. Set Region to 'Etc'
    2. Set City to 'Coordinated Universal Time'
    3. Ensure 'Network Time' is ON
    4. Select 'done' in top left
8. Ensure 'Software Selection' is set to 'Minimal Install'
9. Apply Security Policy if applicable
10. Select 'KDUMP'  
  1. Ensure kdump is disabled
  2. Select 'done' in top left
11. Select 'Installation Destination'
  1. Select both disks (black circle with white checkmark)
  2. One of these will be data partition, one will be set for OS.
  3. Select 'Automatically configure partitioning'
  4. Check 'I would like to make additional space available'
  5. Select 'done' in top left
  6. Select 'delete all'
  7. Select 'reclaim space'
12. Select 'Installation Destination' (again)
  1. Ensure both drives are selected
  2. Check 'I will configure partitioning'
  3. Select 'done' in top left
  4. Select 'click here to create them automatically'
13. Configure Partitions Manually
  1. /home : desired capacity : 1 G       (50 g)
  2. Select 'update settings'
  3. / : desired capacity : 1 G           (blank - do last)
  4. Select 'update settings'
  5. Add '/var/log' partition
      1. In bottom left, hit +
      2. Mount point : '/var/log'
      3. Desired capacity : '1 G'         (50 g)
      4. Select 'add mount point'
  6. Add '/tmp' partition
      1. In bottom left, hit +
      2. Mount point : '/tmp'
      3. Desired capacity : '1 G'         (5 g)
      4. Select 'add mount point'
  7. Add '/var' partition
      1. In bottom left, hit +
      2. Mount point : '/var'
      3. Desired capacity : '1 G'         (50 g)
      4. Select 'add mount point'
  8. Add '/var/log/audit' partition
      1. In bottom left, hit +
      2. Mount point : '/var/log/audit'
      3. Desired capacity : '1 G'         (50 g)
      4. Select 'add mount point'
  9. Add '/var/tmp' partition
      1. In bottom left, hit +
      2. Mount point : '/var/tmp'         (10 g)
      3. Desired capacity : '1 G'
      4. Select 'add mount point'
  10. Add '/data/stenographer' partition
        1. In bottom left, hit +
        2. Mount point : '/data/stenographer'
        3. Desired capacity : '1 G'       (500 g)
        4. Select 'add mount point'
  11. Add '/data/kafka' partition
        1. In bottom left, hit +
        2. Mount point : '/data/kafka'
        3. Desired capacity : '1 G'       (100 g)
        4. Select 'add mount point'
  12. Add '/data/suricata' partition
        1. In bottom left, hit +
        2. Mount point : '/data/suricata'
        3. Desired capacity : '1 G'       (25 g)
        4. Select 'add mount point'
  13. Add '/data/elasticsearch' partition
        1. In bottom left, hit +
        2. Mount point : '/data/elasticsearch'
        3. Desired capacity : '1 G'       (blank - do last)
        4. Select 'add mount point'
  14. Add '/data/zeek' partition
        1. In bottom left, hit +
        2. Mount point : '/data/zeek'
        3. Desired capacity : '1 G'       (25 g)
        4. Select 'add mount point'
  15. Add '/data/fsf' partition
        1. In bottom left, hit +
        2. Mount point : '/data/fsf'
        3. Desired capacity : '1 G'       (10 g)
        4. Select 'add mount point'
  16. Select '/'
        1. Volume group > create a new volume group
        2. Select smaller drive
        3. Name "VG_OS"
        4. Select 'save'
        5. Select 'update settings' in bottom right
  17. Retroactively apply 'VG_OS' to ALL OS files (NOT NAMED DATA)
  18. Select '/data/zeek'
        1. Volume group > create a new volume group
        2. Select larger drive
        3. Name "VG_DATA"
        4. Select 'save'
        5. Select 'update settings' in bottom right
  19. Retroactively apply 'VG_DATA' to all files named 'DATA/'
  20. We can now apply the TRUE desired size of the partitions
        1. Swap - 4 G
  21. Verify partitions are correct and select 'done'
  22. Accept changes
14. Select 'Begin Installation'
  1. Create a user, root will be disabled
      1. Check 'Make this user administrator'
      2.  admin / simple
15. Select 'reboot' and remove CentOS USB
16. log in with admin / simple
17. ip a to verify configuration
18. verify SSH connectivity from student laptop
---
#### Commands to run once connected via SSH
1. `sudo vi /etc/sysconfig/network-scripts/ifcfg-eno1`
```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="none"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="no"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="eno1"
UUID="a7975653-54e8-4fb6-9700-d427ae65ed0a"
DEVICE="eno1"
ONBOOT="yes"
IPADDR="172.16.30.100"
PREFIX="24"
GATEWAY="172.16.30.1"                                                    
```
Within the above file, change IPv6 settings to no. (lines 145,146)

2.  `sudo vi /etc/sysctl.conf`
```
sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
```
```
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
```
Add in lines 171 and 172 to the file to disable IPv6.

3. `sudo vi /etc/hosts`
```
 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
#::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```
Remove or comment line 181 from this file.

4. Ensure SELinux is enabled and reload and restart the system  
`sestatus`
`sudo sysctl -p`
