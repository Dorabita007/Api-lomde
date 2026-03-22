# 🧪 API Testing Guide

## Testing the Complete API System

This guide shows you how to test every endpoint and feature of your Telegram Number Info API.

---

## 1️⃣ Local Development Testing

### Start the Dev Server

```bash
npm install
npm run dev
```

Your app will be at `http://localhost:3000`

### Access URLs

- Home Page: http://localhost:3000
- Admin Dashboard: http://localhost:3000/admin
- Search API: http://localhost:3000/api/search

---

## 2️⃣ Testing Admin Key Creation

### Before Deployment

Set environment variables locally:

```bash
export MONGODB_URI="mongodb+srv://user:password@cluster.mongodb.net/db"
export ADMIN_KEY_SALT="test-salt-string-32-chars-minimum"
```

### Run Admin Key Script

```bash
node scripts/create-admin.js
```

Output example:
```
🔐 Telegram Number Info API - Admin Key Generator

📝 Admin Key Details:
   Raw Key (use for login): a3f8d9e2c1b4f6a8e5d2c9f1b8a4e7d2
   Hashed Key (stored in DB): 7f2a8e9c1d5b3a6f4e8c2b9a1f7d5c3e

Continue and create this admin key? (yes/no): yes
✅ Connected to MongoDB
✅ Admin key created successfully!
   ID: 507f1f77bcf86cd799439011

📌 IMPORTANT - Save your key:
   Raw Key: a3f8d9e2c1b4f6a8e5d2c9f1b8a4e7d2
   ⚠️  This is your login key. Save it in a secure place!
```

---

## 3️⃣ Testing Admin Dashboard

### 1. Login to Admin Dashboard

URL: `http://localhost:3000/admin` (or production URL)

**Form:**
- Admin Key: `a3f8d9e2c1b4f6a8e5d2c9f1b8a4e7d2` (the key from above)
- Click "Login"

**Expected Result:**
- Dashboard loads with Statistics tab
- Shows total keys (likely 0 initially)
- No errors

### 2. Generate Test API Keys

**Navigate to:** "Generate Keys" tab

**Form:**
```
Quantity: 5
Daily Limit: 100
Expires In (Days): 30
```

**Click:** "Generate Keys"

**Expected Result:**
```
Generated Keys:
key_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p
key_2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p1
key_3c4d5e6f7g8h9i0j1k2l3m4n5o6p1q2
key_4d5e6f7g8h9i0j1k2l3m4n5o6p1q2r3
key_5e6f7g8h9i0j1k2l3m4n5o6p1q2r3s4
```

- Copy one key for testing
- Each can be used for 100 searches/day

### 3. View Statistics

**Navigate to:** "Statistics" tab

**Expected:**
```
Total Keys: 5
Active Keys: 5
Expired Keys: 0
Inactive Keys: 0
Total Searches: 0

[Table with all generated keys showing status, limits, searches]
```

### 4. Manage Keys

**Navigate to:** "Manage Keys" tab

**Update a Key:**
```
API Key: key_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p
New Daily Limit: 500
Status: active
```

**Click:** "Update Key"

**Expected:**
```
✅ API key updated successfully!
daily_limit changed: 100 → 500
```

Go back to Stats → table should show updated limit

---

## 4️⃣ Testing Search API Endpoint

### Using cURL

```bash
# Basic test
curl "http://localhost:3000/api/search?key=key_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p&userid=6187167009"
```

**Expected Response:**
```json
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

### Using JavaScript

```javascript
// Test 1: Valid request
const response = await fetch(
  'http://localhost:3000/api/search?key=key_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p&userid=6187167009'
);
const data = await response.json();
console.log(data); // Should be success: true
```

### Using Python

```python
import requests

response = requests.get(
    'http://localhost:3000/api/search',
    params={
        'key': 'key_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p',
        'userid': '6187167009'
    }
)

print(response.status_code)  # 200
print(response.json())       # Full response
```

---

## 5️⃣ Error Handling Tests

### Test 1: Missing Parameters

```bash
curl "http://localhost:3000/api/search"
```

**Expected:**
```json
{
  "success": false,
  "msg": "Missing key or userid parameter"
}
```
**Status:** 400

### Test 2: Invalid API Key

```bash
curl "http://localhost:3000/api/search?key=invalid_key&userid=6187167009"
```

**Expected:**
```json
{
  "success": false,
  "msg": "Invalid API key"
}
```
**Status:** 401

### Test 3: Inactive Key

1. In admin dashboard, set a key to "inactive"
2. Test it:

```bash
curl "http://localhost:3000/api/search?key=inactive_key&userid=6187167009"
```

**Expected:**
```json
{
  "success": false,
  "msg": "API key is not active"
}
```
**Status:** 403

### Test 4: Daily Limit Exceeded

1. Set a key with `daily_limit: 1`
2. Make 2 requests:

```bash
# First request
curl "http://localhost:3000/api/search?key=limited_key&userid=6187167009"

# Second request (should fail)
curl "http://localhost:3000/api/search?key=limited_key&userid=6187167010"
```

**First Response:** 200, success: true
**Second Response:** 429, "Daily search limit reached"

### Test 5: Rate Limiting

Make 101 requests in quick succession:

```bash
for i in {1..101}; do
  curl "http://localhost:3000/api/search?key=key_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p&userid=617870000$i"
