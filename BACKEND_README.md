# 🗑️ Trash Hosting — Unified Backend System

**Complete, production-ready setup with Pterodactyl Panel & Wings integration.**

## 📦 What You Have Now

### Core Files
- **`.env.example`** - Template for all environment variables
- **`backend/config.js`** - Centralized configuration with validation
- **`backend/utils.js`** - Logging system with Discord webhooks
- **`backend/pterodactyl.js`** - Complete Pterodactyl API client
- **`backend/db-init.js`** - Database management with auto-backups
- **`backend/server-additions.js`** - All backend routes

### Documentation
- **`SETUP.md`** - Step-by-step deployment guide
- **`CREDENTIALS.md`** - Where to find every credential value

---

## 🚀 Quick Start (5 minutes)

### 1. Create `.env` file

```bash
cd ~/trashbackend
cp .env.example .env
nano .env  # Fill in your credentials from CREDENTIALS.md
```

### 2. Install dependencies

```bash
npm install express cors dotenv jsonwebtoken lowdb@1.0.0 node-fetch@2
```

### 3. Test it

```bash
node backend/server-additions.js
# Should show: ✅ Configuration validated successfully
```

### 4. Deploy with PM2

```bash
npm install -g pm2
pm2 start backend/server-additions.js --name trash-hosting
pm2 logs trash-hosting
```

---

## 🔑 Required Credentials

See **`CREDENTIALS.md`** for exact locations:

```env
DISCORD_CLIENT_ID=
DISCORD_CLIENT_SECRET=
DISCORD_BOT_TOKEN=
PTERODACTYL_PANEL_URL=https://panel.your-domain.com
PTERODACTYL_API_KEY=
WINGS_URL=https://wings.your-domain:8443
WINGS_AUTH_TOKEN=
JWT_SECRET=
```

---

## ✅ What's Working

✅ Discord OAuth authentication  
✅ Guest login system  
✅ Pterodactyl Panel API  
✅ Pterodactyl Wings commands  
✅ Server creation/start/stop  
✅ Console commands  
✅ Coin system  
✅ User management  
✅ Database with auto-backups  
✅ Centralized logging  
✅ Error tracking  

---

## 📚 Documentation

- **[SETUP.md](./SETUP.md)** - Full deployment walkthrough
- **[CREDENTIALS.md](./CREDENTIALS.md)** - How to get each credential

---

## 🆘 Common Issues

**"Invalid Pterodactyl API Key"**
- Generate a new API key in Panel → Settings → API Tokens
- Make sure it's an Application token, not Personal

**"Wings connection refused"**
- Check Wings URL includes port: `https://domain:8443`
- Verify firewall allows 8443

**"Cannot find module"**
- Run: `npm install jsonwebtoken lowdb@1.0.0 node-fetch@2 cors`

See **[SETUP.md](./SETUP.md#-troubleshooting)** for more help.

---

**Follow SETUP.md to deploy!** 🚀
