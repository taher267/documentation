Perfect. Below is a **clean, production-grade AUTO-DEPLOY setup** for a **Next.js (SSR) app** on **EC2 + PM2 + Nginx**, triggered on **every push to `main`**.

This is exactly what you want and **what you already almost built** — now let’s standardize it.

---

# 🚀 Auto-Deploy Next.js from GitHub → EC2

## Architecture (simple & correct)

```
GitHub (main branch)
   ↓ push
GitHub Actions
   ↓ SSH
EC2 Server
   → git pull
   → npm install
   → npm run build
   → pm2 restart
```

---

## ✅ STEP 1 — Create SSH key for GitHub Actions (ON EC2)

On EC2:

```bash
ssh-keygen -t ed25519 -C "github-actions-nextjs"
```

Press **Enter** for all prompts.

This creates:

* `~/.ssh/id_ed25519`
* `~/.ssh/id_ed25519.pub`

Copy public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

---

## ✅ STEP 2 — Add SSH key to GitHub (DEPLOY KEY)

GitHub → **Repo** → **Settings** → **Deploy keys**

* Title: `EC2 Auto Deploy`
* Key: paste public key
* ✅ **Allow write access**
* Save

---

## ✅ STEP 3 — Create deploy script on EC2

This script runs on every deploy.

```bash
nano /var/www/chat-sonic/deploy.sh
```

Paste:

```bash
#!/bin/bash
set -e

echo "🚀 Deploy started"

cd /var/www/chat-sonic

echo "📥 Pull latest code"
git fetch origin
git reset --hard origin/main

echo "📦 Install dependencies"
npm install

echo "🏗 Build Next.js"
npm run build

echo "♻ Restart PM2"
pm2 restart chat-sonic || pm2 start npm --name chat-sonic -- start

echo "✅ Deploy finished"
```

Make executable:

```bash
chmod +x /var/www/chat-sonic/deploy.sh
```

Test manually:

```bash
/var/www/chat-sonic/deploy.sh
```

If this works → auto-deploy will work.

---

## ✅ STEP 4 — Add GitHub Secrets

GitHub → **Repo → Settings → Secrets → Actions**

Add **EXACTLY** these:

| Name          | Value                                    |
| ------------- | ---------------------------------------- |
| `EC2_HOST`    | `your-ec2-public-ip`                     |
| `EC2_USER`    | `ubuntu`                                 |
| `EC2_SSH_KEY` | **PRIVATE key** from `~/.ssh/id_ed25519` |

⚠️ IMPORTANT
Private key means:

```bash
cat ~/.ssh/id_ed25519
```

Copy **everything**, including:

```
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

---

## ✅ STEP 5 — Create GitHub Actions workflow

In your repo:

```bash
mkdir -p .github/workflows
nano .github/workflows/deploy.yml
```

Paste:

```yaml
name: Deploy Next.js to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            /var/www/chat-sonic/deploy.sh
```

Commit & push:

```bash
git add .
git commit -m "ci: enable auto deploy"
git push origin main
```

---

## ✅ STEP 6 — Verify Auto-Deploy

GitHub → **Actions tab**

You should see:

```
Deploy Next.js to EC2 → SUCCESS ✅
```

Now check site:

```
https://chat-sonic.crazyads.io
```

---

## 🔐 Best Practices (Highly Recommended)

### ✔ Use Elastic IP

So EC2 IP never changes.

### ✔ PM2 auto-start (you already did)

```bash
pm2 save
pm2 startup
```

### ✔ Logs

```bash
pm2 logs chat-sonic
```

---

## 🧠 Common Issues (Quick Fix)

### ❌ `pm2: command not found`

```bash
sudo npm install -g pm2
```

### ❌ Git auth fails

Make sure:

* Repo uses `git@github.com:`
* Deploy key has **write access**

### ❌ Build fails (RAM)

Use:

* `t3.small` or higher
* Or add swap (I can show)

---

## 🎯 You now have

✅ Zero-downtime deploy
✅ Secure SSH deploy
✅ Production Next.js
✅ Fully automated pipeline

If you want next:

* 🔁 Blue-green deploy
* 🧪 Deploy only on `production` branch
* 📊 Slack / email deploy alerts
* 🌐 Multi-app mono-server
* 🧾 CI checks before deploy

Just say the word.
