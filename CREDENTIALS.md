# 🔑 How to Get Each Credential

Complete guide showing **exactly where** to find every value needed in `.env`

---

## 🎮 Discord OAuth

### 1. Client ID & Secret

**Where**: Discord Developer Portal  
**URL**: https://discord.com/developers/applications

**Steps**:
1. Click "New Application" → Name it (e.g., "Trash Hosting")
2. Go to "OAuth2" → "General"
3. Copy **Client ID** → Paste in `.env` as `DISCORD_CLIENT_ID`
4. Click "Reset Secret" → Copy it → Paste as `DISCORD_CLIENT_SECRET`
5. Go to "OAuth2" → "URL Generator"
   - Select scopes: `identify`, `email`
   - Copy the generated URL (we'll use this later)

### 2. Redirect URI

**Value**: Your GitHub Pages URL + `/` at the end

```
https://shivam9864op.github.io/trashfreehosting/
```

**Where to add**:
1. Discord Developer Portal → Your app → OAuth2 → Redirects
2. Click "Add Redirect" → Paste the URL above
3. Save

### 3. Bot Token (Optional, for Discord error logging)

**Where**: Discord Developer Portal → Your app → "Bot" tab

**Steps**:
1. Click "Add Bot"
2. Under TOKEN, click "Copy" → Paste as `DISCORD_BOT_TOKEN`
3. (Optional) In Discord server, add bot with role to send messages

**Test it**:
```bash
curl -H "Authorization: Bot YOUR_BOT_TOKEN" https://discord.com/api/users/@me
# Should return your bot info
```

---

## 🖥️ Pterodactyl Panel

### 1. Panel URL

**Value**: Your Pterodactyl Panel domain

```
https://panel.your-domain.com
```

Or if self-hosted on same machine:

```
https://localhost:8080
```

**Where to add**: `.env` as `PTERODACTYL_PANEL_URL`

### 2. Application API Key

**Where**: Pterodactyl Panel → Settings → API Tokens

**Steps**:
1. Login to your Pterodactyl Panel
2. Go to **Settings** → **API Tokens**
3. Click **Create Token**
4. Description: "Trash Hosting Backend"
5. Allowed IPs: Leave blank (or your backend IP)
6. Click **Create**
7. Copy the token → Paste as `PTERODACTYL_API_KEY` in `.env`

**⚠️ Important**: Must be an **Application** token, NOT personal token

**Test it**:
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json" \
  https://panel.your-domain.com/api/application/servers
# Should return list of servers
```

---

## 🛡️ Pterodactyl Wings

### 1. Wings URL

**Value**: Your Wings domain with port 8443

```
https://wings.your-domain:8443
```

Or if self-hosted on same machine:

```
https://localhost:8443
```

**Where to add**: `.env` as `WINGS_URL`

### 2. Wings Auth Token

**Where**: SSH into your home server → `/etc/pterodactyl/wings/config.yml`

**Steps**:
1. SSH into your home server running Wings
2. Read the config file:
   ```bash
   cat /etc/pterodactyl/wings/config.yml | grep -A 5 "auth:"
   ```
3. Look for the `auth:` section:
   ```yaml
   auth:
     token: "YOUR_WINGS_TOKEN_HERE"
   ```
4. Copy that token → Paste as `WINGS_AUTH_TOKEN` in `.env`

**If Wings is in Docker**:
```bash
docker exec pterodactyl_wings cat /etc/pterodactyl/wings/config.yml | grep -A 5 "auth:"
```

**Test it**:
```bash
curl -H "Authorization: Bearer YOUR_WINGS_TOKEN" \
  -H "Accept: application/json" \
  https://wings.your-domain:8443/api/system
# Should return Wings system info (ignoring SSL cert errors is ok)
```

---

## 🔐 JWT Secret

**Where**: Generate it yourself (security key)

**How to generate**:

```bash
# Linux/Mac
openssl rand -hex 32

# Windows PowerShell
[Convert]::ToHexString((1..32 | ForEach-Object { Get-Random -Maximum 256 }))

# Or use online: https://generate-random.org/ (hex, 32 bytes)
```

**Output example**:
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

**Where to add**: `.env` as `JWT_SECRET`

---

## 🌐 Frontend Origin

**Value**: Your GitHub Pages URL

```
https://shivam9864op.github.io/trashfreehosting
```

**Where to add**: `.env` as `FRONTEND_ORIGIN`

---

## 📋 Quick Copy-Paste Template

Once you have all values, fill this in:

```env
# Discord
DISCORD_CLIENT_ID=
DISCORD_CLIENT_SECRET=
DISCORD_REDIRECT_URI=https://shivam9864op.github.io/trashfreehosting/
DISCORD_BOT_TOKEN=

# Pterodactyl Panel
PTERODACTYL_PANEL_URL=
PTERODACTYL_API_KEY=

# Pterodactyl Wings
WINGS_URL=
WINGS_AUTH_TOKEN=

# JWT
JWT_SECRET=

# Frontend
FRONTEND_ORIGIN=https://shivam9864op.github.io/trashfreehosting

# Server
PORT=3000
NODE_ENV=production
```

---

## ✅ Quick Verification Checklist

After filling `.env`, verify each credential works:

```bash
# 1. Test Discord OAuth
curl -X POST https://discord.com/api/oauth2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&grant_type=client_credentials"
# Should return access_token

# 2. Test Pterodactyl Panel
curl -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json" \
  https://panel.your-domain.com/api/application/servers
# Should return server list (may be empty)

# 3. Test Wings
curl -k -H "Authorization: Bearer YOUR_WINGS_TOKEN" \
  https://wings.your-domain:8443/api/system
# Should return system info

# 4. Test JWT Secret
node -e "const jwt = require('jsonwebtoken'); console.log(jwt.sign({test:1}, 'YOUR_JWT_SECRET'))"
# Should print a token
```

---

## 🆘 Troubleshooting

**"Cannot reach Panel"**
- Check PTERODACTYL_PANEL_URL is correct and accessible
- Try `curl https://your-panel-url` manually

**"Invalid API Key"**
- Make sure it's an **Application** token, not Personal
- Generate a new one if unsure

**"Wings connection refused"**
- Check port 8443 is open: `telnet wings.your-domain 8443`
- Verify WINGS_URL format: `https://domain:8443` (with https and port)

**"Discord OAuth fails"**
- Verify Redirect URI matches exactly in Discord Developer Portal
- Make sure it ends with `/` if your frontend URL does

---

## 📝 Notes

- **Never commit `.env` to git** - it contains secrets!
- Add `.env` to `.gitignore`
- Each credential is required for full functionality
- For development, you can use `http://localhost` URLs if testing locally
- Rotate credentials periodically for security
