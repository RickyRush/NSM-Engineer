### Setting up the local Repo
---
Wipe files in current directory
1. `cd /etc/yum.repos.d`
2. `sudo rm -rf C*`

Create our own local repository by curling down file from instructor repo
1. `sudo curl -LO http://192.168.2.20:8080/local.repo`
2. `sudo yum makecache`
3. `sudo yum update`
4. `sudo systemctl reboot`'
Verify CentOS files did not return on boot:  
5. `ssh admin@172.16.30.100`  
6. `ll /etc/yum.repos.d`
6. `sudo yum makecache --disablerepo="*" --enablerepo=local*`
7. `sudo yum update --disablerepo="*" --enablerepo=local*`
