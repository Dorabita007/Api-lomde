# Telegram Number Info API - Deployment Guide

## 📋 Prerequisites

- MongoDB Atlas account (free tier available)
- Vercel account
- GitHub account (for easy deployment)
- Your main Telegram API endpoint (you already have this)

## 🚀 Step-by-Step Deployment

### 1. **Set Up MongoDB Atlas**

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and create a free account
2. Create a new cluster (Free tier is fine)
3. Create a database user with username and password
4. Add your IP to the network access list (or allow all IPs: 0.0.0.0/0)
5. Get your connection string:
   - Click "Connect" → "Drivers" → "Node.js"
   - Copy the connection string: `mongodb+srv://username:password@cluster.mongodb.net/mydb?retryWrites=true&w=majority`
   - Replace `<username>` and `<password>` with your credentials

### 2. **Push Code to GitHub** (Optional but Recommended)

```bash
# If you haven't already
git init
git add .
git commit -m "Initial commit: Telegram Number Info API"
git branch -M main
git remote add origin https://github.com/yourusername/your-repo-name.git
git push -u origin main
```

### 3. **Deploy to Vercel**

#### Option A: Using Vercel UI (Easiest)

1. Go to [Vercel](https://vercel.com) and sign in
2. Click "Add New" → "Project"
3. Import from GitHub (select your repository)
4. Configure project:
   - **Framework**: Next.js
   - **Environment Variables**: Add the following:
     - `MONGODB_URI`: Your MongoDB connection string
     - `TELEGRAM_API_URL`: Your main API endpoint (e.g., `https://tg-2-num-api-org.vercel.app/api/search`)
     - `ADMIN_KEY_SALT`: Generate a random string (e.g., use `openssl rand -base64 32`)
     - `NEXT_PUBLIC_BOT_LINK`: `https://t.me/doranuminfobot`
5. Click "Deploy"

#### Option B: Using Vercel CLI

```bash
# Install Vercel CLI
npm i -g vercel

# Login to Vercel
vercel login

# Deploy
vercel

# Add environment variables when prompted
# Or add them in vercel.json:
```

### 4. **Create Environment Variables in Vercel**

After deployment, go to your Vercel dashboard:

1. Select your project
2. Go to "Settings" → "Environment Variables"
3. Add these variables:

| Variable | Value | Example |
|----------|-------|---------|
| `MONGODB_URI` | Your MongoDB connection string | `mongodb+srv://user:pass@cluster.mongodb.net/telegram_number_api?retryWrites=true&w=majority` |
| `TELEGRAM_API_URL` | Your main Telegram API endpoint | `https://tg-2-num-api-org.vercel.app/api/search` |
| `ADMIN_KEY_SALT` | Random string for admin key hashing | `your-random-salt-here` |
| `NEXT_PUBLIC_BOT_LINK` | Telegram bot link (public) | `https://t.me/doranuminfobot` |

### 5. **Generate First Admin Key**

Once deployed, you need to set up your first admin key in MongoDB:

1. Go to MongoDB Atlas
2. Navigate to your database and collection `admin_keys`
3. Insert a new document:

```json
{
  "key": "your-secret-admin-key-here",
  "name": "Main Admin",
  "active": true,
  "created_at": new Date()
}
```

Or use this Node.js script to generate and store it:

```javascript
// scripts/create-admin-key.js
const crypto = require('crypto');
const { MongoClient } = require('mongodb');

async function createAdminKey() {
  const adminKey = crypto.randomBytes(32).toString('hex');
  const salt = process.env.ADMIN_KEY_SALT;
  const hashedKey = crypto.createHmac('sha256', salt).update(adminKey).digest('hex');
  
  const uri = process.env.MONGODB_URI;
  const client = new MongoClient(uri);
  
  try {
    await client.connect();
    const db = client.db('telegram_number_api');
    
    await db.collection('admin_keys').insertOne({
      key: hashedKey,
      name: 'Primary Admin',
      active: true,
      created_at: new Date()
    });
    
    console.log('✅ Admin key created successfully!');
    console.log(`📝 Use this key to login: ${adminKey}`);
    console.log('⚠️  Save this key somewhere safe - you won\'t see it again!');
    
  } finally {
    await client.close();
  }
}

createAdminKey().catch(console.error);
```

### 6. **Access Your Application**

- **Home Page**: `https://your-vercel-app.vercel.app`
- **Admin Dashboard**: `https://your-vercel-app.vercel.app/admin`
- **API Search Endpoint**: `https://your-vercel-app.vercel.app/api/search?key=YOUR_KEY&userid=USER_ID`

## 📚 API Documentation

### Public Search Endpoint

```bash
GET /api/search?key=YOUR_API_KEY&userid=TELEGRAM_USER_ID
```

**Response:**
```json
{
  "success": true,
  "msg": "Details fetched",
  "tg_id": "6187167009",
  "country": "India",
  "country_code": "+91",
  "number": "8866361887",
  "channel": "t.me/Doraxcodes",
  "searches_remaining": 95,
  "searches_used_today": 5,
  "daily_limit": 100
}
```

### Admin Endpoints

#### 1. Bulk Generate Keys

```bash
POST /api/admin/keys
Header: x-admin-key: YOUR_ADMIN_KEY
Body: {
  "action": "bulk_generate",
  "quantity": 100,
  "daily_limit": 50,
  "expires_in_days": 30
}
```

#### 2. Update Key Limits

```bash
PUT /api/admin/update
Header: x-admin-key: YOUR_ADMIN_KEY
Body: {
  "api_key": "key_to_update",
  "daily_limit": 200,
  "status": "active" // or "inactive" or "expired"
}
```

#### 3. Get Statistics

```bash
GET /api/admin/stats
Header: x-admin-key: YOUR_ADMIN_KEY
```

## 🔐 Security Recommendations

1. **Protect Your Admin Key**: Never share it publicly
2. **Use HTTPS**: Always (Vercel uses HTTPS by default)
3. **Rate Limiting**: Built-in rate limiting (100 requests per minute per IP)
4. **API Keys**: Regularly rotate admin keys
5. **Monitor Usage**: Check admin dashboard regularly for suspicious activity

## 🛠️ Troubleshooting

### MongoDB Connection Error
- Verify `MONGODB_URI` is correct
- Check that your IP is whitelisted in MongoDB Atlas
- Ensure credentials are URL-encoded (if they contain special characters)

### Admin Key Not Working
- Make sure the key is hashed in MongoDB
- Verify `ADMIN_KEY_SALT` is the same as what you generated
- Check that the admin key document has `active: true`

### Rate Limit Error
- Wait a few seconds and retry
- Check if you're exceeding 100 requests per minute
- Each API key + IP combination has its own limit

### Daily Limit Reached
- Wait until the next day (UTC reset at 00:00 UTC)
- Or use admin dashboard to increase the daily limit

## 📞 Support

If you encounter issues:
1. Check MongoDB Atlas connection
2. Verify all environment variables are set correctly
3. Check Vercel deployment logs
4. Ensure TELEGRAM_API_URL is accessible

## 🎉 You're All Set!

Your Telegram Number Info API is now live! 

### Next Steps:
- Share your API with users via the Telegram bot
- Monitor usage through the admin dashboard
- Update limits as needed
- Keep your admin key secure
