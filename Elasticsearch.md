### Elasticsearch
---

Node - single instance of Elasticsearch  
Cluster - one or more nodes  
Roles - role that node serves. (data, master, ingest)  
Document - basic unit of information that can be indexed   
Index - logical namespace that refers to a collection of documents  
Shard - individual piece of an index  
Indexes are "sharded out" to primary and replicas. The replicas serve as redundancy in the event the primaries become unavailable.  

comment searches for words, comment.keyword searches for exact match  

Now we can move on to the install!  

---

`sudo yum install elasticsearch-7.8.1`  
`sudo chown -R elasticsearch: /data/elasticsearch`  
`sudo -s`  
`sudo cd /etc/elasticsearch`  
`vi /etc/elasticsearch/elasticsearch.yml`  
```
13 # ---------------------------------- Cluster -----------------------------------
14 #
15 # Use a descriptive name for your cluster:
16 #
17 cluster.name: SG-30-cluster
18 #
19 # ------------------------------------ Node ------------------------------------
20 #
21 # Use a descriptive name for the node:
22 #
23 node.name: SG-30-node
29 # ----------------------------------- Paths ------------------------------------
30 #
31 # Path to directory where to store the data (separate multiple locations by comma):
32 #
33 path.data: /data/elasticsearch
39 # ----------------------------------- Memory -----------------------------------
40 #
41 # Lock the memory on startup:
42 #
43 bootstrap.memory_lock: true
51 # ---------------------------------- Network -----------------------------------
52 #
53 # Set the bind address to a specific IP (IPv4 or IPv6):
54 #
55 network.host: _eno1_
56 #
57 # Set a custom port for HTTP:
58 #
59 http.port: 9200
63 # --------------------------------- Discovery ----------------------------------
64 #
65 # Pass an initial list of hosts to perform discovery when this node is started:
66 # The default list of hosts is ["127.0.0.1", "[::1]"]
67 #
68 discovery.type: single-node
69 #discovery.seed_hosts: [172.16.30.100]
70 #
71 # Bootstrap the cluster using an initial set of master-eligible nodes:
72 #
73 #cluster.initial_master_nodes: [172.16.30.100]
```
Ensure that lines 17, 23, 33, 43, 55, 59 are uncommented and say the above. Add in line 68.   



`sudo mkdir -p /usr/lib/systemd/system/elasticsearch.service.d`  
`sudo vi /usr/lib/systemd/system/override.conf`  
```
[Service]
LimitMEMLOCK=infinity
```
`sudo chmod 755 /usr/lib/systemd/system/elasticsearch.service.d`  
`sudo chmod 644 /usr/lib/systemd/system/override.conf`  
`sudo systemctl daemon-reload`  
`sudo systemctl start elasticsearch`  
`sudo systemctl enable elasticsearch`  
`sudo systemctl status elasticsearch`  
`sudo vi /etc/elasticsearch/jvm.options`  
```
17 ################################################################
18
19 # Xms represents the initial size of total heap space
20 # Xmx represents the maximum size of total heap space
21
22 -Xms4g
23 -Xmx4g
24
25 ################################################################
```
 **These must be the same**  
`sudo systemctl restart elasticsearch`  
`sudo firewall-cmd --addport={9200,9300}/tcp --permanent`  
`sudo firewall-cmd --reload`  

 Verify connectivity over port 9200  
```
[root@SG-30 elasticsearch]# ss -lnt
State       Recv-Q Send-Q                                                         Local Address:Port                                                                        Peer Address:Port               
LISTEN      0      128                                                                172.16.30.100:9200                                                                                   *:*                                                                                                 *:*                  
LISTEN      0      128                                                                172.16.30.100:9300                                                                                   *:*           
```

Verify the nodes are working properly.  
```
[root@SG-30 elasticsearch]# curl localhost:9200
{
  "name" : "SG-30",
  "cluster_name" : "SG-30-node",
  "cluster_uuid" : "_na_",
  "version" : {
    "number" : "7.8.1",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "b5ca9c58fb664ca8bf9e4057fc229b3396bf3a89",
    "build_date" : "2020-07-21T16:40:44.668009Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
`curl localhost:9200/_cat/nodes?v`  
