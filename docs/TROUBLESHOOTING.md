# Troubleshooting Guide

Common issues and solutions for Farmly AI.

---

## Table of Contents

1. [Installation Issues](#installation-issues)
2. [Database Connection Issues](#database-connection-issues)
3. [Authentication Issues](#authentication-issues)
4. [API Errors](#api-errors)
5. [Frontend Issues](#frontend-issues)
6. [ML Service Issues](#ml-service-issues)
7. [Deployment Issues](#deployment-issues)
8. [Performance Issues](#performance-issues)

---

## Installation Issues

### pnpm command not found

**Error**: `pnpm: command not found`

**Solution**:
```bash
# Install pnpm globally
npm install -g pnpm

# Verify installation
pnpm --version
```

---

### Python version mismatch

**Error**: `Python 3.11+ required`

**Solution**:
```bash
# Check Python version
python --version

# If version < 3.11, install Python 3.11+
# Windows: Download from https://www.python.org/downloads/
# macOS: brew install python@3.11
# Linux: sudo apt install python3.11

# Use specific Python version
python3.11 -m pip install -r requirements.txt
```

---

### pnpm install fails with EACCES

**Error**: `EACCES: permission denied`

**Solution**:
```bash
# Fix npm/pnpm permissions (Linux/macOS)
sudo chown -R $(whoami) ~/.pnpm-store
sudo chown -R $(whoami) ~/.npm

# Or use sudo (not recommended)
sudo pnpm install
```

---

## Database Connection Issues

### MongoDB connection failed

**Error**: `MongoNetworkError: failed to connect to server`

**Symptoms**:
- Backend fails to start
- Error message: "MongoDB connection failed"

**Solutions**:

#### 1. Check connection string format

```bash
# Correct format
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/farmly_ai?retryWrites=true&w=majority

# Common mistakes:
# ❌ Missing database name
# ❌ Special characters in password not URL-encoded
# ❌ Wrong cluster hostname
```

#### 2. URL-encode special characters in password

If your password contains special characters (`@`, `#`, `$`, etc.), encode them:

| Character | Encoded |
|-----------|---------|
| `@` | `%40` |
| `#` | `%23` |
| `$` | `%24` |
| `%` | `%25` |
| `&` | `%26` |

Example:
```bash
# Password: myP@ss#123
MONGODB_URI=mongodb+srv://username:myP%40ss%23123@cluster.mongodb.net/farmly_ai
```

#### 3. Check IP whitelist

1. Go to MongoDB Atlas dashboard
2. Network Access → Add IP Address
3. Add `0.0.0.0/0` (allow from anywhere) OR your specific IP
4. Wait 1-2 minutes for changes to propagate

#### 4. Verify database user exists

1. Go to MongoDB Atlas dashboard
2. Database Access → Check if user exists
3. If not, create user with "Atlas Admin" privileges

#### 5. Test connection manually

```bash
cd backend
pnpm run test:db
```

Expected output: `✅ MongoDB connected successfully`

---

### Redis connection failed

**Error**: `Redis connection refused`

**Symptoms**:
- Backend starts but logs "Redis connection failed"
- Cache doesn't work

**Solutions**:

#### 1. Check Redis URL format

```bash
# Correct format
REDIS_URL=redis://default:password@redis-host:port

# Example
REDIS_URL=redis://default:abc123@redis-12345.c123.ap-south-1-1.ec2.cloud.redislabs.com:12345
```

#### 2. Verify Redis Cloud database is active

1. Go to Redis Cloud dashboard
2. Check database status (should be "Active")
3. If "Pending", wait for provisioning to complete

#### 3. Check password

1. Redis Cloud dashboard → Configuration → Show password
2. Copy exact password (including case-sensitive characters)
3. Update `.env` file

#### 4. Test connection

```bash
cd backend
pnpm run test:redis
```

Expected output: `✅ Redis connected successfully`

---

## Authentication Issues

### OTP not received

**Symptoms**:
- Phone number entered, but no SMS received

**Solutions**:

#### 1. Check Twilio configuration

```bash
# Verify these are set in backend/.env
TWILIO_ACCOUNT_SID=ACxxxx...
TWILIO_AUTH_TOKEN=your_auth_token
TWILIO_PHONE_NUMBER=+1234567890
```

#### 2. Use demo mode for testing

```bash
# In backend/.env
DEMO_MODE=true

# Now all demo account phone numbers use OTP: 123456
```

#### 3. Check Twilio balance

1. Go to Twilio Console → Balance
2. If balance is $0, add credits or use trial credits

#### 4. Verify phone number format

```bash
# Correct format: International format with +
✅ +919876543210
❌ 9876543210
❌ +91-9876543210
```

#### 5. Check backend logs

```bash
cd backend
pnpm run dev
# Look for Twilio error messages in console
```

---

### Invalid OTP error

**Error**: `Invalid or expired OTP`

**Solutions**:

#### 1. OTP expired

OTPs expire after 10 minutes. Request a new OTP.

#### 2. Check Redis is working

OTPs are stored in Redis. If Redis is down, OTPs won't work.

```bash
cd backend
pnpm run test:redis
```

#### 3. Use demo mode

```bash
# In backend/.env
DEMO_MODE=true

# Use demo phone numbers with OTP: 123456
+919876543210
```

---

### JWT token expired

**Error**: `Token expired` or `Invalid token`

**Solutions**:

#### 1. Refresh token

Frontend should automatically refresh tokens. If not working, log out and log in again.

#### 2. Check token expiration settings

```bash
# In backend/.env
JWT_EXPIRES_IN=7d              # Access token valid for 7 days
JWT_REFRESH_EXPIRES_IN=30d     # Refresh token valid for 30 days
```

#### 3. Clear browser localStorage

```javascript
// In browser console
localStorage.clear()
// Refresh page and login again
```

---

## API Errors

### 500 Internal Server Error

**Solutions**:

#### 1. Check backend logs

```bash
cd backend
pnpm run dev
# Look for error stack traces
```

#### 2. Verify all environment variables are set

```bash
cd backend
cat .env
# Ensure all required variables from .env.example are present
```

#### 3. Check external API status

- Google Cloud Speech API
- OpenWeatherMap API
- Agmarknet API
- Gemini API

Verify API keys are valid and services are available.

---

### 429 Rate Limit Exceeded

**Error**: `Too many requests`

**Solutions**:

#### 1. Wait for rate limit window to reset

Check response headers for `Retry-After` time.

#### 2. Reduce request frequency

See rate limits in [API.md](./API.md#rate-limiting).

#### 3. Use demo mode to test without limits (local only)

```bash
# In backend/.env
DEMO_MODE=true
```

---

### CORS errors in browser

**Error**: `Access-Control-Allow-Origin header`

**Solutions**:

#### 1. Check frontend API URL

```bash
# In frontend/.env
VITE_API_URL=http://localhost:4000   # For local dev
```

#### 2. Verify backend CORS configuration

Backend should allow requests from frontend URL. Check `backend/src/index.ts`:

```typescript
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:5173',
  credentials: true
}));
```

---

## Frontend Issues

### White screen / Blank page

**Solutions**:

#### 1. Check browser console for errors

Press F12 → Console tab

#### 2. Verify API URL is correct

```bash
# In frontend/.env
VITE_API_URL=http://localhost:4000
```

#### 3. Clear browser cache

```bash
# Hard refresh
Ctrl+Shift+R (Windows/Linux)
Cmd+Shift+R (macOS)
```

#### 4. Rebuild frontend

```bash
cd frontend
rm -rf node_modules dist
pnpm install
pnpm run dev
```

---

### Voice input not working

**Symptoms**:
- Microphone button doesn't work
- No audio recording

**Solutions**:

#### 1. Check microphone permissions

Browser must have microphone access permission. Allow when prompted.

#### 2. Use HTTPS or localhost

Web Speech API requires HTTPS (or localhost for development).

#### 3. Test microphone in browser

```javascript
// In browser console
navigator.mediaDevices.getUserMedia({ audio: true })
  .then(() => console.log('Microphone access granted'))
  .catch(err => console.error('Microphone error:', err));
```

#### 4. Check browser support

Web Speech API is supported in:
- ✅ Chrome/Edge 25+
- ✅ Safari 14.1+
- ❌ Firefox (limited support)

---

### Camera not working

**Solutions**:

#### 1. Check camera permissions

Allow camera access when prompted by browser.

#### 2. Use HTTPS or localhost

Camera API requires HTTPS (or localhost).

#### 3. Test camera

```javascript
// In browser console
navigator.mediaDevices.getUserMedia({ video: true })
  .then(() => console.log('Camera access granted'))
  .catch(err => console.error('Camera error:', err));
```

---

### Translations not loading

**Symptoms**:
- UI shows English instead of selected language
- Missing translation keys

**Solutions**:

#### 1. Check language files exist

```bash
cd frontend/src/locales
ls
# Should show: en.json, hi.json, ta.json, ml.json, te.json, kn.json
```

#### 2. Verify translation completeness

```bash
cd frontend
pnpm run i18n:check
```

#### 3. Clear browser cache and hard refresh

---

### PWA not installing

**Symptoms**:
- No "Install App" prompt
- Cannot add to home screen

**Solutions**:

#### 1. Use HTTPS

PWAs require HTTPS (or localhost for development).

#### 2. Check manifest.json

```bash
# Visit in browser
http://localhost:5173/manifest.json

# Should return valid JSON with app details
```

#### 3. Check service worker registration

```javascript
// In browser console
navigator.serviceWorker.getRegistrations()
  .then(regs => console.log('Service workers:', regs));
```

#### 4. Verify PWA criteria

Chrome DevTools → Application → Manifest
- ✅ manifest.json valid
- ✅ Service worker registered
- ✅ HTTPS (or localhost)
- ✅ Icons present

---

## ML Service Issues

### ML service fails to start

**Error**: `ModuleNotFoundError` or `ImportError`

**Solutions**:

#### 1. Install dependencies

```bash
cd ml-service
pip install -r requirements.txt
```

#### 2. Use virtual environment

```bash
cd ml-service
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

#### 3. Check Python version

```bash
python --version
# Should be 3.11+
```

---

### Disease detection returns low confidence

**Symptoms**:
- All predictions < 50% confidence
- Incorrect disease detected

**Solutions**:

#### 1. Use high-quality images

- ✅ Clear, well-lit photos
- ✅ Close-up of affected area
- ✅ In-focus image
- ❌ Blurry images
- ❌ Too dark/bright
- ❌ Multiple plants in frame

#### 2. Check model is loaded

```bash
curl http://localhost:8000/health
# Should return: {"status":"ok","model_loaded":true}
```

#### 3. Test with known disease images

Use sample images from the PlantVillage dataset.

---

### ML service timeout

**Error**: `Request timeout` or `504 Gateway Timeout`

**Solutions**:

#### 1. Increase timeout in backend

```typescript
// backend/src/services/ml.service.ts
const response = await axios.post(
  `${ML_SERVICE_URL}/ml/detect-disease`,
  data,
  { timeout: 30000 } // 30 seconds
);
```

#### 2. Use GPU for faster inference

```bash
# In ml-service/.env
DEVICE=cuda  # If GPU available
```

#### 3. Optimize image size

Backend should resize images before sending to ML service (max 1024x1024).

---

## Deployment Issues

### Vercel deployment fails

**Error**: Build fails on Vercel

**Solutions**:

#### 1. Check build logs

Vercel Dashboard → Deployments → Click deployment → View logs

#### 2. Verify build command

```json
// package.json
{
  "scripts": {
    "build": "tsc && vite build"  // Frontend
    "build": "tsc"                 // Backend
  }
}
```

#### 3. Set environment variables

Vercel Dashboard → Settings → Environment Variables
Add all variables from `.env.example`

#### 4. Test build locally

```bash
cd frontend
pnpm run build

cd backend
pnpm run build
```

---

### Railway/Render ML service crashes

**Error**: ML service exits with error

**Solutions**:

#### 1. Check logs

Railway/Render Dashboard → Logs

Common issues:
- Out of memory (increase instance size)
- Missing dependencies
- Python version mismatch

#### 2. Verify start command

```bash
# Correct start command for Railway/Render
uvicorn app.main:app --host 0.0.0.0 --port $PORT
```

#### 3. Add health check endpoint

Railway/Render will check `/health` endpoint.

#### 4. Set environment variables

- `PORT`: Railway/Render sets this automatically
- `GEMINI_API_KEY`: Add in dashboard
- `DEVICE`: Set to `cpu` (GPU not available on free tier)

---

### MongoDB Atlas "IP not whitelisted"

**Error**: `MongoNetworkError: IP address is not whitelisted`

**Solutions**:

#### 1. Add Vercel IPs to whitelist

MongoDB Atlas → Network Access → Add IP Address → `0.0.0.0/0`

**Note**: For production, use specific Vercel IP ranges.

#### 2. Wait for propagation

After adding IP, wait 2-3 minutes for changes to take effect.

---

## Performance Issues

### Slow API responses

**Symptoms**:
- API calls take >5 seconds
- Timeout errors

**Solutions**:

#### 1. Check database indexes

```bash
cd backend
pnpm run init:db  # Creates indexes
```

#### 2. Enable Redis caching

Verify Redis is connected:
```bash
cd backend
pnpm run test:redis
```

#### 3. Check external API latency

Use separate requests to test:
- Google Cloud Speech: ~1-2s
- OpenWeatherMap: ~200ms
- Agmarknet: ~500ms

#### 4. Monitor database performance

MongoDB Atlas → Metrics
Look for slow queries.

---

### Large bundle size

**Symptoms**:
- Frontend loads slowly
- Large JavaScript files

**Solutions**:

#### 1. Analyze bundle

```bash
cd frontend
pnpm run analyze
```

#### 2. Enable code splitting

Vite does this automatically. Ensure lazy loading for routes:

```typescript
// frontend/src/App.tsx
const HomePage = lazy(() => import('./pages/HomePage'));
const DiseaseDetectionPage = lazy(() => import('./pages/DiseaseDetectionPage'));
```

#### 3. Optimize images

Convert to WebP format and use lazy loading.

#### 4. Remove unused dependencies

```bash
cd frontend
pnpm prune
```

---

### High memory usage

**Symptoms**:
- Backend/ML service crashes with OOM (Out of Memory)
- Vercel/Railway/Render instance restarts

**Solutions**:

#### 1. Increase instance size

Upgrade to paid tier on Vercel/Railway/Render for more memory.

#### 2. Optimize ML model

Use quantized model (smaller size, faster inference):
```bash
# In ml-service
# Use TensorFlow Lite INT8 quantization
```

#### 3. Limit concurrent requests

Add queue system for ML inference requests.

#### 4. Clear cache periodically

```bash
# Redis cache cleanup
redis-cli FLUSHDB  # Clears all cache (use carefully!)
```

---

## General Debugging Tips

### Enable debug logs

```bash
# Backend
NODE_ENV=development pnpm run dev

# Frontend
VITE_DEBUG=true pnpm run dev
```

### Check service health

```bash
# Backend health
curl http://localhost:4000/health

# ML service health
curl http://localhost:8000/health
```

### Clear all caches

```bash
# Backend Redis cache
redis-cli FLUSHALL

# Frontend browser cache
# Browser: Ctrl+Shift+Delete → Clear all

# Node modules
rm -rf node_modules package-lock.json
pnpm install
```

### Reset database

```bash
cd backend
pnpm run init:db
pnpm run seed:schemes
pnpm run seed:demo
```

---

## Getting Help

If issues persist:

1. **Check documentation**:
   - [README.md](../README.md)
   - [QUICK_START.md](./QUICK_START.md)
   - [API.md](./API.md)

2. **Review logs**:
   - Backend: Console output
   - Frontend: Browser DevTools Console
   - MongoDB: Atlas Logs
   - Vercel/Railway: Deployment logs

3. **Search existing issues**:
   - GitHub Issues
   - Stack Overflow

4. **Ask for help**:
   - Create detailed GitHub issue with:
     - Error message (full stack trace)
     - Steps to reproduce
     - Environment (OS, Node version, browser)
     - Logs (backend and frontend)

---

**Last Updated**: February 13, 2026
