# 🚀 Complete Deployment Guide

Step-by-step instructions to get your Trash Hosting backend **live and working** with your Pterodactyl setup.

---

## 📋 Prerequisites

- Node.js 16+ installed on your VPS
- Pterodactyl Panel running on VPS
- Pterodactyl Wings running (on home server or same VPS)
- Discord Developer Application (for OAuth)
- GitHub account with Pages enabled

---

## 🔑 Step 1: Gather All Credentials (15 minutes)

**Open `CREDENTIALS.md` and follow it completely.** Get values for:

- [ ] `DISCORD_CLIENT_ID`
- [ ] `DISCORD_CLIENT_SECRET`
- [ ] `DISCORD_REDIRECT_URI`
- [ ] `PTERODACTYL_PANEL_URL`
- [ ] `PTERODACTYL_API_KEY`
- [ ] `WINGS_URL`
- [ ] `WINGS_AUTH_TOKEN`
- [ ] `JWT_SECRET` (generate it)
- [ ] `FRONTEND_ORIGIN`

**Keep these values handy** — you'll paste them in the next step.

---

## 🔧 Step 2: Setup Backend Directory

```bash
# SSH into your VPS
ssh root@your-vps-ip

# Create backend directory
mkdir -p ~/trashbackend
cd ~/trashbackend

# Copy all backend files
# (Or clone your repo if you pushed them)
git clone https://github.com/Shivam9864op/trashfreehostingv2.git .
```

---

## 📝 Step 3: Create `.env` File

```bash
cd ~/trashbackend

# Copy template
cp .env.example .env

# Edit with your values
nano .env
```

Fill in all the values you gathered from Step 1. **Example**:

```env
DISCORD_CLIENT_ID=1234567890123456789
DISCORD_CLIENT_SECRET=xyz_very_long_secret_string
DISCORD_REDIRECT_URI=https://shivam9864op.github.io/trashfreehosting/
DISCORD_BOT_TOKEN=Bot_token_optional

PTERODACTYL_PANEL_URL=https://panel.your-domain.com
PTERODACTYL_API_KEY=app_abc123...

WINGS_URL=https://wings.your-domain:8443
WINGS_AUTH_TOKEN=wings_token_here

JWT_SECRET=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6

FRONTEND_ORIGIN=https://shivam9864op.github.io/trashfreehosting

PORT=3000
NODE_ENV=production
```

**Save & exit**: `Ctrl+X` → `Y` → `Enter`

---

## 📦 Step 4: Install Dependencies

```bash
cd ~/trashbackend

npm install express cors dotenv jsonwebtoken lowdb@1.0.0 node-fetch@2

# Optional: Install PM2 globally for production
npm install -g pm2
```

---

## ✅ Step 5: Test Backend

```bash
# Test configuration loads correctly
node -e "require('dotenv').config(); require('./backend/config').validate()"

# Should print: ✅ Configuration validated successfully

# Test all modules load
node -e "require('./backend/config'); require('./backend/utils'); require('./backend/pterodactyl'); require('./backend/db-init'); console.log('✅ All modules loaded')"
```

---

## 🏃 Step 6: Run Backend

### Option A: Direct (Development/Testing)

```bash
node backend/server-additions.js

# Should show:
# ℹ️  🗑️ Trash Hosting Backend Starting...
# ✅ Configuration validated successfully
# ✅ Backend running on port 3000
```

**Test it**:
```bash
curl http://localhost:3000/api/health
# Should return JSON with status and health checks
```

### Option B: PM2 (Production - Recommended)

```bash
# Start with PM2
pm2 start backend/server-additions.js --name trash-hosting

# View logs
pm2 logs trash-hosting

# Make it restart on reboot
pm2 startup
pm2 save
```

---

## 🌐 Step 7: Setup NGINX Reverse Proxy

Your backend is running on port 3000, but you want it accessible via HTTPS on your domain.

```bash
# Create NGINX config
sudo nano /etc/nginx/sites-available/trash-hosting
```

Paste this config:

```nginx
server {
    listen 80;
    server_name hosting.trashmcpe.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}
```

Enable it:

```bash
sudo ln -s /etc/nginx/sites-available/trash-hosting /etc/nginx/sites-enabled/
sudo nginx -t  # Test config
sudo systemctl restart nginx
```

