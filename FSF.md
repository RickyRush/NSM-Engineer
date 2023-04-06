### Configuring FSF
---
First things first, install fsf  
`sudo yum install fsf`    

And we can begin server configurations.  
`sudo vi /opt/fsf/fsf-server/conf/config.py`  
```
                                      + + +
 9 SCANNER_CONFIG = { 'LOG_PATH' : '/data/fsf/logs',
10                    'YARA_PATH' : '/var/lib/yara-rules/rules.yara',
11                    'PID_PATH' : '/run/fsf/fsf.pid',
12                    'EXPORT_PATH' : '/data/fsf/archive',
13                    'TIMEOUT' : 60,
14                    'MAX_DEPTH' : 10,
15                    'ACTIVE_LOGGING_MODULES' : ['scan_log', 'rockout'],
16                    }
17
18 SERVER_CONFIG = { 'IP_ADDRESS' : "172.16.30.100",
19                   'PORT' : 5800 }
```

Now we need to create the folders mentioned in the file.  
`sudo mkdir /data/fsf/{logs,archive}`  
`sudo chown -R fsf: /data/fsf`  
`sudo chmod -R 755 /data/fsf`    

Now we can look at the fsf client. We need to update the IP to the sensor IP.   
`sudo vi /opt/fsf/fsf-client/conf/config.py`  
```
1 #!/usr/bin/env python
2 #
3 # Basic configuration attributes for scanner client.
4 #
5
6 # 'IP Address' is a list. It can contain one element, or more.
7 # If you put multiple FSF servers in, the one your client chooses will
8 # be done at random. A rudimentary way to distribute tasks.
9 SERVER_CONFIG = { 'IP_ADDRESS' : ['172.16.30.100',],
10                   'PORT' : 5800 }
11
12 # Full path to debug file if run with --suppress-report
13 CLIENT_CONFIG = { 'LOG_FILE' : '/tmp/client_dbg.log' }
```

Create another systemd service for fsf.  
`sudo vi /etc/systemd/system/fsf.service`  
```
 1 [Unit]
 2 Description=File Scanning Framework (FSF-Server) Service
 3 After=network.target
 4
 5 [Service]
 6 Type=forking
 7 User=fsf
 8 Group=fsf
 9 WorkingDirectory=/
10 PIDFile=/run/fsf/fsf.pid
11 PermissionsStartOnly=true
12 ExecStartPre=/bin/mkdir -p /run/fsf
13 ExecStartPre=/bin/chown -R fsf:fsf /run/fsf
14 ExecStart=/opt/fsf/fsf-server/main.py start
15 ExecStop=/opt/fsf/fsf-server/main.py stop
16 ExecReload=/opt/fsf/fsf-server/main.py restart
17
18 [Install]
19 WantedBy=multi-user.target
```

Start the service and verify it works.  
`sudo systemctl daemon-reload`  
`sudo systemctl start fsf.service`  
`sudo systemctl enable fsf.service`  
`sudo systemctl status fsf.service`  


We can test fsf by making a dummy file and passing it to fsf.  
`echo "bababooey > ~/bababooey.txt"`  
`/opt/fsf/fsf-client/fsf_client.py --full ~/bababooey.txt`  
`cat /data/fsf/logs/rockout.log`  
We should see the created file in this log.  

Now we will create the directory the extracted files will go to.  
`sudo mkdir /data/zeek/extract_files`  
`sudo chown -R zeek: /data/zeek/`    
`sudo chmod 755 /data/zeek/extract_files`  

Now we write the script that will actually extract the files.  
`sudo curl -LO 192.168.2.20:8080/zeek_scripts/extract-files.zeek`  
`sudo vi /usr/share/zeek/site/scripts/extract-files.zeek`  
```
@load /usr/share/zeek/policy/frameworks/files/extract-all-files.zeek
redef FileExtract::prefix = "/data/zeek/extract_files/";
redef FileExtract::default_limit = 1048576000;
```

Now we need to add this script to the bottom of the local.zeek file.  
`sudo vi /usr/share/zeek/site/local.zeek`  
```
@load /usr/share/zeek/site/scripts/afpacket.zeek
@load /usr/share/zeek/site/scripts/extension.zeek
@load /usr/share/zeek/site/scripts/kafka.zeek
@load /usr/share/zeek/site/scripts/extract-files.zeek
```
It should now look like this. We need to reload the service since we changed a configuration. Hopefully no errors :D  

`sudo systemctl restart zeek`   

Now we begin final verification.  
`sudo curl 192.168.2.20:8080/putty.exe`  
```
[admin@SG-30 ~]$ ll /data/zeek/extract_files
total 1612
-rw-r--r--. 1 zeek zeek 1647912 Mar 30 19:31 extract-1680204688.362197-HTTP-F6AdXh1XLthVg4KJHb
```

We can see the file was successfully extracted by fsf.

We now download another zeek script for fsf. Ensure line 10 matches our local extracted files directory.  
`cd /usr/share/zeek/site/scripts/`  
`sudo curl -LO 192.168.2.20:8080/zeek_scripts/fsf.zeek`  
```
1 # Put this in /usr/share/zeek/site/scripts
2 # And add a @load statement to the /usr/share/zeek/site/local.zeek script
3
4 event file_state_remove(f: fa_file)
5     {
6         if ( f$info?$extracted )
7         {
8                # invoke the FSF-CLIENT and add the source metadata of ROCK01 (sensorID), we're suppressing the returned report
9                # becuase we don't need that
10                local file_path = "/data/zeek/extracted_files/";
11                local script_path = "/opt/fsf/fsf-client/fsf_client.py";
12                local scan_cmd = fmt("python %s --suppress-report --archive none  %s%s", script_path, file_path, f$info$extracted);
13                system(scan_cmd);
14          }
15 }
```
Change line 10 to match our extracted_files directory.  

And again, add this to the local.zeek.  
`sudo vi /usr/share/zeek/site/local.zeek`  
```
@load /usr/share/zeek/site/scripts/afpacket.zeek
@load /usr/share/zeek/site/scripts/extension.zeek
@load /usr/share/zeek/site/scripts/kafka.zeek
@load /usr/share/zeek/site/scripts/extract_files.zeek
@load /usr/share/zeek/site/scripts/fsf.zeek
```
Like mechagodzilla, this part of the file just keeps getting bigger.  
`sudo systemctl restart zeek`  
