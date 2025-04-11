## Grafana Connecting to Wazuh

The following documentation discribes the configuration need to  connect Grafana Data Source to Wazuh-Indexer for trending data.

This isnatll discribed below was installed with  Wazuh Script so it has all defualt settings. Some configuration are needed to connect remotely to the Wazuh-Indexer.

1. change the Cluster name.
2. Change the Node name.
3. Add ip address to network host

Example:
```
network.host: "127.0.0.1,192.168.0.207"
node.name: "node-1"
cluster.initial_master_nodes:
- "node-2"
cluster.name: "wazuh-cluster"
```
