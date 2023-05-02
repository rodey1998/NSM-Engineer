ssh into repo  
host fileshare using nginx  
`sudo yum install nginx` - install nginx
`sudo unzip ~/all-class-files.zip -d /usr/share/nginx`  - unzip packages
`sudo mv /usr/share/nginx/all-class-files /usr/share/nginx/fileshare` - move files
`sudo mv ~/emerging.rules.tar.gz /usr/share/nginx/fileshare` - move rule files 
`cd /usr/share/nginx/fileshare/`
`sudo vi /etc/nginx/conf.d/fileshare.conf`

```
server {
  listen 8000;
  location / {
    root /usr/share/nginx/fileshare;
    autoindex on;
    index index.html index.htm;
  }
}

```
`sudo firewall-cmd --add-port=8000/tcp --permanent`
`sudo firewall-cmd --reload`
`sudo firewall-cmd --list-all`
`sudo systemctl enable --now nginx`
`sudo systemctl status nginx`
`ss -lnt` - port number and listeners, look for port 8000

open web browser "repo:8000" should see files