done
```

**Expected:** 
- First 100: 200/429 (depending on daily limit)
- 101st: 429, "Rate limit exceeded"

---

## 6️⃣ Testing Admin Endpoints

### Generate Keys (Bulk)

```bash
curl -X POST http://localhost:3000/api/admin/keys \
  -H "x-admin-key: a3f8d9e2c1b4f6a8e5d2c9f1b8a4e7d2" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "bulk_generate",
    "quantity": 50,
    "daily_limit": 200,
    "expires_in_days": 60
  }'
```

**Expected:**
```json
{
  "success": true,
  "msg": "Keys generated successfully",
  "generated_keys": ["key_...", "key_...", ...],
  "count": 50,
  "daily_limit": 200
}
```

### Update Key

```bash
curl -X PUT http://localhost:3000/api/admin/update \
  -H "x-admin-key: a3f8d9e2c1b4f6a8e5d2c9f1b8a4e7d2" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "key_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p",
    "daily_limit": 1000,
    "status": "active"
  }'
```

**Expected:**
```json
{
  "success": true,
  "msg": "API key updated successfully",
  "api_key": "key_1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p",
  "updates": {
    "daily_limit": 1000,
    "status": "active"
  }
}
```

### Get Statistics

```bash
curl http://localhost:3000/api/admin/stats \
  -H "x-admin-key: a3f8d9e2c1b4f6a8e5d2c9f1b8a4e7d2"
```

**Expected:**
```json
{
  "success": true,
  "msg": "Statistics fetched",
  "total_keys": 55,
  "active_keys": 55,
  "expired_keys": 0,
  "inactive_keys": 0,
  "total_searches_all_time": 42,
  "keys": [
    {
      "key": "key_1a2b...",
      "status": "active",
      "daily_limit": 1000,
      "searches_today": 5,
      "total_searches": 12,
      "unique_users": 8,
      "created_at": "2024-01-15T10:30:00Z",
      ...
    }
  ]
}
```

### Invalid Admin Key

```bash
curl http://localhost:3000/api/admin/stats \
  -H "x-admin-key: wrong-admin-key"
```

**Expected:**
```json
{
  "success": false,
  "msg": "Invalid admin key"
}
```
**Status:** 401

---

## 7️⃣ Database Verification

### Check in MongoDB

1. Go to MongoDB Atlas dashboard
2. Browse Collections
3. Verify data in:
   - `api_keys` - Should show generated keys
   - `search_history` - Should show search requests
   - `admin_keys` - Should show your admin key (hashed)
   - `rate_limits` - Should show recent rate limit entries
   - `key_statistics` - Should show aggregated stats

---

## 8️⃣ Performance Testing

### Response Time Check

```bash
time curl "http://localhost:3000/api/search?key=key_abc&userid=6187167009"
```

**Expected:** < 100ms for successful requests

### Concurrent Requests

```bash
# Test with 10 concurrent requests
for i in {1..10}; do
  curl "http://localhost:3000/api/search?key=key_abc&userid=618716700$i" &
done
wait
```

**Expected:** All complete successfully

---

## 9️⃣ Deployment Testing Checklist

Before deploying to production:

- [ ] ✅ Admin dashboard works
- [ ] ✅ Generate keys works
- [ ] ✅ Search API returns correct format
- [ ] ✅ Rate limiting prevents abuse
- [ ] ✅ Daily limits reset properly
- [ ] ✅ Invalid keys are rejected
- [ ] ✅ Admin stats are accurate
- [ ] ✅ Database is properly indexed
- [ ] ✅ All environment variables set
- [ ] ✅ CORS headers (if needed)

---

## 🔟 Production Testing

### Test Vercel Deployment

After deploying to Vercel:

```bash
# Replace with your Vercel URL
PROD_URL="https://your-app.vercel.app"

# Test home page
curl $PROD_URL

# Test admin dashboard
curl $PROD_URL/admin

# Test API
curl "$PROD_URL/api/search?key=test_key&userid=6187167009"
```

### Monitor in Vercel Dashboard

1. Go to vercel.com dashboard
2. Select your project
3. View:
   - Function logs
   - Database queries
   - Error rates
   - Response times

---

## 📋 Test Result Template

```
Date: 2024-01-15
Tester: Your Name
Environment: Development/Production

✅ Admin Login: PASS
✅ Generate Keys: PASS
✅ View Statistics: PASS
✅ Update Keys: PASS
✅ Search API: PASS (avg response: 45ms)
✅ Error Handling: PASS
✅ Rate Limiting: PASS
✅ Daily Limits: PASS
✅ Database: PASS

Notes: All tests passed successfully
```

---

## 🎯 Automation Testing

### Node.js Test Suite

```javascript
// test.js
const BASE_URL = 'http://localhost:3000';

async function testAPI() {
  console.log('Testing Search API...');
  
  const res = await fetch(
    `${BASE_URL}/api/search?key=test_key&userid=6187167009`
  );
  const data = await res.json();
  
  if (data.success) {
    console.log('✅ Search API works');
  } else {
    console.log('❌ Search API failed:', data.msg);
  }
}

testAPI();
```

Run with: `node test.js`

---

## ✅ You're Ready!

All tests passing? Your API is production-ready.

Deploy with confidence!
