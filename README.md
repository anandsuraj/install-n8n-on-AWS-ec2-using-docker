
# 🚀 Deploy n8n on AWS EC2 (Amazon Linux 2023) with Docker, Nginx & SSL

This guide will help you deploy [n8n](https://n8n.io) on an EC2 instance using **Amazon Linux 2023**, **Docker**, **Nginx** as a reverse proxy, and **Let's Encrypt (Certbot)** for free SSL.


---

**Primary Keywords:** - n8n EC2 installation - n8n AWS deployment - n8n on Amazon Linux - n8n Docker setup AWS - how to deploy n8n on EC2 - n8n installation with Docker and Nginx - n8n EC2 with SSL - n8n self-hosted AWS - n8n Amazon Linux 2023 setup - n8n EC2 Docker Nginx Certbot **Technical Tags:** - n8n - AWS EC2 - Amazon Linux 2023 - Docker - Nginx - Certbot - Let's Encrypt - reverse proxy - secure deployment - workflow automation - self-hosted n8n - DevOps - backend automation tools **Long-Tail Keywords:** - Step-by-step guide to install n8n on AWS EC2 using Docker - Securely deploy n8n on Amazon Linux 2023 with Nginx and SSL - Install and configure n8n on EC2 instance with HTTPS - Self-hosting n8n with Docker and Let's Encrypt on AWS
## ✅ Prerequisites

- Amazon Linux 2023 EC2 instance (e.g., t2.micro, free-tier)
- Domain pointing to EC2 public IP (e.g., `ekaivakriti.zapto.org`)
- EC2 Security Group allows ports: 22, 80, 443
- Logged in as `ec2-user`

---

## 🔧 Step 1: Install Docker

```bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
````

> Log out and back in if needed to activate Docker group.

---

## 🐳 Step 2: Start n8n Container

```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_EXTERNAL_URL="https://ekaivakriti.zapto.org" \
-e N8N_SECURE_COOKIE=false \
-e N8N_HOST="ekaivakriti.zapto.org" \
-e VUE_APP_URL_BASE_API="https://ekaivakriti.zapto.org/" \
-e WEBHOOK_TUNNEL_URL="https://ekaivakriti.zapto.org/" \
-e N8N_EDITOR_BASE_URL="https://ekaivakriti.zapto.org/" \
-e N8N_PROTOCOL="https" \
-e WEBHOOK_URL="https://ekaivakriti.zapto.org/webhook/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
```

---

## 🌐 Step 3: Install Nginx

```bash
sudo yum install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## ⚙️ Step 4: Configure Nginx

Create the config file:

```bash
sudo nano /etc/nginx/conf.d/n8n.conf
```

Paste the following configuration:

```nginx
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    # Redirect HTTP to HTTPS
    server {
        listen 80;
        server_name ekaivakriti.zapto.org;

        location / {
            return 301 https://$host$request_uri;
        }
    }

    # HTTPS server
    server {
        listen 443 ssl;
        server_name ekaivakriti.zapto.org;

        #ssl_certificate /etc/letsencrypt/..../fullchain.pem; #update to your ssl path
        #ssl_certificate_key /etc/letsencrypt/..../privkey.pem; #update to your ssl path

        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;

        location / {
            proxy_pass http://localhost:5678;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_buffering off;
            proxy_cache off;
        }
    }
}
```

Reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔒 Step 5: Install Certbot & Add SSL

```bash
sudo yum install -y python3-certbot-nginx
sudo certbot --nginx -d ekaivakriti.zapto.org
```

> Choose the option to redirect HTTP to HTTPS when prompted.

---

## 🧪 Debugging Tips

**Check running container:**

```bash
sudo docker ps
sudo docker logs n8n
```

**Restart services:**

```bash
sudo systemctl restart docker
sudo systemctl restart nginx
```

**Firewall/port issues:**

Ensure EC2 security group allows **80**, **443**, and **22**

**Check domain resolution:**

```bash
nslookup ekaivakriti.zapto.org
```

**Access your instance:**

Open in browser: `https://ekaivakriti.zapto.org`

**Note: Replace https://ekaivakriti.zapto.org to Your Domain.**

---

## ✅ All Done!
