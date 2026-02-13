# Production Deployment Guide - Farmly AI

This guide provides step-by-step instructions for deploying Farmly AI to production environments.

## Table of Contents
1. [Pre-Deployment Checklist](#pre-deployment-checklist)
2. [Frontend Deployment (Vercel)](#frontend-deployment-vercel)
3. [Backend API Deployment (Vercel)](#backend-api-deployment-vercel)
4. [ML Service Deployment (Railway)](#ml-service-deployment-railway)
5. [Database Production Setup](#database-production-setup)
6. [Environment Variables Configuration](#environment-variables-configuration)
7. [Health Checks and Monitoring](#health-checks-and-monitoring)
8. [Post-Deployment Verification](#post-deployment-verification)
9. [Production URLs](#production-urls)

---

## Pre-Deployment Checklist

Before deploying to production, ensure all these items are completed:

### Code Quality
- [x] All TypeScript errors resolved (`pnpm run typecheck`)
- [x] All ESLint warnings fixed (`pnpm run lint`)
- [x] All translations complete (`pnpm run i18n:check`)
- [x] Demo data seeded (`pnpm run seed:demo`)
- [x] Government schemes seeded (`pnpm run seed:schemes`)

### Testing
- [x] End-to-end testing completed
- [x] All 6 languages tested
- [x] Mobile responsiveness verified (iOS Safari, Android Chrome)
- [x] PWA functionality verified
- [x] Offline mode tested
- [x] Disease detection tested with sample images
- [x] Voice interface tested in regional languages

### Environment Setup
- [x] MongoDB Atlas production cluster created
- [x] Redis Cloud production instance created
- [x] Vercel account ready
- [x] Railway/Render account ready
- [x] Google Cloud credentials obtained
- [x] Twilio account configured
- [x] OpenWeatherMap API key obtained
- [x] Gemini API key obtained

---

## Frontend Deployment (Vercel)

### Step 1: Prepare Frontend Build

```bash
cd frontend
pnpm install
pnpm run typecheck  # Must pass
pnpm run lint       # Must pass
pnpm run build      # Test production build
```

**Verify build output:**
- Check `dist/` directory created
- Verify PWA manifest and service worker generated
- Check bundle sizes: `dist/assets/*.js` should be <500KB gzipped

### Step 2: Configure Environment Variables

Create production environment file:

**frontend/.env.production**
```bash
VITE_API_URL=https://farmly-ai.vercel.app
VITE_ML_API_URL=https://farmly-ai-ml.up.railway.app
```

### Step 3: Deploy via Vercel CLI

```bash
# Install Vercel CLI globally
npm install -g vercel

# Login to Vercel
vercel login

# Deploy frontend
cd frontend
vercel --prod

# Or deploy entire project from root
cd ..
vercel --prod
```

### Step 4: Configure Vercel Project Settings

1. Go to [Vercel Dashboard](https://vercel.com/dashboard)
2. Select **farmly-ai** project
3. **Settings** → **General**
   - Framework Preset: **Vite**
   - Build Command: `cd frontend && pnpm run build`
   - Output Directory: `frontend/dist`
   - Install Command: `pnpm install`

4. **Settings** → **Domains**
   - Add custom domain (optional): `farmly-ai.vercel.app`
   - Enable HTTPS (automatic)

5. **Settings** → **Caching**
   - Enable **Edge Caching**
   - Cache control headers: `max-age=31536000, immutable` (for assets)

---

## Backend API Deployment (Vercel)

### Step 1: Prepare Backend Build

```bash
cd backend
pnpm install
pnpm run typecheck  # Must pass
pnpm run lint       # Must pass
pnpm run build      # Compile TypeScript to JavaScript
```

### Step 2: Verify Vercel Configuration

Ensure `vercel.json` in project root is configured correctly:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "frontend/package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "dist"
      }
    },
    {
      "src": "backend/src/index.ts",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "backend/src/index.ts"
    },
    {
      "src": "/(.*)",
      "dest": "frontend/$1"
    }
  ],
  "env": {
    "NODE_ENV": "production"
  }
}
```

### Step 3: Upload Google Cloud Credentials

**For Vercel Serverless Functions:**

Google Cloud credentials JSON needs to be converted to environment variable:

```bash
# Read the JSON file and base64 encode it
cat backend/credentials/google-cloud-key.json | base64
```

Add to Vercel environment variables as `GOOGLE_CLOUD_CREDENTIALS_BASE64`

Update backend code to decode:
```typescript
// backend/src/utils/google-cloud.ts
const credentials = process.env.GOOGLE_CLOUD_CREDENTIALS_BASE64
  ? JSON.parse(Buffer.from(process.env.GOOGLE_CLOUD_CREDENTIALS_BASE64, 'base64').toString('utf-8'))
  : require(process.env.GOOGLE_CLOUD_CREDENTIALS);
```

### Step 4: Configure Backend Environment Variables in Vercel

Go to **Project Settings** → **Environment Variables** and add all from `backend/.env.example`:

**Required Variables:**
```bash
NODE_ENV=production
PORT=4000
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/farmly_ai
REDIS_URL=redis://default:password@redis-host:port
JWT_SECRET=<generated-secret>
JWT_REFRESH_SECRET=<generated-secret>
JWT_EXPIRES_IN=7d
JWT_REFRESH_EXPIRES_IN=30d
TWILIO_ACCOUNT_SID=ACxxxxx
TWILIO_AUTH_TOKEN=xxxxxx
TWILIO_PHONE_NUMBER=+1234567890
GOOGLE_CLOUD_PROJECT_ID=your-project-id
GOOGLE_CLOUD_CREDENTIALS_BASE64=<base64-encoded-json>
OPENWEATHER_API_KEY=xxxxxx
GEMINI_API_KEY=xxxxxx
ML_SERVICE_URL=https://farmly-ai-ml.up.railway.app
BLOB_READ_WRITE_TOKEN=vercel_blob_xxxxxx
DEMO_MODE=false
```

**For each variable:**
- Select environments: **Production**, **Preview**, **Development**
- Click **Save**

### Step 5: Enable CORS and Rate Limiting

Ensure `backend/src/index.ts` has production CORS configuration:

```typescript
app.use(cors({
  origin: process.env.NODE_ENV === 'production' 
    ? ['https://farmly-ai.vercel.app']
    : '*',
  credentials: true
}));
```

---

## ML Service Deployment (Railway)

### Option A: Railway.app (Recommended)

#### Step 1: Prepare ML Service

```bash
cd ml-service
pip install -r requirements.txt
python test_detection.py  # Verify model works locally
```

#### Step 2: Login to Railway

1. Go to [Railway.app](https://railway.app/)
2. Sign in with GitHub
3. Authorize Railway to access your repository

#### Step 3: Create New Project

1. Click **"New Project"**
2. Select **"Deploy from GitHub repo"**
3. Choose **farmly-ai-7a37** repository
4. Railway detects Python project

#### Step 4: Configure Service

1. **Root Directory**: `ml-service`
2. **Build Command**: (auto-detected from requirements.txt)
   ```bash
   pip install -r requirements.txt
   ```
3. **Start Command**:
   ```bash
   uvicorn app.main:app --host 0.0.0.0 --port $PORT
   ```
4. **Health Check Path**: `/health`

#### Step 5: Set Environment Variables

In Railway dashboard, add variables:
```bash
PORT=8000
PYTHONUNBUFFERED=1
GEMINI_API_KEY=your_gemini_api_key
DEVICE=cpu
```

#### Step 6: Deploy

1. Railway automatically deploys on push to main branch
2. Wait for deployment (~3-5 minutes)
3. Check deployment logs for errors

#### Step 7: Get Public URL

1. Go to **Settings** → **Domains**
2. Click **"Generate Domain"**
3. Copy URL: `https://farmly-ai-ml-production.up.railway.app`
4. Test health endpoint:
   ```bash
   curl https://farmly-ai-ml-production.up.railway.app/health
   ```

#### Step 8: Update Backend ML_SERVICE_URL

1. Go to Vercel project → **Environment Variables**
2. Update `ML_SERVICE_URL` to Railway URL
3. Redeploy backend

### Option B: Render.com

Use the provided `ml-service/render.yaml` configuration:

1. Go to [Render.com](https://render.com/)
2. **New +** → **Blueprint**
3. Connect GitHub repository
4. Render auto-deploys using `render.yaml`
5. Get URL from dashboard
6. Update `ML_SERVICE_URL` in Vercel

---

## Database Production Setup

### MongoDB Atlas Production Configuration

#### Step 1: Review Cluster Configuration

1. Go to [MongoDB Atlas](https://cloud.mongodb.com/)
2. Select **farmly-ai-cluster**
3. Verify:
   - Cluster Tier: **M0 (Free)** or upgrade to **M2** for better performance
   - Region: **ap-south-1 (Mumbai)** for India-based app
   - Backup: **Enabled** (automatic point-in-time backups)

#### Step 2: Setup Database Indexes

Indexes should be created automatically by `init:db` script, but verify:

```bash
cd backend
pnpm run init:db
```

**Required Indexes:**
- `users.phoneNumber` (unique)
- `users.farmProfile.location` (2dsphere for geospatial queries)
- `diseaseDetections.userId`
- `queries.userId`
- `schemes.eligibility.state`

**Verify in MongoDB Atlas:**
1. **Database** → **Collections**
2. Click on each collection
3. **Indexes** tab
4. Should see indexes listed

#### Step 3: Configure IP Whitelist

For production security:
1. **Network Access** → **IP Access List**
2. Remove `0.0.0.0/0` if added during development
3. Add Vercel IP ranges:
   ```
   76.76.21.0/24
   76.223.126.0/24
   ```
4. Add Railway IP ranges (or allow all for Railway free tier)

#### Step 4: Enable Database Monitoring

1. **Metrics** tab
2. Set up alerts for:
   - Connection spikes (>80% of limit)
   - Storage usage (>80% of 512MB for M0)
   - Query performance (slow queries >100ms)

### Redis Cloud Production Configuration

#### Step 1: Verify Redis Instance

1. Go to [Redis Cloud](https://redis.com/)
2. Select **farmly-ai-cache** database
3. Verify:
   - Plan: **30MB Free** or upgrade to **100MB** ($5/month)
   - Region: **ap-south-1 (Mumbai)**
   - High Availability: Enabled (if paid plan)

#### Step 2: Configure TTL Values

Ensure proper cache expiration in `backend/src/services/*.service.ts`:

```typescript
// Weather cache: 1 hour
await redis.setex(`weather:${lat},${lon}`, 3600, JSON.stringify(weatherData));

// Market prices cache: 6 hours
await redis.setex(`market:${crop}:${location}`, 21600, JSON.stringify(prices));

// OTP cache: 10 minutes
await redis.setex(`otp:${phoneNumber}`, 600, otp);
```

#### Step 3: Monitor Memory Usage

1. **Metrics** tab
2. Set alert for memory usage >80%
3. If exceeded, consider upgrading to 100MB plan

---

## Environment Variables Configuration

### Generate Secure Secrets

```bash
# JWT Secret
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"

# JWT Refresh Secret
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

### Complete Environment Variables Checklist

#### Vercel Environment Variables (Backend)

| Variable | Source | Production Value |
|----------|--------|-----------------|
| `NODE_ENV` | Manual | `production` |
| `PORT` | Manual | `4000` |
| `MONGODB_URI` | MongoDB Atlas | `mongodb+srv://...` |
| `REDIS_URL` | Redis Cloud | `redis://default:...` |
| `JWT_SECRET` | Generated | `<base64-secret>` |
| `JWT_REFRESH_SECRET` | Generated | `<base64-secret>` |
| `JWT_EXPIRES_IN` | Manual | `7d` |
| `JWT_REFRESH_EXPIRES_IN` | Manual | `30d` |
| `TWILIO_ACCOUNT_SID` | Twilio Console | `ACxxxxx` |
| `TWILIO_AUTH_TOKEN` | Twilio Console | `xxxxxx` |
| `TWILIO_PHONE_NUMBER` | Twilio Console | `+1234567890` |
| `GOOGLE_CLOUD_PROJECT_ID` | Google Cloud Console | `farmly-ai-xxxxx` |
| `GOOGLE_CLOUD_CREDENTIALS_BASE64` | Service Account JSON (base64) | `<base64-json>` |
| `OPENWEATHER_API_KEY` | OpenWeatherMap | `xxxxxx` |
| `GEMINI_API_KEY` | Google AI Studio | `xxxxxx` |
| `ML_SERVICE_URL` | Railway/Render | `https://farmly-ai-ml...` |
| `BLOB_READ_WRITE_TOKEN` | Vercel Blob Storage | `vercel_blob_xxxxx` |
| `DEMO_MODE` | Manual | `false` (IMPORTANT!) |

#### Railway Environment Variables (ML Service)

| Variable | Value |
|----------|-------|
| `PORT` | `8000` |
| `PYTHONUNBUFFERED` | `1` |
| `GEMINI_API_KEY` | `<your-key>` |
| `DEVICE` | `cpu` |

---

## Health Checks and Monitoring

### Setup Health Check Endpoints

Ensure these endpoints exist and return proper responses:

#### Backend Health Check

**Endpoint:** `GET /api/health`

**Expected Response:**
```json
{
  "status": "ok",
  "timestamp": "2026-02-13T10:00:00.000Z",
  "db": "connected",
  "redis": "connected",
  "services": {
    "mongodb": "healthy",
    "redis": "healthy",
    "mlService": "healthy"
  }
}
```

**Implementation in `backend/src/routes/health.routes.ts`:**
```typescript
import { Router } from 'express';
import mongoose from 'mongoose';
import { getRedisClient } from '../utils/redis';
import axios from 'axios';

const router = Router();

router.get('/health', async (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    db: mongoose.connection.readyState === 1 ? 'connected' : 'disconnected',
    redis: 'unknown',
    services: {
      mongodb: 'unknown',
      redis: 'unknown',
      mlService: 'unknown'
    }
  };

  try {
    // Check Redis
    const redis = getRedisClient();
    await redis.ping();
    health.redis = 'connected';
    health.services.redis = 'healthy';
  } catch (error) {
    health.redis = 'disconnected';
    health.services.redis = 'unhealthy';
    health.status = 'degraded';
  }

  try {
    // Check MongoDB
    if (mongoose.connection.readyState === 1) {
      await mongoose.connection.db.admin().ping();
      health.services.mongodb = 'healthy';
    } else {
      health.services.mongodb = 'unhealthy';
      health.status = 'degraded';
    }
  } catch (error) {
    health.services.mongodb = 'unhealthy';
    health.status = 'degraded';
  }

  try {
    // Check ML Service
    const mlServiceUrl = process.env.ML_SERVICE_URL;
    if (mlServiceUrl) {
      const response = await axios.get(`${mlServiceUrl}/health`, { timeout: 5000 });
      health.services.mlService = response.data.status === 'ok' ? 'healthy' : 'unhealthy';
    }
  } catch (error) {
    health.services.mlService = 'unhealthy';
    health.status = 'degraded';
  }

  const statusCode = health.status === 'ok' ? 200 : 503;
  res.status(statusCode).json(health);
});

export default router;
```

#### ML Service Health Check

**Endpoint:** `GET /health`

**Expected Response:**
```json
{
  "status": "ok",
  "model_loaded": true,
  "version": "1.0.0",
  "timestamp": "2026-02-13T10:00:00.000Z"
}
```

**Verify in `ml-service/app/main.py`:**
```python
@app.get("/health")
async def health_check():
    return {
        "status": "ok",
        "model_loaded": disease_detector is not None,
        "version": "1.0.0",
        "timestamp": datetime.utcnow().isoformat()
    }
```

### Setup Uptime Monitoring

Use free services to monitor production endpoints:

#### Option 1: UptimeRobot (Recommended)

1. Sign up at [UptimeRobot.com](https://uptimerobot.com/)
2. Add monitors:
   - **Frontend**: `https://farmly-ai.vercel.app`
   - **Backend API**: `https://farmly-ai.vercel.app/api/health`
   - **ML Service**: `https://farmly-ai-ml.up.railway.app/health`
3. Set check interval: **5 minutes**
4. Configure alerts:
   - Email notifications
   - SMS notifications (optional)

#### Option 2: Better Uptime

1. Sign up at [BetterUptime.com](https://betteruptime.com/)
2. Create monitors with same URLs
3. Set up status page (optional for hackathon judges)

### Error Logging with Winston

Ensure Winston is configured for production in `backend/src/utils/logger.ts`:

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console({
      format: winston.format.simple()
    })
  ]
});

if (process.env.NODE_ENV === 'production') {
  // Add file transport for production
  logger.add(new winston.transports.File({ 
    filename: 'error.log', 
    level: 'error' 
  }));
  logger.add(new winston.transports.File({ 
    filename: 'combined.log' 
  }));
}

export default logger;
```

---

## Post-Deployment Verification

### Complete Production Testing Checklist

Run these tests after deployment:

#### 1. API Health Checks

```bash
# Backend health
curl https://farmly-ai.vercel.app/api/health
# Expected: {"status":"ok","db":"connected","redis":"connected"}

# ML service health
curl https://farmly-ai-ml.up.railway.app/health
# Expected: {"status":"ok","model_loaded":true}
```

#### 2. Authentication Flow

```bash
# Send OTP
curl -X POST https://farmly-ai.vercel.app/api/auth/send-otp \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "+919876543210"}'
# Expected: {"success":true,"message":"OTP sent"}

# Verify OTP (use demo account in demo mode)
curl -X POST https://farmly-ai.vercel.app/api/auth/verify-otp \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "+919876543210", "otp": "123456"}'
# Expected: {"success":true,"token":"eyJhbGciOi..."}
```

#### 3. Disease Detection Test

```bash
# Get auth token first from above
TOKEN="<your-jwt-token>"

# Test disease detection with sample image
curl -X POST https://farmly-ai.vercel.app/api/disease/detect \
  -H "Authorization: Bearer $TOKEN" \
  -F "image=@./test-images/tomato-late-blight.jpg" \
  -F "language=hi"
# Expected: Disease prediction with recommendations
```

#### 4. Market Prices Test

```bash
curl "https://farmly-ai.vercel.app/api/market/prices?crop=tomato&language=hi" \
  -H "Authorization: Bearer $TOKEN"
# Expected: Market prices from nearby mandis
```

#### 5. Voice Query Test

```bash
curl -X POST https://farmly-ai.vercel.app/api/query/voice \
  -H "Authorization: Bearer $TOKEN" \
  -F "audio=@./test-audio/hindi-query.wav" \
  -F "language=hi"
# Expected: Transcription and AI response
```

#### 6. Government Schemes Test

```bash
curl "https://farmly-ai.vercel.app/api/schemes/match?language=ta" \
  -H "Authorization: Bearer $TOKEN"
# Expected: List of eligible schemes
```

#### 7. Advisory Recommendations Test

```bash
curl "https://farmly-ai.vercel.app/api/advisory/recommendations?language=hi" \
  -H "Authorization: Bearer $TOKEN"
# Expected: Personalized farm advisory
```

#### 8. Frontend Load Test

1. Open `https://farmly-ai.vercel.app` in browser
2. Check:
   - [ ] Page loads in <3 seconds
   - [ ] PWA install prompt appears
   - [ ] All images load correctly
   - [ ] No console errors
   - [ ] Responsive on mobile (test with DevTools)

#### 9. PWA Test

1. Open site on mobile device
2. Tap "Add to Home Screen"
3. Open PWA from home screen
4. Verify:
   - [ ] Standalone mode (no browser UI)
   - [ ] Splash screen shows
   - [ ] Icons correct
   - [ ] Offline mode works (airplane mode test)

#### 10. Language Test

Test each language:
- [ ] Hindi (hi)
- [ ] Tamil (ta)
- [ ] Malayalam (ml)
- [ ] Telugu (te)
- [ ] Kannada (kn)
- [ ] English (en)

Verify all UI text translated correctly.

#### 11. Performance Test

Run Lighthouse audit:
```bash
npx lighthouse https://farmly-ai.vercel.app --view
```

**Target Scores:**
- Performance: >80
- Accessibility: >90
- Best Practices: >90
- SEO: >90
- PWA: 100

#### 12. Load Test (Optional)

Use Apache Bench or Artillery to test load:
```bash
# Install Apache Bench
# On macOS: brew install httpd
# On Ubuntu: apt-get install apache2-utils

# Test 100 concurrent requests
ab -n 100 -c 10 https://farmly-ai.vercel.app/api/health
```

**Expected:** No errors, average response time <2s

---

## Production URLs

### Live Application URLs

| Service | URL | Description |
|---------|-----|-------------|
| **Frontend** | https://farmly-ai.vercel.app | Main PWA application |
| **Backend API** | https://farmly-ai.vercel.app/api | REST API endpoints |
| **ML Service** | https://farmly-ai-ml.up.railway.app | Disease detection service |
| **Health Check** | https://farmly-ai.vercel.app/api/health | Backend health status |
| **ML Health Check** | https://farmly-ai-ml.up.railway.app/health | ML service health |

### Demo Credentials

For hackathon judges and demo purposes:

| Language | Phone Number | OTP (if DEMO_MODE=true) |
|----------|--------------|------------------------|
| Hindi | +919876543210 | 123456 |
| Tamil | +919876543211 | 123456 |
| Malayalam | +919876543212 | 123456 |
| Telugu | +919876543213 | 123456 |
| Kannada | +919876543214 | 123456 |

**Note:** For production demo, set `DEMO_MODE=true` in Vercel environment variables to bypass SMS and use fixed OTP.

### QR Code for Mobile Testing

Generate QR code for easy mobile access:
```bash
npx qrcode-terminal https://farmly-ai.vercel.app
```

Or use online generator: https://www.qr-code-generator.com/

---

## Deployment Commands Summary

### Quick Deploy (All Services)

```bash
# 1. Deploy Frontend + Backend to Vercel
vercel --prod

# 2. Deploy ML Service to Railway (via GitHub push)
git push origin main
# Railway auto-deploys on push

# 3. Verify all services
curl https://farmly-ai.vercel.app/api/health
curl https://farmly-ai-ml.up.railway.app/health

# 4. Test frontend
open https://farmly-ai.vercel.app
```

### Rollback Procedure

If deployment fails:

```bash
# Vercel: Rollback to previous deployment
vercel rollback

# Railway: Rollback via dashboard
# 1. Go to Railway dashboard
# 2. Deployments tab
# 3. Click on previous successful deployment
# 4. Click "Redeploy"
```

---

## Production Monitoring Dashboard

### Key Metrics to Monitor

| Metric | Tool | Alert Threshold |
|--------|------|-----------------|
| API Response Time | Vercel Analytics | p95 >2s |
| Error Rate | Winston Logs | >1% of requests |
| Database Connections | MongoDB Atlas | >80% of limit |
| Redis Memory Usage | Redis Cloud | >80% of 30MB |
| ML Service Uptime | UptimeRobot | <99% |
| Frontend Load Time | Lighthouse CI | FCP >2s |

### Weekly Monitoring Tasks

- [ ] Check Vercel Analytics for traffic patterns
- [ ] Review error logs in Vercel dashboard
- [ ] Monitor MongoDB storage usage
- [ ] Check Redis cache hit rates
- [ ] Review Railway ML service logs
- [ ] Run Lighthouse audit
- [ ] Test demo accounts still work
- [ ] Verify all 6 languages working

---

## Troubleshooting Production Issues

### Issue: API returning 500 errors

**Check:**
1. Vercel function logs: Dashboard → Functions → View Logs
2. Check environment variables set correctly
3. Test database connection: `curl https://farmly-ai.vercel.app/api/health`
4. Check MongoDB Atlas IP whitelist

### Issue: ML service timeouts

**Check:**
1. Railway deployment logs
2. Model file exists and loaded correctly
3. Railway service not sleeping (upgrade to paid tier)
4. Health endpoint: `curl https://farmly-ai-ml.up.railway.app/health`

### Issue: Voice recognition not working

**Check:**
1. Google Cloud credentials environment variable set correctly
2. Google Cloud Speech API enabled
3. Check backend logs for authentication errors
4. Verify API quotas not exceeded

### Issue: Market prices not loading

**Check:**
1. Agmarknet API status
2. Gemini API fallback working
3. Redis cache connection
4. Check backend logs for API errors

---

## Security Checklist

Production security best practices:

- [x] `DEMO_MODE=false` in production (or `true` only for hackathon demo)
- [x] Strong JWT secrets (32+ character random strings)
- [x] HTTPS enabled (automatic with Vercel)
- [x] CORS restricted to production domain
- [x] Rate limiting enabled (express-rate-limit)
- [x] MongoDB IP whitelist configured
- [x] Redis password authentication enabled
- [x] Sensitive environment variables never committed to Git
- [x] Google Cloud service account with minimal permissions
- [x] Twilio credentials secured
- [x] No API keys in frontend code

---

## Cost Monitoring

### Free Tier Limits

| Service | Free Limit | Warning Level | Upgrade Cost |
|---------|------------|---------------|--------------|
| Vercel | 100GB bandwidth/month | >80GB | $20/month Pro |
| Railway | $5 credit/month | >$4 used | Pay-as-you-go |
| MongoDB Atlas | 512MB storage | >400MB | $9/month M2 |
| Redis Cloud | 30MB memory | >24MB | $5/month 100MB |
| Twilio | $15 trial credit | <$5 remaining | Pay-as-you-go |
| Google Cloud | $300 trial credit | <$50 remaining | Pay-as-you-go |

### Monthly Cost Estimation (Production)

For ~1000 active users:
- Vercel: **$0** (within free tier)
- Railway: **$5-10** (recommended paid tier to avoid cold starts)
- MongoDB: **$0** (M0 sufficient)
- Redis: **$0** (30MB sufficient)
- Twilio: **~$10** (200 SMS at $0.05 each)
- Google Cloud: **~$5** (Speech API usage)

**Total Monthly: ~$20-25**

---

## Success Criteria

Deployment is successful when:

- [x] All health checks return 200 OK
- [x] All 5 core features working (auth, disease detection, voice, market, schemes)
- [x] All 6 languages functional
- [x] PWA installable on mobile
- [x] Lighthouse score >80
- [x] No critical errors in logs
- [x] Demo accounts working
- [x] Response times <2s (p95)
- [x] Uptime monitoring configured

---

## Post-Hackathon Maintenance

After hackathon, consider:

1. **Disable Demo Mode**: Set `DEMO_MODE=false`
2. **Enable Real OTP**: Uncomment Twilio SMS sending
3. **Add Analytics**: Google Analytics, Mixpanel, or PostHog
4. **Setup CI/CD**: GitHub Actions for automated testing and deployment
5. **Add More Tests**: Unit tests, integration tests, E2E tests with Playwright
6. **Improve Monitoring**: Add Sentry for error tracking
7. **Scale Database**: Upgrade MongoDB to M2 or M5 for production traffic
8. **CDN for Images**: Move disease images to dedicated CDN
9. **Add Caching**: Implement CDN caching for static assets
10. **Security Audit**: Penetration testing and security review

---

## Contact and Support

For deployment issues:
- Check [docs/TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md)
- Review [docs/CLOUD_SETUP.md](./docs/CLOUD_SETUP.md)
- Contact: [Your Email/Support]

---

**Last Updated:** February 13, 2026
**Version:** 1.0.0
**Status:** ✅ Production Ready
