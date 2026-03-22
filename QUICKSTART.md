# 🎯 Quick Setup Guide

## What's Been Built

Your complete **Telegram Number Info API** platform with:

✅ **Public Website** - Professional landing page at `/`
✅ **Search API** - `/api/search?key=KEY&userid=ID`  
✅ **Admin Dashboard** - Management panel at `/admin`
✅ **API Key Management** - Generate, update, and revoke keys
✅ **Analytics** - Track searches, users, IPs, and response times
✅ **Rate Limiting** - Protection against abuse
✅ **MongoDB Integration** - All data persistence ready
✅ **Security** - Admin authentication, key hashing, validation

---

## 🚀 Deploy in 5 Minutes

### Step 1: Prepare Environment Variables

You need these values:

1. **MONGODB_URI** - From MongoDB Atlas
   - Go to mongodb.com/cloud/atlas
   - Create free account → Cluster → Get connection string
   - Format: `mongodb+srv://user:password@cluster.mongodb.net/telegram_number_api?retryWrites=true&w=majority`

2. **TELEGRAM_API_URL** - Your API endpoint
   - You already have: `https://tg-2-num-api-org.vercel.app/api/search`

3. **ADMIN_KEY_SALT** - Random string for security
   - Generate: `openssl rand -base64 32`
   - Or any random 32+ character string

4. **NEXT_PUBLIC_BOT_LINK** - Bot link (public)
   - Use: `https://t.me/doranuminfobot`

### Step 2: Deploy to Vercel

**Option A: GitHub Deployment (Recommended)**

```bash
# 1. Push to GitHub
git init
git add .
git commit -m "Telegram Number Info API"
git branch -M main
git remote add origin https://github.com/YOUR_USER/YOUR_REPO.git
git push -u origin main

# 2. Go to vercel.com → Import Project → Select GitHub repo
# 3. Add environment variables in Vercel dashboard
# 4. Deploy - Done!
```

**Option B: Direct Vercel CLI**

```bash
npm i -g vercel
vercel login
vercel
# Add environment variables when prompted
```

### Step 3: Create First Admin Key

After deployment is live:

```bash
# Set these in your terminal:
export MONGODB_URI="your-mongodb-uri"
export ADMIN_KEY_SALT="your-salt"

# Run the script:
node scripts/create-admin.js

# Save the key it generates!
```

### Step 4: Access Your App

- **Home Page**: `https://your-app.vercel.app`
- **Admin Dashboard**: `https://your-app.vercel.app/admin` (use the key from Step 3)
- **API Endpoint**: `https://your-app.vercel.app/api/search`

---

## 📝 Testing the API

### Get an API Key

1. Go to Admin Dashboard (`/admin`)
2. Login with your admin key
3. Go to "Generate Keys" tab
4. Set quantity: 1, daily limit: 100, expires: 30 days
5. Click "Generate Keys" → Copy the key

### Test the Search API

```bash
# Replace YOUR_KEY and TELEGRAM_USER_ID
curl "https://your-app.vercel.app/api/search?key=YOUR_KEY&userid=6187167009"

# Response:
{
  "success": true,
  "msg": "Details fetched",
  "tg_id": "6187167009",
  "country": "India",
  "country_code": "+91",
  "number": "8866361887",
  "channel": "t.me/Doraxcodes",
  "searches_remaining": 99,
  "searches_used_today": 1,
  "daily_limit": 100
}
```

---

## 📊 Admin Dashboard Features

### Statistics Tab
- View total keys, active keys, expired keys
- See total searches all-time
- View per-key detailed usage

### Generate Keys Tab
- Bulk generate multiple API keys
- Set daily limit for each key
- Set expiration period
- Copy generated keys

### Manage Keys Tab
- Update daily limits
- Change key status (active/inactive/expired)
- Update individual keys

---

## 🔌 API Integration Examples

### JavaScript/Node.js
```javascript
const apiKey = 'your-api-key';
const userId = '6187167009';

fetch(`/api/search?key=${apiKey}&userid=${userId}`)
  .then(r => r.json())
  .then(data => console.log(data));
```

### Python
```python
import requests

response = requests.get(
    'https://your-app.vercel.app/api/search',
    params={'key': 'your-api-key', 'userid': '6187167009'}
)
print(response.json())
```

### cURL
```bash
curl "https://your-app.vercel.app/api/search?key=YOUR_KEY&userid=6187167009"
```

---

## ⚠️ Important Notes

1. **Save Your Admin Key** - Keep it safe, you can't recover it
2. **MongoDB Connection** - Whitelist your Vercel IP or use 0.0.0.0/0
3. **Rate Limiting** - 100 requests/minute per API key + IP
4. **Daily Reset** - Searches reset at 00:00 UTC
5. **HTTPS Only** - Vercel provides free SSL/TLS

---

## 🆘 Troubleshooting

### "Invalid API key"
- Check the key exists in your database
- Make sure it's not expired

### "MongoDB connection error"
- Verify MONGODB_URI is correct
- Check IP whitelist in MongoDB Atlas
- Test connection with MongoDB Compass

### "Rate limit exceeded"
- Wait a few minutes before retrying
- Limit is per (key + IP) combination

### Admin dashboard won't load
- Clear browser cache
- Check admin key is correct
- Verify ADMIN_KEY_SALT environment variable

---

## 📖 Full Documentation

See these files for more details:
- **README.md** - Complete overview
- **DEPLOYMENT.md** - Detailed deployment guide
- **Database schema** - Check lib/models.ts

---

## ✅ Ready to Go!

Your API is now ready for production. Here's what to do next:

1. ✅ Set up MongoDB
2. ✅ Deploy to Vercel
3. ✅ Create admin key
4. ✅ Generate API keys for users
5. ✅ Share API endpoint with users via Telegram bot

**Questions?** Check the documentation files or the Telegram channel @Doraxcodes

Good luck! 🚀