**Get SSL Certificate** (Let's Encrypt):

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d hosting.trashmcpe.com
# Follow prompts, auto-renews
```

Now your backend is accessible at: `https://hosting.trashmcpe.com`

---

## 🧪 Step 8: Verify Everything Works

### Test Health Check
```bash
curl https://hosting.trashmcpe.com/api/health

# Should return:
{
  "status": "ok",
  "timestamp": "2026-05-05T...",
  "checks": {
    "database": "ok",
    "pterodactyl": "ok",
    "wings": "ok"
  }
}
```

### Test Discord OAuth
```bash
# Create test user with Guest login first
curl -X POST https://hosting.trashmcpe.com/api/auth/guest \
  -H "Content-Type: application/json" \
  -d '{"username": "TestPlayer"}'

# Should return token and user info
```

### Test Coin System
```bash
# Replace TOKEN with the token from guest login above
curl -X POST https://hosting.trashmcpe.com/api/user/coins/add \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"amount": 50, "reason": "test"}'

# Should return updated coin balance
```

### Test Pterodactyl Connection
```bash
curl -H "Authorization: Bearer TOKEN" \
  https://hosting.trashmcpe.com/api/servers

# Should return server list (may be empty if none created yet)
```

---

## 📊 Step 9: Monitor & Maintain

### View Live Logs
```bash
pm2 logs trash-hosting --lines 50
```

### Monitor Dashboard
```bash
pm2 monit
```

### Database Location
```bash
# Your database is at:
~/trashbackend/db.json

# Backups are at:
~/trashbackend/backups/
```

### Automatic Backups
Database backs up **every hour** automatically. Manual backup:

```bash
curl -X POST https://hosting.trashmcpe.com/api/admin/backup \
  -H "Authorization: Bearer ADMIN_TOKEN"
```

---

## 🔗 Step 10: Connect Frontend

Update your frontend code to point to your backend:

In `script.js`, change:

```javascript
const BACKEND = "https://hosting.trashmcpe.com/backend";
```

Or in your `.env` (if using one):

```env
VITE_BACKEND_URL=https://hosting.trashmcpe.com/backend
```

---

## 🎉 Deployment Checklist

- [ ] All credentials gathered from CREDENTIALS.md
- [ ] `.env` file created and filled with values
- [ ] Dependencies installed
- [ ] Backend tests pass (health check)
- [ ] PM2 running backend
- [ ] NGINX reverse proxy configured
- [ ] SSL certificate installed
- [ ] Discord OAuth redirect URI set
- [ ] Frontend updated to use backend URL
- [ ] Database initialized
- [ ] Pterodactyl Panel API key verified
- [ ] Wings auth token verified

---

## 🚨 Troubleshooting

### Backend Won't Start
```bash
# Check .env is valid
cat .env

# Check all required vars are set
node -e "require('dotenv').config(); require('./backend/config').validate()"

# Check for JS errors
node backend/server-additions.js
```

### "Cannot reach Pterodactyl"
```bash
# Test Panel connectivity
curl https://PANEL_URL/api/application/servers \
  -H "Authorization: Bearer YOUR_API_KEY"

# Test Wings connectivity
curl -k https://WINGS_URL/api/system \
  -H "Authorization: Bearer YOUR_WINGS_TOKEN"
```

### "Discord OAuth fails"
- Verify `DISCORD_REDIRECT_URI` matches exactly in Discord Developer Portal
- Check `DISCORD_CLIENT_ID` and `DISCORD_CLIENT_SECRET` are correct

### "Port already in use"
```bash
# Find what's using port 3000
lsof -i :3000

# Kill it
kill -9 <PID>

# Or use different port in .env
PORT=3001
```

### "Database corrupted"
```bash
# Restore from backup
cp ~/trashbackend/backups/db-backup-*.json ~/trashbackend/db.json

# Or reset (WARNING: loses data)
rm ~/trashbackend/db.json
# Restart backend to recreate
```

### Check PM2 Errors
```bash
pm2 logs trash-hosting --err
```

---

## 📞 Getting Help

1. **Check logs first**: `pm2 logs trash-hosting`
2. **Read error messages** in CREDENTIALS.md troubleshooting section
3. **Verify each credential** works individually
4. **Test endpoints** with curl commands above

---

## 🎯 What's Next

✅ Backend is live and connected to Pterodactyl  
✅ Discord OAuth is working  
✅ Coin system is operational  
✅ Database is auto-backing up  

Now:
1. Test creating a server from the frontend
2. Monitor server lifecycle (start/stop/restart)
3. Verify coins are earned correctly
4. Check leaderboard
5. Setup admin panel access

---

**Your Trash Hosting is now LIVE! 🗑️🚀**
