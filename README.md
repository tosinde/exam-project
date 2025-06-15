ALtSchool Second Semester Examination Project for Cloud Engineering
This project documents the process of deploying a website on an AWS EC2 instance using Nginx, DuckDNS, and securing it with a free SSL certificate from Let's Encrypt.

1. Launching EC2 Instance
a. I used Amazon Linux 2023 (free tier)
  - Instance type: t2.micro
  - Key pair: create or use existing `.pem` file
  - Security group: allow ports 22 (SSH), 80 (HTTP), and 443 (HTTPS)

b. Connecting to EC2
From my terminal:

chmod 400 your-key.pem
ssh -i your-key.pem ec2-user@<your-public-ip>

2. Installing and Configuring Nginx

sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx

I decided to paste my html and css files in the nano text editor after writing them on VsCode and placed them in:

sudo cp index.html styles.css /usr/share/nginx/html/

3. Setting Up a DuckDNS Domain

1. Go to [https://duckdns.org](https://duckdns.org)
2. Created a subdomain (e.g. `dalamuesther`)
3. Noted the DuckDNS token
4. Pointed it to my EC2 public IP - 108.129.227.136

4. Installing Certbot and Getting SSL Certificate
Install Certbot (via snap):

sudo dnf install snapd -y
sudo snap install core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

 Create the DNS Auth Script

.I wrote a bash script for the certbot and saved it as `/etc/letsencrypt/certbot-auth.sh`:

#!/bin/bash

DUCKDNS_DOMAIN="dalamuesther"
DUCKDNS_TOKEN="e419ac8f-ec5a-41a8-ba41-f198038b3917"

curl -s "https://www.duckdns.org/update?domains=$DUCKDNS_DOMAIN&token=$DUCKDNS_TOKEN&txt=$CERTBOT_VALIDATION&clear=true"

sleep 60

Making it executable:

sudo chmod +x /etc/letsencrypt/certbot-auth.sh

Run Certbot for DuckDNS DNS Challenge


sudo certbot certonly \
  --manual \
  --manual-auth-hook /etc/letsencrypt/certbot-auth.sh \
  --preferred-challenges dns \
  --manual-public-ip-logging-ok \
  --disable-hook-validation \
  -d dalamuesther.duckdns.org

5. Configure Nginx for HTTPS

Edit Nginx config:

bash
sudo nano /etc/nginx/conf.d/default.conf

Example HTTPS server block:

nginx
server {
    listen 80;
    server_name dalamuesther.duckdns.org;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name dalamuesther.duckdns.org;

    ssl_certificate /etc/letsencrypt/live/dalamuesther.duckdns.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dalamuesther.duckdns.org/privkey.pem;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

Reload Nginx:

bash
sudo systemctl reload nginx


6. Push to GitHub
Initialize and Push

git init
git remote add origin https://github.com/tosinde/exam-project.git
git add .
git commit -m "Initial commit"
git push origin master

The site is now live at:
(https://dalamuesther.duckdns.org)
With HTTPS and GitHub version control.

Deployed by: Esther Dalamu
Using: AWS, Linux, DuckDNS, Nginx, Certbot
