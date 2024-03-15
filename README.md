# jenkins-nginx-as-reverse-proxy-ssl


Install jenkins:
----------------

```
- take t2.medium ubuntu latest os
- open 8080,80,443 ports

sudo -i
sudo apt update -y

sudo apt install fontconfig openjdk-17-jdk
java -version

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y 

sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

http://34.220.142.18:8080

```

nginx steps:
------------
```
sudo apt install nginx -y
nginx -v
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx

http://34.220.142.18:80
http://34.220.142.18
Welcome to nginx!

- by default nginx runs on 80 port

```


Add DNS record:
---------------
```
My Domain name is Registered in GODADDY.so,
- Create a DNS record that associates your domain name and your       server’s public IP address

- add dns A record in godaddy

A    jenkins	34.220.142.18
```

![image](https://github.com/vijay2181/jenkins-nginx-as-reverse-proxy-ssl/assets/66196388/15239bbe-07e2-4d42-b006-dafdf6086bb2)





Add Nginx configuration with SSL:
---------------------------------

![image](https://github.com/vijay2181/jenkins-nginx-as-reverse-proxy-ssl/assets/66196388/d1bcffd1-fd0d-4cc4-9557-5dd839a7a0d2)


```
/etc/nginx/sites-available

root@ip-172-31-25-76:/etc/nginx/sites-available# ls
default

- create a new configuration file for jenkins
- add subdomain name -> jenkins.vijayops.shop inside file  

sudo mv default default.backup


vi /etc/nginx/sites-available/default
----------------------------------------------------------
server {
    listen 80;
    server_name jenkins.vijayops.shop;


    location / {
      proxy_set_header        Host $host:$server_port;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;
      # Fix the "It appears that your reverse proxy set up is broken" error.
      proxy_pass          http://127.0.0.1:8080;
      proxy_read_timeout  90;
      proxy_redirect      http://127.0.0.1:8080 https://jenkins.vijayops.shop;
      # proxy_redirect http:// https://;

      # Required for new HTTP-based CLI
      proxy_http_version 1.1;
      proxy_request_buffering off;
    }
  }

----------------------------------------------------------------

nginx -t
sudo systemctl restart nginx

http://http://jenkins.vijayops.shop       ---> check in browser

- so you hit http://jenkins.vijayops.shop which is on port 80, it is redirecting to jenkins server

```


Change Jenkins bind address:
----------------------------
```
By default Jenkins listens on all network interfaces. But we need to disable it because we are using Nginx as a reverse proxy and there is no reason for Jenkins to be exposed to other network interfaces.
We can change this by editing /etc/default/jenkins
sudo vi /etc/default/jenkins
Locate the line starting with JENKINS_ARGS (It’s usually the last line) and append
--httpListenAddress=127.0.0.1

JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=$HTTP_PORT --httpListenAddress=127.0.0.1"

sudo systemctl restart jenkins
sudo systemctl status jenkins

```

Step-Install SSL Certificates by using Letsencrypt:
-----------------------------------------------------
```
Install Certbot:-
Certbot is a letsencrypt client, we need to download.certbot can automatically configure NGINX for SSL/TLS.The Certbot packages on your system come with a cron job or systemd timer that will renew your certificates automatically before they expire.

sudo apt-get update
sudo apt-get install python3-certbot-nginx

sudo certbot --nginx -d jenkins.vijayops.shop
or
sudo certbot --nginx -n -d jenkins.vijayops.shop --email devopsvijay6@gmail.com --agree-tos --redirect --hsts

Requesting a certificate for jenkins.vijayops.shop
Successfully received certificate.
Deploying certificate
Successfully deployed certificate for jenkins.vijayops.shop to /etc/nginx/sites-enabled/default
```

How Let’s Encrypt Works:-
-------------------------
```
Before issuing a certificate, Let’s Encrypt validates ownership of your domain. The Let’s Encrypt client, running on your host, creates a temporary file (a token) with the required information in it. The Let’s Encrypt validation server then makes an HTTP request to retrieve the file and validates the token, which verifies that the DNS record for your domain resolves to the server running the Let’s Encrypt client.
- why 80,443 openend ?
- Ensure that your firewall allows incoming connections on port 80 (HTTP) and 443 (HTTPS)
- Certbot often utilizes port 80 for the HTTP-01 challenge method, which is a common method for domain validation when obtaining SSL/TLS certificates from Let's Encrypt.
- For Certbot to function properly and obtain SSL certificates via the HTTP-01 challenge method, you need to ensure that port 80 is open and that your web server can serve files from the specified directory over HTTP during the certificate issuance process.

Note:-
certbot  can automatically configure NGINX for SSL/TLS

You can open sudo vi  /etc/nginx/sites-available/default  file 
and check ssl certificates are automatically configured by Nginx

-----------------------------------------

  listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/jenkins.vijayops.shop/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/jenkins.vijayops.shop/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

-----------------------------------------

https://jenkins.vijayops.shop
```
