## Grafana Connecting to Wazuh

The following documentation discribes the configuration need to  connect Grafana Data Source to Wazuh-Indexer for trending data.

This insatll discribed below was installed with  Wazuh Script so it has all defualt settings. Some configuration are needed to connect remotely to the Wazuh-Indexer.
Edit the following file.

```
vi /etc/wazuh-indexer/opensearch.yml
```
1. Change the Cluster name.
2. Change the Node name.
3. Add ip address to network host.

Example:

NOTE: For production the network host configurations should be a static address.

```
network.host: "0.0.0.0"
```


```
node.name: "node02"
cluster.initial_master_nodes:
- "node02"
cluster.name: "wazuh-cluster"
```

### Cluster running

This section is import in many ways especially for the Node Name shown on Grafana dashboard.

Check ossec file.

```
cat /var/ossec/etc/ossec.conf | grep -A12 "<cluster>"
```
Ensure the key is created.

```
openssl rand -hex 16
```
Paste that in ossec file under cluster between <key></key> 
```
<key>3e465e7f1b6eaa52f22b93618ced96eb</key>
```
Cluster nodes setting.

```
<node>192.168.1.100</node>
```
Enable the cluster.

Default:

```
<disabled>yes</disabled>
```
Configured as following.

```
<disabled>no</disabled>
```

Results:

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

Restart  Wazuh-manager service

```
systemctl restart wazuh-manager
```
