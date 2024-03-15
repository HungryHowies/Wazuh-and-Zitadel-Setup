# Wazuh-and-Zitadel-Setup
This documentation will show the basic Installation, configuration and connection for installing  up Wazuh components/ certificates and connection to Zitadel.

### Update && Upgrade

sudo apt update


### Wazuh installation

To instal all dependencies needed.
 
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

Create custom certificates.

make a directory  

mkdir /tmp/wazuh

curl -sO https://packages.wazuh.com/4.7/wazuh-certs-tool.sh
curl -sO https://packages.wazuh.com/4.7/config.yml


Edit ./config.yml and replace the node names and IP values 

Run ./wazuh-certs-tool.sh to create the certificates

bash ./wazuh-certs-tool.sh -A

tar -cvf ./wazuh-certificates.tar -C ./wazuh-certificates/ .

rm -rf ./wazuh-certificates

apt-get install debconf adduser procps
apt-get install gnupg apt-transport-https

Edit the /etc/wazuh-indexer/opensearch.yml 
network.host: Sets the address of this node for both HTTP and transport traffic. 
node.name: Name of the Wazuh indexer node as defined in the config.yml
cluster.initial_master_nodes: List of the names of the master-eligible nodes. 

Create certs directory

mkdir /etc/wazuh-indexer/certs
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-indexer/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./admin.pem ./admin-key.pem ./root-ca.pem
mv -n /etc/wazuh-indexer/certs/$NODE_NAME.pem /etc/wazuh-indexer/certs/indexer.pem
mv -n /etc/wazuh-indexer/certs/$NODE_NAME-key.pem /etc/wazuh-indexer/certs/indexer-key.pem
chmod 500 /etc/wazuh-indexer/certs
chmod 400 /etc/wazuh-indexer/certs/*
chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs

Services

systemctl daemon-reload
systemctl enable wazuh-indexer
systemctl start wazuh-indexer
check status

systemctl status wazuh-indexer

Run the Wazuh indexer indexer-security-init.sh script  to load the new certificates information and start the single-node.

cd /usr/share/wazuh-indexer/bin

./indexer-security-init.sh

### Wazuh-dashboard

check packages are installed
apt-get install debhelper tar curl libcap2-bin 

Edit Wazuh-Dashboard file

vi /etc/wazuh-dashboard/opensearch_dashboards.yml

server.host: This setting specifies the host of the Wazuh dashboard server. To allow remote users to connect, set the value to the IP address or DNS name of the Wazuh dashboard server
opensearch.hosts: The URLs of the Wazuh indexer instances to use for all your queries. 


deploy certs

mkdir /etc/wazuh-dashboard/certs
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-dashboard/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME.pem /etc/wazuh-dashboard/certs/dashboard.pem
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME-key.pem /etc/wazuh-dashboard/certs/dashboard-key.pem
chmod 500 /etc/wazuh-dashboard/certs
chmod 400 /etc/wazuh-dashboard/certs/*
chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs

Services

systemctl daemon-reload
systemctl enable wazuh-dashboard
systemctl start wazuh-dashboard

