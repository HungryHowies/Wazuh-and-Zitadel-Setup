## Grafana Connecting to Wazuh

The following documentation discribes the configuration needed to connect Grafana Data Source to Wazuh-Indexer.

This install will discribe howto install with the Wazuh Script. Some configuration are needed to connect remotely to the Wazuh-Indexer.
Edit the following file.

```
vi /etc/wazuh-indexer/opensearch.yml
```
1. Adjust the Cluster name.
2. Adjust the Node name.
3. Adjust the network host address.

Example:

NOTE: For production the network host configurations should be a static address/fqdn.

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

The Node Name shown on Grafana dashboard.

Check ossec file.

```
cat /var/ossec/etc/ossec.conf | grep -A12 "<cluster>"
```
Ensure the key is created.

```
openssl rand -hex 16
```
Paste hex in the ossec file. This will be under cluster between  <key> </key> .

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
        <node>192.168.1.100</node>
    </nodes>
    <hidden>no</hidden>
    <disabled>no</disabled>
  </cluster>
```

Restart  Wazuh-manager service

```
systemctl restart wazuh-manager
```


### Grafana Data Sources Configuration

In this example im using OpenSearch Plugin.

Settings
HTTP URL  section
```
https://opensearch.com:9200
```
Access 

```
Server ( Default)
```
Auth

Enable Basic auth tic box. For exampel I used the default admin and password.

Enable Skip TLS Verify tic box

OpenSearch Details
Index name, set this for the indices connection.
Logs
Message field name cluster.node, this will show on the visulization section.



 
