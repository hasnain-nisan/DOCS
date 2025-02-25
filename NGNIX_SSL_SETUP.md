# Nginx Setup, SSL Configuration, and Deployment Instructions

## Table of Contents
- [1. Install Nginx](#1-install-nginx)
- [2. Start and Enable Nginx](#2-start-and-enable-nginx)
- [3. Configure Nginx](#3-configure-nginx)
- [4. Restart and Check Nginx Status](#4-restart-and-check-nginx-status)
- [5. Install Certbot for SSL](#5-install-certbot-for-ssl)
- [6. Obtain and Configure SSL Certificate](#6-obtain-and-configure-ssl-certificate)
- [7. Automate SSL Renewal](#7-automate-ssl-renewal)

---

## 1. Install Nginx
To install Nginx on your server, run:
```sh
sudo yum install nginx -y
```

## 2. Start and Enable Nginx
Once installed, start and enable Nginx with:
```sh
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 3. Configure Nginx
Edit the default Nginx configuration file:
```sh
sudo nano /etc/nginx/conf.d/default.conf
```

Paste the following configuration:
```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    access_log /var/log/nginx/your-app.access.log;
    error_log /var/log/nginx/your-app.error.log;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    location /static/ {
        alias /path/to/your/static/files/;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
}
```

## 4. Restart and Check Nginx Status
Restart Nginx to apply changes:
```sh
sudo systemctl restart nginx
```
Check the configuration:
```sh
sudo nginx -t
```

## 5. Install Certbot for SSL
Install Certbot and its Nginx plugin:
```sh
sudo yum install -y certbot python-certbot-nginx
```

## 6. Obtain and Configure SSL Certificate
Modify the `server_name` in the Nginx configuration to your domain, then run:
```sh
sudo certbot --nginx -d your-domain.com
```
Follow the prompts to install and configure SSL.

## 7. Automate SSL Renewal
Create a directory for logs:
```sh
sudo mkdir -p /var/log/certbot
```
Create a renewal script:
```sh
sudo nano /usr/local/bin/renew-certificates.sh
```
Paste the following script:
```sh
#!/bin/bash
certbot renew --quiet --no-self-upgrade
if [ $? -eq 0 ]; then
    systemctl reload nginx
    echo "$(date): Certificate renewal successful" >> /var/log/certbot/renewal.log
else
    echo "$(date): Certificate renewal failed" >> /var/log/certbot/renewal.log
fi
find /var/log/certbot/renewal.log -mtime +30 -delete
```
Make the script executable:
```sh
sudo chmod +x /usr/local/bin/renew-certificates.sh
```
Test the script:
```sh
sudo /usr/local/bin/renew-certificates.sh
```

### Schedule SSL Renewal with Cron
Install Cron if not installed:
```sh
sudo yum install cronie -y
```
Start and enable the cron service:
```sh
sudo systemctl start crond
sudo systemctl enable crond
```
Edit the crontab:
```sh
sudo su
crontab -e
```
Add the following line to renew certificates twice daily:
```sh
0 0,12 * * * /usr/local/bin/renew-certificates.sh
```
Verify the crontab entry:
```sh
crontab -l
```
Check if cron service is running:
```sh
systemctl status crond
```

---
## Conclusion
This guide walks through setting up Nginx, configuring SSL and automating renewal. If you encounter issues, check logs using:
```sh
sudo journalctl -u nginx --no-pager -n 100
```
Happy coding! 🚀

