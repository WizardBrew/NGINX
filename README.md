# NGINX
# Ubuntu Nginx Installation & Site Configuration Guide

## Overview

This guide explains how to:

* Install Nginx on Ubuntu
* Understand the Nginx directory structure
* Create and configure websites
* Enable and disable sites
* Configure reverse proxies
* Enable HTTPS with Let's Encrypt
* Troubleshoot common issues

---

# 1. Update the System

```bash
# Update package information
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y
```

---

# 2. Install Nginx

```bash
# Install Nginx
sudo apt install nginx -y
```

Verify installation:

```bash
# Check service status
sudo systemctl status nginx
```

Start and enable automatic startup:

```bash
# Start Nginx service
sudo systemctl start nginx

# Enable startup on boot
sudo systemctl enable nginx
```

---

# 3. Configure Firewall (Optional)

If UFW is enabled:

```bash
# Allow HTTP and HTTPS traffic
sudo ufw allow 'Nginx Full'

# Reload firewall rules
sudo ufw reload
```

Check firewall status:

```bash
sudo ufw status
```

---

# 4. Nginx Directory Structure

```text
/etc/nginx/
├── nginx.conf             # Main configuration file
├── sites-available/       # All website configurations
├── sites-enabled/         # Enabled websites (symlinks)
├── conf.d/                # Additional configuration files
├── snippets/              # Reusable configuration snippets
├── mime.types             # MIME type mappings
└── modules-enabled/       # Enabled modules
```

---

# 5. Request Processing Workflow

```text
Client Browser
       │
       ▼
Port 80 / 443
       │
       ▼
Nginx Master Process
       │
       ▼
Worker Processes
       │
       ▼
Match server_name
       │
       ├── Serve static files
       │
       └── Reverse proxy to backend
```

---

# 6. Create Website Directory

```bash
# Create website root directory
sudo mkdir -p /var/www/mysite
```

Set ownership:

```bash
# Give current user ownership
sudo chown -R $USER:$USER /var/www/mysite
```

Set permissions:

```bash
# Owner: read/write/execute
# Others: read/execute
sudo chmod -R 755 /var/www/mysite
```

---

# 7. Create Website Content

Create index.html:

```bash
nano /var/www/mysite/index.html
```

Example:

```html
<!DOCTYPE html>
<html>

<head>
    <title>My Website</title>
</head>

<body>

<h1>Welcome to Nginx</h1>
<p>Website deployment successful.</p>

</body>

</html>
```

Save:

```text
CTRL + O
ENTER
CTRL + X
```

---

# 8. Create Nginx Site Configuration

Create:

```bash
sudo nano /etc/nginx/sites-available/mysite
```

Example configuration:

```nginx
server {

    # Listen on HTTP port
    listen 80;

    # Domain names
    server_name mysite.com www.mysite.com;

    # Website root folder
    root /var/www/mysite;

    # Default files
    index index.html index.htm;

    # Access logs
    access_log /var/log/nginx/mysite-access.log;

    # Error logs
    error_log /var/log/nginx/mysite-error.log;

    location / {

        # Check file, directory, or return 404
        try_files $uri $uri/ =404;

    }

}
```

---

# 9. Enable Website

Create a symbolic link:

```bash
sudo ln -s \
/etc/nginx/sites-available/mysite \
/etc/nginx/sites-enabled/
```

Nginx loads only configurations present inside:

```text
/etc/nginx/sites-enabled/
```

---

# 10. Disable Default Site

Remove Ubuntu's default site:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

---

# 11. Validate Configuration

Always test before reloading:

```bash
sudo nginx -t
```

Expected output:

```text
nginx: configuration file test is successful
```

---

# 12. Reload Configuration

```bash
sudo systemctl reload nginx
```

Difference:

```text
reload  = Apply changes without downtime
restart = Stop and start service
```

---

# 13. Multiple Website Setup

Example:

```text
sites-available/

├── blog
├── shop
└── api
```

Enabled:

```text
sites-enabled/

blog -> ../sites-available/blog
shop -> ../sites-available/shop
api  -> ../sites-available/api
```

Request routing:

```text
blog.example.com
      │
      ▼
server_name blog.example.com
      │
      ▼
/var/www/blog
```

---

# 14. Reverse Proxy Example

Suppose an application runs on:

```text
127.0.0.1:3000
```

Nginx configuration:

```nginx
server {

    listen 80;

    server_name api.example.com;

    location / {

        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

    }

}
```

Workflow:

```text
Browser
   │
   ▼
Nginx
   │
   ▼
Node.js / Python / Docker
   │
   ▼
Response
```

---

# 15. Enable HTTPS

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Generate certificate:

```bash
sudo certbot --nginx \
-d mysite.com \
-d www.mysite.com
```

Certbot automatically:

* Verifies ownership
* Creates certificates
* Updates Nginx configuration
* Enables HTTPS redirection

---

# 16. Useful Commands

```bash
# Validate configuration
sudo nginx -t

# Reload without downtime
sudo systemctl reload nginx

# Restart service
sudo systemctl restart nginx

# Check service status
sudo systemctl status nginx

# Follow service logs
sudo journalctl -u nginx -f

# View error logs
tail -f /var/log/nginx/error.log

# View access logs
tail -f /var/log/nginx/access.log
```

---

# 17. Complete Deployment Workflow

```text
Install Ubuntu
      │
      ▼
Update Packages
      │
      ▼
Install Nginx
      │
      ▼
Configure Firewall
      │
      ▼
Create Website Directory
      │
      ▼
Upload HTML/CSS/JS
      │
      ▼
Create Site Configuration
      │
      ▼
Enable Site
      │
      ▼
Test Configuration
      │
      ▼
Reload Nginx
      │
      ▼
Configure DNS
      │
      ▼
Enable HTTPS
      │
      ▼
Production Deployment
```

---

# Best Practices

* Always run `nginx -t` before reloading.
* Use `reload` instead of `restart` whenever possible.
* Keep one configuration file per website.
* Store reusable configurations inside `snippets/`.
* Enable HTTPS for every public website.
* Maintain separate access and error logs for each site.
* Back up `/etc/nginx/` before major changes.

---

# License

This document is provided for educational and operational use.
