## Nginx Reverse Proxy Setup

This documentation shows how to configure Nginx reverse proxy with Let's Encrypt which connects to Wazuh.

If your setup is not in a cluster (single node). The default settings for opnesearch/opensearch-dashboard configuration file as shown below.

### Wazuh Indexer Configuration

Wazuh Indexer should be set as shown below.

Default:

```
network.host: "127.0.0.1"
```

### Wazuh Dashboard Configuration

Change the port to something else besides 443. In this example its 4443.

```
server.port: 4443 
```

Completed settings as shown below. 

```
server.host: 0.0.0.0
opensearch.hosts: https://127.0.0.1:9200
server.port: 4443
```

Restart wazuh services.

```
systemctl restart wazuh-manager wazuh-dashboard 
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
    server_name wazuh.domain.com;

    location / {
        proxy_pass http://127.0.0.1:4443/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Check the default site settings.

```
nginx -t
```

Reload the configuration file.

```
systemctl reload nginx
```

Install Certbot for lets encrypt.

```
sudo apt install certbot python3-certbot-nginx
```

Run Certbot and fill in all the requirements needed.

```
sudo certbot --nginx
```

Once completed restart Nginx service.

```
systemctl restart nginx
```

Access the Wazuh-Dashboard URL.

```
https://wazuh.domain.com
```




