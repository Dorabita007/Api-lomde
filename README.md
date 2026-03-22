# 🚀 Telegram Number Info API

A powerful, production-ready API service for retrieving Telegram user information with comprehensive analytics, admin dashboard, and flexible API key management.

**Live Demo**: https://doraxcodes-number-api.vercel.app

## ✨ Features

- **🔑 API Key Management**: Generate, manage, and revoke API keys with custom daily limits
- **📊 Analytics Dashboard**: Track searches, unique users, response times, and IP addresses
- **🔒 Secure**: Rate limiting, admin authentication, and encrypted credentials
- **⚡ Fast**: Sub-100ms response times with MongoDB indexing
- **🎯 Flexible**: Bulk key generation, automatic expiration, and status management
- **📱 Telegram Integration**: Bot-based key requests and user management
- **🌍 Global**: Multi-region deployment ready

## 📋 Quick Start

### Prerequisites

- Node.js 18+ 
- MongoDB Atlas account (free tier available)
- Vercel account (for deployment)

### 1. Local Development

```bash
# Clone or download the repository
cd telegram-number-api

# Install dependencies
npm install

# Create .env.local file
cat > .env.local << EOF
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/telegram_number_api?retryWrites=true&w=majority
TELEGRAM_API_URL=https://tg-2-num-api-org.vercel.app/api/search
ADMIN_KEY_SALT=your-random-salt-here
NEXT_PUBLIC_BOT_LINK=https://t.me/doranuminfobot
EOF

# Run development server
npm run dev

# Open http://localhost:3000
```

### 2. Deploy to Vercel

```bash
# Option A: Using Vercel CLI
npm i -g vercel
vercel

# Option B: Connect GitHub repository in Vercel UI
# 1. Push code to GitHub
# 2. Go to vercel.com
# 3. Import your GitHub repository
# 4. Add environment variables
# 5. Deploy
```

### 3. Create Admin Key

```bash
# After deployment, set environment variables and run:
node scripts/create-admin.js

# This will generate your admin key
# Save it in a secure place - you'll need it for the admin dashboard
```

## 🔌 API Endpoints

### Public Search Endpoint

```bash
GET /api/search?key=YOUR_API_KEY&userid=TELEGRAM_USER_ID
```

**Example:**
```bash
curl "https://your-app.vercel.app/api/search?key=key_abc123&userid=6187167009"
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

#### Generate API Keys (Bulk)

```bash
POST /api/admin/keys
Header: x-admin-key: YOUR_ADMIN_KEY
Content-Type: application/json

{
  "action": "bulk_generate",
  "quantity": 100,
  "daily_limit": 50,
  "expires_in_days": 30
}
```

#### Update Key Settings

```bash
PUT /api/admin/update
Header: x-admin-key: YOUR_ADMIN_KEY
Content-Type: application/json

{
  "api_key": "key_to_update",
  "daily_limit": 200,
  "status": "active"
}
```

#### Get Statistics

```bash
GET /api/admin/stats
Header: x-admin-key: YOUR_ADMIN_KEY
```

**Response includes:**
- Total keys, active keys, expired keys
- Total searches all-time
- Per-key statistics (searches, unique users, creation date, etc.)

## 🎯 Admin Dashboard

Access the admin dashboard at: `/admin`

**Features:**
- 📊 View all statistics and key metrics
- 🔑 Bulk generate new API keys
- ⚙️ Manage existing keys (update limits, change status)
- 📈 Track usage per key and per day
- 🕐 Monitor last search times and response metrics

## 🗄️ Database Schema

### Collections

#### `api_keys`
```javascript
{
  key: String,           // Unique API key
  daily_limit: Number,   // Search limit per day
  searches_today: Number,
  last_reset: Date,
  status: String,        // 'active', 'inactive', 'expired'
  created_by: String,    // 'admin' or 'bulk_generation'
  created_at: Date,
  expires_at: Date       // Optional expiration date
}
```

#### `search_history`
```javascript
{
  api_key: String,
  user_id: String,
  ip_address: String,
  response_time: Number, // in milliseconds
  country: String,
  country_code: String,
  number: String,
  status: String,        // 'success' or 'failed'
  error: String,         // If failed
  created_at: Date
}
```

#### `admin_keys`
```javascript
{
  key: String,    // Hashed admin key
  name: String,
  active: Boolean,
  created_at: Date
}
```

#### `key_statistics`
```javascript
{
  api_key: String,
  daily_searches: Object,        // { "2024-01-15": 42, ... }
  searched_user_ids: [String],
  total_searches: Number,
  active_days: Number,
  status: String,
  created_at: Date,
  updated_at: Date
}
```

#### `rate_limits`
```javascript
{
  key: String,    // "api_key:ip_address"
  count: Number,
  resetAt: Date
}
```

## 🔐 Security

- **API Key Validation**: All requests verified against database
- **Rate Limiting**: 100 requests per minute per API key + IP combination
- **Admin Authentication**: Hashed admin keys with HMAC-SHA256
- **Daily Limits**: Automatic reset at 00:00 UTC
- **Key Expiration**: Automatic status update when keys expire
- **IP Tracking**: All requests logged with IP addresses
- **Input Validation**: All parameters validated before processing

## 📊 Monitoring & Analytics

The admin dashboard provides:

- **Real-time Stats**: Active keys, total searches, system health
- **Per-Key Analytics**: Individual key usage patterns
- **Search History**: Detailed logs of all API requests
- **User Tracking**: Unique users searched per key
- **Response Times**: Performance metrics for all searches
- **Geographic Data**: Countries and regions of searches

## 🚀 Scaling & Performance

### Optimizations

- MongoDB indexes on: `key`, `api_key`, `created_at`, `user_id`
- Connection pooling with MongoDB
- Response time tracking for performance monitoring
- Automatic rate limit cleanup for expired entries
- Efficient aggregation queries for analytics

### For High Traffic

1. Enable MongoDB sharding on `api_keys` and `search_history`
2. Add caching layer (Redis) for frequently accessed keys
3. Use CDN for static assets
4. Consider API Gateway for additional rate limiting

## 🛠️ Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `MONGODB_URI` | MongoDB connection string | `mongodb+srv://user:pass@cluster.mongodb.net/db` |
| `TELEGRAM_API_URL` | Your Telegram API endpoint | `https://api.example.com/search` |
| `ADMIN_KEY_SALT` | Salt for hashing admin keys | `random-secret-string` |
| `NEXT_PUBLIC_BOT_LINK` | Telegram bot link (public) | `https://t.me/botname` |

## 📖 Full Deployment Guide

See [DEPLOYMENT.md](./DEPLOYMENT.md) for detailed step-by-step deployment instructions.

## 🤝 Support & Contact

- 📱 Telegram Channel: [@Doraxcodes](https://t.me/Doraxcodes)
- 👨‍💻 Developer: [@talktodorabot](https://t.me/talktodorabot)
- 🤖 Bot Link: [@doranuminfobot](https://t.me/doranuminfobot)

## 📄 License

This project is proprietary. All rights reserved.

## 🎉 Ready to Deploy?

1. Set up MongoDB Atlas account
2. Create environment variables
3. Deploy to Vercel
4. Run `node scripts/create-admin.js` to create your first admin key
5. Access `/admin` dashboard and start managing keys

---

**Made with ❤️ by Doraxcodes**
