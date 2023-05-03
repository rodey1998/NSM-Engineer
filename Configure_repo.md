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

# Creating local repo - offline updates

`sudo yum install yum-utils -y` -y will skip the agreement
`ll /repo` to view local repos
`sudo reposync -l --repoid=extras --download_path=/repo/local-extras` sync centos extras repo creating a local version and naming it local-extras
verify all packages are there

`sudo yum install createrepo` converts the rpm files into usable db files by yum

`cd /repo`
`sudo createrepo local-extras`
`ll /repo/local-extras/` should see extras and repodata

## create local nginx 

`sudo vi /etc/nginx/conf.d/packages.conf`

```
server {
  listen 8008;
  location / {
    root /repo;
    autoindex on;
    index index.html index.htm;
  }
}
```
add firewall port through on port 8008
reload
list-all to verify
have to restart nginx service
`sudo systemctl restart nginx`
`^restart^status` replaces restart with status, should see greenage  
view listeners
should be able to get to port 8008 in browser

---

# Secure Repo and Fileshare

`mkdir ~/certs`
`cd ~/certs`
`openssl genrsa -des3 -out localCA.key 2048`
Enter pass phrase for localCA.key: training
`openssl req -x509 -new -nodes -key localCA.key -sha256 -days 1095 -out localCA.crt`

> Info below is optional
```
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:CA
Locality Name (eg, city) [Default City]:Mountain View
Organization Name (eg, company) [Default Company Ltd]:Elastic
Organizational Unit Name (eg, section) []:Security
Common Name (eg, your name or your server's hostname) []:Instructor
Email Address []:

```

`openssl genrsa -out repo.key 2048`
`openssl req -new -key repo.key -out repo.csr`
enter in the info, no chall password


`sudo vi ~/certs/repo.ext1
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = repo
IP.1 = 10.81.139.10
```
`openssl x509 -req -in repo.csr -CA localCA.crt -CAkey localCA.key -CAcreateserial -out repo.crt -days 365 -sha256 -extfile repo.ext`

`sudo mv repo.crt /etc/nginx`
`sudo mv repo.key /etc/nginx`

`ll /etc/nginx/` to verify keys have moved
`sudo mv repo.key /etc/nginx`
`sudo curl -LO http://repo:8000/nginx/proxy.conf`
> should see proxy.conf. Look in here if you want

## turn off port 80
`sudo vi /etc/nginx/nginx.conf`

`:set nu` gives line numbers in vi
  go to server section line 39-41. comment out using #

## modiy fileshare package configs to list locally

`sudo vi /etc/nginx/conf.d/packages.conf`

```
server {
  listen 127.0.0.1:8008;
  location / {
    root /repo;
    autoindex on;
    index index.html index.htm;
  }
}
```
`sudo vi /etc/nginx/conf.d/fileshare.conf`

```
server {

  listen 127.0.0.1:8000;

  location / {

    root /usr/share/nginx/fileshare;

    autoindex on;

    index index.html index.htm;

  }

}
```

## Add ports 80 and 443 to firewall

`sudo firewall-cmd --add-port={80,443}/tcp --permanent`  
`sudo firewall-cmd --remove-port={8000,8008}/tcp --permanent`  
`sudo firewall-cmd --remove-port={8000,8008}/tcp --permanent`  
`sudo firewall-cmd --list-all`

## restart nginx

`sudo systemctl restart nginx`
`^restart^status`

`ss -lnt`

See configure sensor for further instruction












