# Wazuh-and-Zitadel-Setup
This documentation will show the basic Installation, configuration and connection for installing  Wazuh components, certificates and connection to Zitadel.


## Prerequisite:
* Ubuntu-22.0.4
* Updates/Upgrades Completed
* Network Configured (Static address and DNS)
* Date/Time is set
* Zitadel-v2.47.0


## Wazuh installation

I found it easier to run Wazuh install script and then modify configuration files and certs later.

Create custom Diretory.

```
mkdir /tmp/wazuh
```

Change directory.

```
cd /tmp/wazuh
```
 
To install all components needed download and run Wazuh custom script.

``` 
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
All servcie at this time should be running and enabled.
Once completed and no issues then  all three services need to be stopped before continuing.

```
systemctl stop wazuh-indexer wazuh-dashboards wazuh-manager
```
Download script and configuration file.
```
curl -sO https://packages.wazuh.com/4.7/wazuh-certs-tool.sh
```
```
curl -sO https://packages.wazuh.com/4.7/config.yml
```

Edit ./config.yml and replace the node names and IP values. In this example I replaced the with local ip address.

```
nodes:
  # Wazuh indexer nodes
  indexer:
    - name: node-1
      ip: "192.168.1.100"
    #- name: node-2
    #  ip: "<indexer-node-ip>"
    #- name: node-3
    #  ip: "<indexer-node-ip>"

  # Wazuh server nodes
  # If there is more than one Wazuh server
  # node, each one must have a node_type
  server:
    - name: server-1
      ip: "192.168.1.100"
    #  node_type: master
    #- name: wazuh-2
    #  ip: "<wazuh-manager-ip>"
    #  node_type: worker
    #- name: wazuh-3
    #  ip: "<wazuh-manager-ip>"
    #  node_type: worker

  # Wazuh dashboard nodes
  dashboard:
    - name: dashboard
      ip: "192.168.1.100"
~
```
Run ./wazuh-certs-tool.sh to create the certificates
```
bash ./wazuh-certs-tool.sh -A
```
```
tar -cvf ./wazuh-certificates.tar -C ./wazuh-certificates/ .
```
```
rm -rf ./wazuh-certificates
```
Ensure dependencies are installed.

```
apt-get install debconf adduser procps
```
```
apt-get install gnupg apt-transport-https
```

Edit opensearch.yml

```
vi /etc/wazuh-indexer/opensearch.yml
```

* network.host: Sets the address of this node for both HTTP and transport traffic. 
* node.name: Name of the Wazuh indexer node as defined in the config.yml
* cluster.initial_master_nodes: List of the names of the master-eligible nodes. 

### Create certificates 

This section will create a directory for certificates and use the certificates from the TAR file created in the above steps. Then move these certificates into Wazuh-indexer cert directory.

NOTE: Certs directory should have been made when running the install script. If not it will need to be made.

```
mkdir /etc/wazuh-indexer/certs
```
```
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-indexer/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./admin.pem ./admin-key.pem ./root-ca.pem
```
```
mv -n /etc/wazuh-indexer/certs/$NODE_NAME.pem /etc/wazuh-indexer/certs/indexer.pem
```
```
mv -n /etc/wazuh-indexer/certs/$NODE_NAME-key.pem /etc/wazuh-indexer/certs/indexer-key.pem
```
Set permissions.

```
chmod 500 /etc/wazuh-indexer/certs
```
```
chmod 400 /etc/wazuh-indexer/certs/*
```
```
chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs
```
Adjust Wazuh-indexer configuration file. The default certificates names need to be renamed to match the ones inthe certs directory.
Example:

```
plugins.security.ssl.http.pemcert_filepath: /etc/wazuh-indexer/certs/indexer.pem
plugins.security.ssl.http.pemkey_filepath: /etc/wazuh-indexer/certs/indexer-key.pem
plugins.security.ssl.http.pemtrustedcas_filepath: /etc/wazuh-indexer/certs/root-ca.pem
plugins.security.ssl.transport.pemcert_filepath: /etc/wazuh-indexer/certs/indexer.pem
plugins.security.ssl.transport.pemkey_filepath: /etc/wazuh-indexer/certs/indexer-key.pem
plugins.security.ssl.transport.pemtrustedcas_filepath: /etc/wazuh-indexer/certs/root-ca.pem
```
Wazuh-indexer Services.

```
systemctl daemon-reload
```
```
systemctl enable wazuh-indexer
```
```
systemctl start wazuh-indexer
```

Check status.

```
systemctl status wazuh-indexer
```

Run the indexer-security-init.sh script to load the new certificates information and start the single-node.

Change directory.

```
cd /usr/share/wazuh-indexer/bin
```
Execute script.

```
./indexer-security-init.sh
```

### Wazuh-dashboard

Ensure all dependencies are installed.

```
apt-get install debhelper tar curl libcap2-bin
```

Edit the Wazuh-Dashboard file.

```
vi /etc/wazuh-dashboard/opensearch_dashboards.yml
```

* server.host: This setting specifies the host of the Wazuh dashboard server. To allow remote users to connect, set the value to the IP address or DNS name of the Wazuh dashboard server
* opensearch.hosts: The URLs of the Wazuh indexer instances to use for all your queries. 


### Deploy Certificates for Wazuh-dashboard.
This section will create a directory from the TAR file created in the above steps and move them into Wazuh-dashboards cert directory.
Certs directory should have been made when running the install script earlier. If not it will need to be made.

```
mkdir /etc/wazuh-dashboard/certs
```

Change directory.

```
cd /tmp/wazuh/
```

Deploy Certificates.

```
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-dashboard/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
```
```
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME.pem /etc/wazuh-dashboard/certs/dashboard.pem
```
```
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME-key.pem /etc/wazuh-dashboard/certs/dashboard-key.pem
```

Set permissions.

```
chmod 500 /etc/wazuh-dashboard/certs
```
```
chmod 400 /etc/wazuh-dashboard/certs/*
```
```
chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs
```

Adjust Wazuh-dashboard configuration file. The default certificates need to be renamed.

Example:

```
server.ssl.key: "/etc/wazuh-dashboard/certs/dashboard-key.pem"
server.ssl.certificate: "/etc/wazuh-dashboard/certs/dashboard.pem"
opensearch.ssl.certificateAuthorities: ["/etc/wazuh-dashboard/certs/root-ca.pem"]
```

Wazuh-Dashboard Services

```
systemctl daemon-reload
```
```
systemctl enable wazuh-dashboard
```
```
systemctl start wazuh-dashboard
```
Login with FQDN ```https://wazuh.domain.com```
