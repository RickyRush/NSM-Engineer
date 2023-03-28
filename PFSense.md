### PFSense
---
#### How to install PFSense:
1. Press F11 to enter boot configuration
2. Hit enter on UEFI: USB DISK 2.0 PMAP(Fifth selection)
3. Hit Accept
4. Install PFSense
5. Default Keymap
6. "Auto (UFS) with UEFI"
7. When install is finished, select 'no'
8. Reboot
9. Remove Flash Drive
10. Select option 1
11. Should vLAN be set up? NO
12. Enter WAN interface > em0
13. Enter LAN interface > em1
14. Enter Optional 1 interface > leave blank, hit Enter
15. Do you want to proceed? Y
16. Enter an option > 2 (Set interface IP address)
17. Select interface > 2 (LAN)
18. Enter new IP > 172.16.30.1
19. Enter new subnet bit count > 24
20. Enter the same IP > 172.16.30.1
21. Blank IP for IPv6 (ew)
22. Enable DHCP? Yes
23. Start address for IPv4 client range > 172.16.30.100
24. End address for IPv4 client range > 172.16.30.254
25. Revert to HTTP webConfig protocol? Yes
26. Plug laptop into PFSense LAN port over ethernet
27. Manually configure laptop IP to be within PFSense subnet
28. Connect to PFSense over http > 172.16.30.1
29. Login Credentials: admin / pfsense
30. Hostname : SG-3
31. Domain: home.arpa
32. Primary DNS: 192.168.2.1
33. Override DNS : checked
34. next, next
35. WAN : DHCP
36. Uncheck "Block RFC, Block bogon networks"
37. LAN IP address : 172.16.30.1/24
38. update password / next to skip
39. reload
40. Congratulations! PFSense is now configured!
---
#### Configure Firewall
1. Select Firewalls
1. Firewalls > WAN
2. Select add
3. Action > pass
4. Interface > WAN
5. Address family > IPv4
6. Protocol > any
7. source network > 192.168.2.0/24 (perched-wifi)
8. Save & Apply changes
---
#### Configure DNS
1. Select DNS
1. Uncheck 'enable DNS forwarder'
2. Save & Apply Changes
---
#### Diagnostics
1. Select Diagnostics
1. Halt system
2. Confirm
3. Power on the system
