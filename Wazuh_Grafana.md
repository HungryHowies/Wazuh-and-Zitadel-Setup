## Grafana Connecting to Wazuh

The following documentation discribes the configuration need to  connect Grafana Data Source to Wazuh-Indexer for trending data.

This isnatll discribed below was installed with  Wazuh Script so it has all defualt settings. Some configuration are needed to connect remotely to the Wazuh-Indexer.

1. change the Cluster name.
2. Change the Node name.
3. Add ip address to network host

Example:
```
network.host: "127.0.0.1,192.168.1.255"
node.name: "node-1"
cluster.initial_master_nodes:
- "node-2"
cluster.name: "wazuh-cluster"
```

### Cluster running
Check ossec file.
```
cat /var/ossec/etc/ossec.conf | grep -A12 "<cluster>"
```
Ensure you created a key.
 ```
openssl rand -hex 16
```
Paste that in ossec file under cluster between <key></key> 
```
<key>3e465e7f1b6eaa52f22b93618ced96eb</key>
```

Results

```
 <cluster>
    <name>wazuh</name>
    <node_name>node01</node_name>
    <node_type>master</node_type>
    <key>3e465e7f1b6eaa52f22b93618ced96eb</key>
    <port>1516</port>
    <bind_addr>0.0.0.0</bind_addr>
    <nodes>
        <node>node-ipaddress</node>
    </nodes>
    <hidden>no</hidden>
    <disabled>no</disabled>
  </cluster>
```
