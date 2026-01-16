Absolutely — here is a **clean, complete, one-shot checklist**
for deploying a **Next.js (SSR) app** on **AWS EC2** with **Nginx + PM2 + SSL**.

This is the **best production setup** and works for ALL Next.js versions including 13/14/15/16 (App Router or Pages Router).

---

# ✅ **FULL DEPLOYMENT GUIDE (EC2 → Next.js → SSL)**

### **Step-by-step from scratch**

---

# 🟩 **STEP 1 — Launch EC2 Instance**

Choose:

* **Ubuntu 22.04 or 24.04**
* Instance type:
  → `t3.small` (2GB RAM) minimum for Next.js
* Storage:
  → 20–30GB gp3
* Create Security Group:

  * Allow **22** (SSH)
  * Allow **80** (HTTP)
  * Allow **443** (HTTPS)
* Download your `.pem` key

---

# 🟩 **STEP 2 — SSH into server**

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP
```

---

# 🟩 **STEP 3 — Update server**

```bash
sudo apt update && sudo apt upgrade -y
```

---

# 🟩 **STEP 4 — Install Node.js (LTS recommended)**

For stability, install Node 20:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Check:

```bash
node -v
npm -v
```

---

# 🟩 **STEP 5 — Install Git**

```bash
sudo apt install git -y
```

---

# 🟩 **STEP 6 — Setup folder**

```bash
sudo mkdir -p /var/www/app
sudo chown -R ubuntu:ubuntu /var/www/app
cd /var/www/app
```

---

# 🟩 **STEP 7 — Setup SSH key for private Github repo**

```bash
ssh-keygen -t ed25519 -C "deploy-key"
```

Press Enter for all questions.

Get your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy → add to:

**GitHub → Repo → Settings → Deploy keys → Add key (Read-only)**

---

# 🟩 **STEP 8 — Clone your repo**

```bash
git clone git@github.com:USERNAME/REPO.git .
```

---

# 🟩 **STEP 9 — Add your `.env`**

Create:

```bash
nano .env
```

Add your values (public + private).
Save → exit.

Lock permissions:

```bash
chmod 600 .env
```

---

# 🟩 **STEP 10 — Install dependencies + build Next.js**

```bash
npm install
npm run build
```

---

# 🟩 **STEP 11 — Install PM2 (for production runtime)**

```bash
sudo npm install -g pm2
```

Start your app:

```bash
pm2 start npm --name app -- start
pm2 save
pm2 startup
```

Run the generated command:

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

---

# 🟩 **STEP 12 — Setup Nginx reverse proxy**

Create file:

```bash
sudo nano /etc/nginx/sites-available/yourdomain.com
```

Paste:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable:

```bash
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

# 🟩 **STEP 13 — Add DNS record in Namecheap**

In **Advanced DNS**:

| Type | Host          | Value (EC2 Public IP) |
| ---- | ------------- | --------------------- |
| A    | @ / subdomain | 3.xxx.xxx.xxx         |

Wait 1–5 minutes.

---

# 🟩 **STEP 14 — Enable SSL (HTTPS)**

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run:

```bash
sudo certbot --nginx -d yourdomain.com
```

Choose:
✔ Redirect HTTP → HTTPS

Test:

```bash
sudo certbot renew --dry-run
```

---

# 🟩 **STEP 15 — Verify everything**

### App running?

```bash
curl http://localhost:3000
```

### Nginx working?

```bash
curl -I http://yourdomain.com
```

### PM2 working?

```bash
pm2 status
```

### Site live?

Open:

```
https://yourdomain.com
```

---

# 🟩 **STEP 16 — Optional (Highly Recommended)**

### ✓ Enable Auto-Deploy from GitHub

### ✓ Add firewall hardening

### ✓ Setup monitoring (pm2 logs, uptime-kuma)

### ✓ Use Elastic IP (static IP)

I can help with any of these.

---

# 🎉 FINISHED

This is the **complete production deployment workflow** from creating an instance → running Next.js → adding SSL.

If you want, I can generate a **copy-paste-ready doc** for your team or future projects.

Just tell me:
👉 **“Make a documentation file”**
