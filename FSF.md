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
18 SERVER_CONFIG = { 'IP_ADDRESS' : "localhost",
19                   'PORT' : 5800 }
```

Now we need to create the folders mentioned in the file.  
`sudo mkdir /data/fsf/{logs,archive}`  
`sudo chown -R fsf: /data/fsf`  
`sudo chmod -R 755 /data/fsf`    

Now we can look at the fsf client. We don't need to make any changes but this is where you'd go to update any client information.   
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
9 SERVER_CONFIG = { 'IP_ADDRESS' : ['127.0.0.1',],
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
14 ExecStartPre=/opt/fsf/fsf-server/main.py start
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
