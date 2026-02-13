# Demo Device Setup Guide

## 📱 Purpose
This guide ensures your demo device (smartphone) is optimally configured for the live demonstration. Follow this checklist **1 hour before** your presentation slot.

---

## ✅ Pre-Demo Checklist (Complete 1 Hour Before)

### 🔋 Battery & Power
- [ ] Charge phone to **100%** battery
- [ ] Bring backup portable charger (fully charged)
- [ ] Disable battery saver mode (can slow performance)
- [ ] Set screen to never auto-lock during demo (or set to 15 minutes)

**How to disable auto-lock:**
- **iOS**: Settings → Display & Brightness → Auto-Lock → Never
- **Android**: Settings → Display → Screen timeout → 30 minutes (max)

---

### 🌐 Network & Connectivity
- [ ] Connect to stable WiFi (preferably venue WiFi you've tested beforehand)
- [ ] Test internet speed (should be >5 Mbps download)
- [ ] Have mobile data enabled as backup (4G/5G)
- [ ] Disable automatic WiFi switching (sticky WiFi setting)
- [ ] Turn OFF Bluetooth (reduces interference and battery drain)
- [ ] Turn OFF NFC (unnecessary during demo)

**Speed test**: Use fast.com or speedtest.net - document the speed

**Backup plan**: If venue WiFi is unstable, use mobile hotspot from teammate's phone

---

### 🔕 Notifications & Interruptions
- [ ] Enable **Do Not Disturb** mode (no calls/texts during demo)
- [ ] Disable all app notifications (no popup banners)
- [ ] Close all background apps (swipe up/clear all)
- [ ] Log out of personal social media apps (avoid embarrassing notifications)
- [ ] Set phone to **Silent** mode (not vibrate - judges will hear it)
- [ ] Disable incoming call popup (or set to "Decline automatically")

**How to enable DND:**
- **iOS**: Control Center → Moon icon → Enable
- **Android**: Swipe down → Do Not Disturb → Enable

---

### 📱 App Configuration
- [ ] Open Farmly AI app and ensure it's fully loaded
- [ ] Log in with primary demo account: `+919876543210`
- [ ] Verify farm profile is complete (location, crops, language: Hindi)
- [ ] Clear app cache if performance is sluggish
- [ ] Ensure app is on latest version (check for updates)
- [ ] Test that app loads in <2 seconds (close and reopen)

**Demo account details:**
- **Phone**: `+919876543210`
- **Language**: Hindi
- **Location**: Pune, Maharashtra  
- **Crops**: Tomato, Onion
- **Demo Mode**: OTP bypass enabled on backend

---

### 🎤 Microphone & Audio
- [ ] Test microphone by recording a voice note
- [ ] Set **volume to 70%** (loud enough for judges to hear, not jarring)
- [ ] Test speaker by playing a video
- [ ] Clean microphone grill (dust can muffle audio)
- [ ] If using external speaker, pair and test beforehand
- [ ] Test voice recognition in Farmly AI app (speak sample query)

**Sample test query**: "टमाटर की कीमत क्या है?" (What is the price of tomato?)

**Expected result**: Transcription should appear within 2-3 seconds

---

### 📷 Camera
- [ ] Clean camera lens (use microfiber cloth)
- [ ] Test camera by taking a photo
- [ ] Ensure camera permission granted for Farmly AI app
- [ ] Prepare **sample diseased leaf image** in gallery
  - Use: `backend/uploads/sample-tomato-blight.jpg`
  - Transfer to phone via AirDrop/Google Drive/USB
- [ ] Test disease detection feature with sample image (should return result <2s)

**Fallback**: If live camera fails, use pre-loaded gallery image

---

### 📂 Sample Data & Content
- [ ] Verify query history is populated (should show 15 past queries)
- [ ] Verify disease detection history shows 8 past detections
- [ ] Load sample diseased leaf images in gallery:
  - `tomato-blight.jpg`
  - `tomato-leaf-curl.jpg`
  - `potato-early-blight.jpg`
- [ ] Bookmark home screen (for quick demo restart if needed)

**If data is missing**: Run backend seed script:
```bash
cd backend && pnpm run seed:demo-data
```

---

### 🖥️ Display & Brightness
- [ ] Set screen brightness to **100%** (ensures visibility on projector)
- [ ] Disable auto-brightness (prevents brightness changes mid-demo)
- [ ] Enable "Touch to Show Taps" (if presenting on projector)
  - **iOS**: Settings → Accessibility → Touch → AssistiveTouch → Enable
  - **Android**: Developer Options → Show Taps → Enable
- [ ] Set phone to **Portrait** orientation lock (prevent accidental rotation)
- [ ] Clean phone screen (fingerprints look bad on projector)

---

### 🔐 Permissions
Verify all permissions are granted for Farmly AI app:
- [ ] **Microphone** permission: Enabled
- [ ] **Camera** permission: Enabled
- [ ] **Location** permission: Enabled (While Using App)
- [ ] **Notifications** permission: Enabled (for alerts demo)

**How to check:**
- **iOS**: Settings → Farmly AI → Check all permissions
- **Android**: Settings → Apps → Farmly AI → Permissions

---

### 🧪 Pre-Demo Test Run (15 Minutes Before)

Run through this **complete demo flow** to ensure everything works:

#### Test 1: Voice Query
1. Open app → Tap microphone button
2. Speak: "टमाटर के पत्ते पर सफेद धब्बे दिख रहे हैं, क्या करूं?"
3. Verify transcription appears correctly
4. Verify response is returned in Hindi
5. Verify audio plays automatically

**Expected time**: <3 seconds from speech end to response start

**If fails**: 
- Check microphone permission
- Check internet connectivity
- Switch to Tamil demo account (different voice profile)

---

#### Test 2: Disease Detection
1. Navigate to Disease Detection page
2. Tap camera icon
3. Select sample image from gallery: `tomato-blight.jpg`
4. Verify result appears: "Tomato Late Blight, 96% confidence"
5. Verify recommendations are shown in Hindi
6. Verify "Save to History" button works

**Expected time**: <2 seconds from image upload to result

**If fails**:
- Check camera permission
- Verify ML service is running: `curl https://ml.farmly-ai.railway.app/health`
- Use pre-captured screenshot as backup

---

#### Test 3: Market Prices
1. Navigate to Market Prices page
2. Select crop: Tomato
3. Verify 5 nearest markets appear
4. Verify price chart renders
5. Verify AI recommendation shows: "Sell now" or "Hold for 3 days"

**Expected time**: <1 second (cached data)

**If fails**:
- Check internet connectivity
- Verify backend API is running: `curl https://api.farmly-ai.vercel.app/health`

---

#### Test 4: Government Schemes
1. Navigate to Schemes page
2. Verify eligible schemes appear (should show 6-8 schemes)
3. Tap on PM-KISAN scheme
4. Verify details and application steps appear

**Expected time**: <500ms (cached data)

---

### 📸 Backup Content
Prepare these backups in case live demo fails:

- [ ] Screenshots of each key screen (home, voice query result, disease detection result, market prices)
- [ ] Pre-recorded demo video (90 seconds, covers all features)
  - Stored in: `docs/demo-video.mp4`
  - Also uploaded to: Google Drive, USB drive
- [ ] Backup phone with same demo account logged in
- [ ] Laptop with app open in browser (PWA version)

**Backup demo video should show:**
1. Voice query in Hindi (15s)
2. Disease detection with result (20s)
3. Market prices and recommendation (15s)
4. Government schemes matching (10s)
5. Impact testimonial from farmer (30s)

---

## 🔧 Troubleshooting Guide

### Issue: App won't load / stuck on splash screen
**Solutions:**
1. Force close app and reopen
2. Clear app cache: Settings → Apps → Farmly AI → Clear Cache
3. Check internet connection
4. Restart phone
5. Switch to backup device

---

### Issue: Voice recognition not working
**Solutions:**
1. Check microphone permission (Settings → Permissions)
2. Speak closer to phone (6-8 inches from mic)
3. Reduce background noise
4. Switch to Tamil demo account (different accent)
5. Use text input as fallback (type query instead)

---

### Issue: Disease detection fails / returns error
**Solutions:**
1. Check camera permission
2. Use gallery image instead of live camera
3. Verify ML service health: `curl https://ml.farmly-ai.railway.app/health`
4. Show pre-captured result screenshot
5. Explain: "In production, this takes <2 seconds, but let me show you a pre-captured result..."

---

### Issue: APIs are slow / timing out
**Solutions:**
1. Check WiFi speed (might need to switch to mobile data)
2. Show cached data if available
3. Explain: "We have offline capabilities, so cached data is shown immediately..."
4. Switch to backup video
5. Use this as opportunity to highlight offline-first architecture

---

### Issue: Phone screen not visible on projector
**Solutions:**
1. Increase phone brightness to 100%
2. Clean phone screen
3. Adjust projector focus
4. Use phone mirroring (AirPlay/Chromecast) if available
5. Pass phone to judges to see directly

---

### Issue: Unexpected notification / call during demo
**Solutions:**
1. Quickly dismiss notification (swipe away)
2. Don't acknowledge it verbally (judges will forget it)
3. Continue with demo smoothly
4. After demo, joke: "Even farmers get interruptions—that's why we have Do Not Disturb mode!"

---

## 📋 Device Specs (For Reference)

Document your demo device specifications:

**Device Model**: [e.g., iPhone 14, Samsung Galaxy S23]  
**OS Version**: [e.g., iOS 17.2, Android 14]  
**Screen Size**: [e.g., 6.1 inches]  
**RAM**: [e.g., 6 GB]  
**Network**: [WiFi + 5G]  
**Battery Capacity**: [e.g., 3,279 mAh]

**Why this matters**: If judges ask "Will this work on a ₹5,000 phone?" you can answer confidently about minimum specs.

**Minimum specs for Farmly AI:**
- **OS**: iOS 14+ or Android 8+
- **RAM**: 2 GB minimum
- **Storage**: 100 MB for app + 50 MB for cached data
- **Camera**: 5 MP minimum
- **Network**: 2G minimum (offline mode works without network)

---

## 🎒 Physical Items to Bring

### Essential:
- [ ] **Primary demo phone** (charged to 100%)
- [ ] **Backup phone** (with same account logged in)
- [ ] **Portable charger** (fully charged, with cable)
- [ ] **USB cable** (for emergency file transfer)
- [ ] **Printed sample leaf image** (physical backup if camera fails)
- [ ] **Business cards** (with demo credentials written on back)
- [ ] **Laptop** (with backup video and app open in browser)

### Nice to have:
- [ ] External speaker (for louder audio)
- [ ] Phone stand/tripod (for stable demo on table)
- [ ] Microfiber cloth (for cleaning screen between runs)
- [ ] Sticky notes with demo script key points
- [ ] Bottle of water (stay hydrated while presenting)

---

## ⏰ Timeline: Day of Demo

### 2 Hours Before:
- [ ] Fully charge demo phone and backup phone
- [ ] Download backup video to laptop
- [ ] Test venue WiFi speed
- [ ] Verify all backend services are running (health checks)

### 1 Hour Before:
- [ ] Complete full checklist above
- [ ] Run through all 4 test scenarios
- [ ] Clean phone screen and camera lens
- [ ] Set up backup devices

### 30 Minutes Before:
- [ ] Set phone to Do Not Disturb
- [ ] Set brightness to 100%
- [ ] Open Farmly AI app and leave on home screen
- [ ] Put backup phone and charger in easy-to-reach pocket

### 5 Minutes Before:
- [ ] Final screen clean
- [ ] Check battery (should be >90%)
- [ ] Test microphone one last time (record voice note)
- [ ] Deep breath—you've got this!

---

## 🎬 Demo Day Setup

### At the presentation venue:

**Step 1**: Arrive at venue 15 minutes early  
**Step 2**: Test projector connectivity (if screen mirroring needed)  
**Step 3**: Verify WiFi connectivity at presentation spot  
**Step 4**: Run one quick test query to ensure everything works  
**Step 5**: Close app, return to home screen  
**Step 6**: Place backup devices within reach  
**Step 7**: Set phone on table in landscape mode (easier for judges to see)

---

## 📱 Phone Settings Summary

Quick reference for optimal demo settings:

| Setting | Value | Reason |
|---------|-------|--------|
| Battery | 100% | Avoid low battery warnings |
| Brightness | 100% | Visibility on projector |
| Auto-lock | 15 minutes | Prevent screen turning off |
| Do Not Disturb | ON | No interruptions |
| Volume | 70% | Audible but not jarring |
| Bluetooth | OFF | Reduce interference |
| WiFi | ON (stable network) | Fast API responses |
| Mobile Data | ON (as backup) | Failover if WiFi drops |
| Orientation Lock | Portrait | Prevent rotation |
| Show Taps | ON (if projecting) | Visual feedback for judges |

---

## 🎯 Final Pre-Demo Verification

**30 seconds before you start presenting**, verify:

✅ Phone in hand, unlocked  
✅ Farmly AI app open on home screen  
✅ Volume at 70%  
✅ Do Not Disturb enabled  
✅ Backup devices within reach  
✅ You're connected to stable WiFi  
✅ Battery >90%  
✅ You know your opening line

**Opening line suggestion:**
> "Meet Ramesh, a farmer from rural Maharashtra who lost 30% of his crop last year because he couldn't identify disease early enough. Today, I'll show you how Farmly AI solves this problem using voice-first AI. Let me demonstrate..."

---

## 💡 Pro Tips

1. **Hold phone at 45-degree angle** toward audience when demonstrating—easier for judges to see screen
2. **Speak slower than usual**—nerves make us rush
3. **Pause after results appear**—let judges see the result for 2 seconds before explaining
4. **Point at key UI elements** as you mention them
5. **Keep phone steady**—shaky hands look unprofessional (use two hands if needed)
6. **Make eye contact**—don't stare at phone for entire demo
7. **Smile**—shows confidence and passion

---

## 🆘 Emergency Contacts

If technical issues arise, contact:

**Backend/API Issues:**  
Contact: [Backend Developer Name]  
Phone: [Number]

**ML Service Issues:**  
Contact: [ML Developer Name]  
Phone: [Number]

**Frontend/App Issues:**  
Contact: [Frontend Developer Name]  
Phone: [Number]

**General Questions:**  
Contact: [Team Lead Name]  
Phone: [Number]

**Server Status Dashboard:**  
URL: https://farmly-ai.instatus.com (or your status page)

---

## 📊 Health Check Commands

Run these 15 minutes before demo to ensure all services are operational:

```bash
# Frontend (PWA)
curl -I https://farmly-ai.vercel.app
# Expected: HTTP 200 OK

# Backend API
curl https://api.farmly-ai.vercel.app/health
# Expected: {"status":"ok","db":"connected","redis":"connected"}

# ML Service
curl https://ml.farmly-ai.railway.app/health
# Expected: {"status":"ok","model_loaded":true}

# MongoDB
# Check from backend logs or MongoDB Atlas dashboard

# Redis
# Check from Redis Cloud dashboard
```

**If any service is down:**
1. Check deployment logs (Vercel, Railway)
2. Restart service if needed
3. Contact team member responsible for that service
4. Prepare to explain during demo: "We're experiencing high traffic, but in production this has 99.9% uptime"

---

## 🎤 What to Say If Something Goes Wrong

**If voice recognition fails:**
> "Voice recognition is typically 95%+ accurate, but let me show you using the text input option as well—the app supports both voice and text."

**If disease detection is slow:**
> "The model usually responds in under 2 seconds, but network latency can vary. This is why we've also built offline mode, which I'll demonstrate next."

**If app crashes:**
> "Let me restart the app—this actually demonstrates our offline-first design. Notice how quickly it loads cached data even without a network request."

**If internet connection drops:**
> "Perfect timing—this is exactly why we built offline capabilities. Let me show you what works without internet..." [Switch to offline mode demo]

**If phone battery dies:**
> "And this is why we have a backup device..." [Switch to backup phone] OR "Let me show you our backup video while this charges..."

---

## 🏆 Success Criteria

Your device is demo-ready when:

✅ All 4 test scenarios pass (voice, disease, market, schemes)  
✅ Response times are <3 seconds  
✅ No unexpected notifications appear  
✅ Battery is >90%  
✅ Audio is clear and audible  
✅ Camera and microphone permissions work  
✅ Sample data is loaded (history, images)  
✅ You've practiced the demo 3+ times on this device  
✅ Backup plan is ready (video, screenshots, backup device)

---

**You're ready! Trust your preparation. 🚀**
