# Configuring Kibana
1. `sudo yum install kibana`
2. `sudo mv /etc/kibana/kibana{.yml,.yml.bk}`
3. `sudo vi /etc/kibana/kibana.yml`

```
server.port: 5601
server.host: localhost
server.name: kibana
elasticsearch.hosts: ["http://elastic0:9200","http://elastic1:9200","http://elastic2:9200"]

```
4. `sudo yum install nginx`
5. `sudo vi /etc/nginx/conf.d/kibana.conf` - new file

```
server {
  listen 80;
  server_name kibana;
  proxy_max_temp_file_size 0;

  location / {
    proxy_pass http://127.0.0.1:5601/;

    proxy_redirect off;
    proxy_buffering off;

    proxy_http_version 1.1;
    proxy_set_header Connection "Keep-Alive";
    proxy_set_header Proxy-Connection "Keep-Alive";

  }

}
```

6. `sudo vi /etc/nginx/nginx.conf`
```
comment out 39-41

```
7. `sudo firewall-cmd --add-port=80/tcp --permanent`
8. `sudo firewall-cmd --reload`
9. `sudo systemctl enable nginx --now`
10. `sudo systemctl enable kibana --now`
11. `sudo systemctl status kibana`

12. Browser `http://kibana`

13. `sudo curl -LO https://repo/fileshare/kibana/ecskibana.tar.gz`
14. `tar -zxvf ecskibana.tar.gz`
15. `sudo yum install jq -y`
16. `cd ecskibana`
17. `sudo ./import-index-templates.sh http://elastic0:9200` - 6 changed, 0 failed loading index templates
18. In Kibana console dev tools `GET _cat/templates` - should see templates
