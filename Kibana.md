### Kibana
---

`sudo yum install kibana-7.8.1`  

We only have two changes to make to the Kibana yml.  
`sudo vi /etc/kibana/kibana/yml`  
```
 7 server.host: "172.16.30.100"
28 elasticsearch.hosts: ["http://172.16.30.100:9200"]
```
`sudo firewall-cmd --addport-port=5601/tcp --permanent`  
`sudo firewall-cmd --reload`  
```
[root@SG-30 elasticsearch]# ss -lnt
State       Recv-Q Send-Q                                                         Local Address:Port                                                                        Peer Address:Port              
LISTEN      0      128                                                            172.16.30.100:5601                                                                                   *:*     
```
We can now browse to Kibana!  

We're now going to curl down a .tar file.  
`cd ~`  
`sudo curl -LO 192.168.2.20:8080/ecskibana.tar.gz`  
`tar -zxvf ecskibana.tar.gz`  
`cd ecskibana`  
`vi import-index-templates.sh`  
```
#!/bin/bash

_URL=$1
ES_URL=${_URL:='http://172.16.30.100:9200'}

function is_integer() {
    [ "$1" -eq "$1" ] > /dev/null 2>&1
    return $?
}

changed=0
failed=0
ok=0

for item in *.json; do
  name=$(cat ${item} | jq -r 'keys | .[]')
  version=$(cat ${item} | jq -r ".${name}.version")

  # Verify that a version number exists in the object
  if ! [[ ${version} =~ ^[0-9]+$ ]] ; then
    echo "Could not find a version number for $(realpath ${name}.json). \
Verify a version exists at the top level and is an integer." >&2; exit 1
  fi

  existing_version='-1'
  # Check if template already exists
  if curl -sI "${ES_URL}/_template/${name}" | grep -q "200 OK"; then
    existing_version=$(
      curl -s "${ES_URL}/_template/${name}?filter_path=*.version" | \
      jq -r ".${name}.version"
    )
    if ! is_integer ${existing_version}; then
      existing_version=-1
    fi
  fi

  if [ "${version}" -gt "${existing_version}" ]; then
    echo "Installing index mapping template: ${name} version ${version}" >/dev/stderr
    response=$(curl -s -XPUT "${ES_URL}/_template/${name}" -H "Content-Type: application/json" --data "$(cat ${item} | jq ".${name}")")
    result=$(echo ${response} | jq -r '.acknowledged')
    if [ "${result}" == "true" ]; then
      changed=$[$changed + 1]
    else
      failed=$[$failed + 1]
      echo "Failed putting ${name} version ${version}: \n ${response}" > /dev/null
    fi
  else
    ok=$(($ok + 1))
  fi
done

echo "OK: ${ok}"
echo "Changed: ${changed}"
echo "Failed: ${failed}"
```
`sudo ./import-index-templates.sh`  
In Kibana GUI, enter "explore on my own", and navigate to dev tools.  
`GET _cat/templates/ecs*`  
`GET _cat/templates?v&s=name`  
Here we can verify the templates we just loaded by running the above query. We won't see any data in Kibana yet because Logstash is not installed.  

---
After finishing installing the remaining services, we need to create an index pattern. This is done via Kibana GUI (management > stack management > index patterns > create index pattern). Index pattern name `ecs-suricata-*` Select @timestamp. Select create index and return to index patterns.    

We need to create the following indexes:  
`ecs-suricata-*`  
`ecs-zeek-*`  
`fsf-*`  
