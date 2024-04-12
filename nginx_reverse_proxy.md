## Nginx Reverse Proxy Setup

If not in a cluster (single node) this should be a default configurtion for opnesearch/opensearch-dashboard configuration file.


### Wazuh Indexer Configuration

Wazuh Indexer should be set as shown below.

```
network.host: "127.0.0.1"
```

### Wazuh Dashboard Configuration

Change the port to  something else besides 443. In this example it 4443
```
server.port: 4443 
```

Settings as shown below. 
```
server.host: 0.0.0.0
opensearch.hosts: https://127.0.0.1:9200
server.port: 4443
```
Restart serives
```
systemctl restart wazuh-inder wazuh-dashboard
```

###  Nginx

Install nginx package.

```
apt install nginx
```

Edit nginx default site file.

```
vi /etc/nginx/sites-available/default
```

Configure the default site file as shown below.

```
server {
    listen 80;
    server_name grafana.domain.com;

    location / {
        proxy_pass http://127.0.0.1:4443/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Install Certbot for lets encrypt.

```
sudo apt install certbot python3-certbot-nginx
```

Run Certbot and fill in all the requirements needed.

```
sudo certbot --nginx
```


