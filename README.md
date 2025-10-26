# CraHo Web - GitHub Pages Deployment

Production-ready web interface for CraHo provider onboarding and universal links.

## 🌐 Live URL
**Production:** https://craho.dk  
**GitHub Pages:** https://r13lundorff.github.io/craho-web

---

## 📋 Quick Start Deployment

### Step 1: Create GitHub Repository
1. Go to https://github.com/new
2. Repository name: `craho-web`
3. Set to **Public** (required for GitHub Pages)
4. ✅ Initialize with README
5. Click **Create repository**

### Step 2: Upload Files
Upload these files to the repository:

```
craho-web/
├── provider-setup/
│   └── index.html          ← Provider setup page
├── .well-known/
│   ├── apple-app-site-association  ← iOS universal links
│   └── assetlinks.json     ← Android app links
├── assets/
│   ├── styles.css          ← Shared styles
│   └── logo.svg            ← CraHo logo (add your logo)
├── index.html              ← Landing page
├── CNAME                   ← Custom domain config
└── README.md               ← This file
```

### Step 3: Enable GitHub Pages
1. Go to **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: **main** / **root**
4. Click **Save**
5. Wait 2-3 minutes for deployment

Your site will be live at: `https://r13lundorff.github.io/craho-web`

---

## 🌍 Custom Domain Setup (craho.dk)

### Step 1: Configure DNS (at your domain registrar)

Add these DNS records for `craho.dk`:

#### Primary Configuration (Choose A or B):

**Option A: Apex Domain (craho.dk)**
```
Type    Host    Value                   TTL
A       @       185.199.108.153        3600
A       @       185.199.109.153        3600
A       @       185.199.110.153        3600
A       @       185.199.111.153        3600
CNAME   www     r13lundorff.github.io   3600
```

**Option B: WWW Subdomain (www.craho.dk)**
```
Type    Host    Value                   TTL
CNAME   www     r13lundorff.github.io   3600
CNAME   @       www.craho.dk           3600
```

### Step 2: Add CNAME File
The `CNAME` file in this repository contains:
```
craho.dk
```

**Important:** This file must be in the root directory.

### Step 3: Configure GitHub Pages Custom Domain
1. Go to **Settings** → **Pages**
2. Custom domain: `craho.dk`
3. Click **Save**
4. ✅ Wait for DNS check (can take 24-48 hours)
5. ✅ Enable **Enforce HTTPS** (after DNS check passes)

### Step 4: Verify Domain
After DNS propagation (24-48 hours):
- Visit https://craho.dk
- Should show your landing page
- HTTPS should be enabled

---

## 📱 Universal Links Configuration

### iOS Universal Links

**File:** `.well-known/apple-app-site-association`

#### Update Required:
Replace `TEAM_ID` with your Apple Developer Team ID:

1. Go to https://developer.apple.com/account
2. Membership → Team ID
3. Copy your Team ID (example: `ABC123DEF4`)
4. Replace in file:
```json
"appID": "ABC123DEF4.com.craho.app"
```

#### Verify Installation:
```bash
curl https://craho.dk/.well-known/apple-app-site-association
```

Should return JSON (no .json extension in URL).

#### Test Universal Link:
1. Send link via iMessage to iOS device
2. Link format: `https://craho.dk/provider-setup?token=xyz`
3. Long-press link → Should show "Open in CraHo"

### Android App Links

**File:** `.well-known/assetlinks.json`

#### Get SHA256 Fingerprint:
```powershell
# From your keystore
keytool -list -v -keystore @rasmusdev__craho.jks
```

Or use EAS:
```powershell
npx eas credentials
# Select Android → keystore → View fingerprints
```

#### Update assetlinks.json:
Replace `YOUR_SHA256_FINGERPRINT_HERE` with your actual fingerprint:
```json
"sha256_cert_fingerprints": [
  "AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99"
]
```

#### Verify Installation:
```bash
curl https://craho.dk/.well-known/assetlinks.json
```

#### Test App Link:
```bash
adb shell am start -a android.intent.action.VIEW -d "https://craho.dk/provider-setup?token=xyz"
```

---

## 🧪 Testing the Provider Setup Flow

### Test URL Format:
```
https://craho.dk/provider-setup?token=<invitation_token>
```

### Complete Test Flow:

1. **Admin sends invitation** (from mobile admin panel)
   - Provider receives email with links

2. **Provider clicks web link**
   - Opens `https://craho.dk/provider-setup?token=xyz`
   - Page validates token via Railway API
   - Shows provider business info

3. **Provider creates account**
   - Enters password
   - Submits form
   - API creates Supabase user account

4. **Success redirect**
   - Mobile: Deep link to `craho://auth/login`
   - Desktop: Shows app download buttons

### Test on Different Devices:

**Desktop Browser:**
- Should show full web interface
- App download buttons visible
- QR codes (optional feature)

**Mobile Browser (iOS):**
- Should show web interface
- "Open in CraHo App" button
- Falls back to App Store if app not installed

**Mobile Browser (Android):**
- Should show web interface  
- "Open in CraHo App" button
- Falls back to Google Play if app not installed

---

## 🔧 Backend API Configuration

The web page connects to Railway backend:
```javascript
const BACKEND_URL = 'https://craho-production.up.railway.app';
```

### Required Endpoints:
```
GET  /provider-setup/validate/:token
POST /provider-setup/complete
```

These are already implemented in your `server/index.js`.

### CORS Configuration:
Update `server/index.js` to allow `craho.dk`:

```javascript
const corsOptions = {
  origin: process.env.NODE_ENV === 'production' 
    ? [
        'https://craho-production.up.railway.app',
        'https://craho.dk',
        'https://www.craho.dk',
        'https://r13lundorff.github.io',  // GitHub Pages
      ]
    : [...],
};
```

---

## 🎨 Customization

### Add Your Logo:
Replace `assets/logo.svg` with your CraHo logo.

Or use an image:
```html
<img src="assets/logo.png" alt="CraHo" width="120" height="120">
```

### Update Colors:
Edit `assets/styles.css`:
```css
:root {
    --primary: #007AFF;      /* Your brand color */
    --success: #28a745;      /* Success actions */
}
```

### Update App Store Links:
Edit `index.html` and `provider-setup/index.html`:
```html
<!-- iOS -->
<a href="https://apps.apple.com/app/YOUR_APP_ID">

<!-- Android -->
<a href="https://play.google.com/store/apps/details?id=com.craho.app">
```

---

## 📊 Analytics (Optional)

### Add Google Analytics:
```html
<!-- Add to <head> in index.html and provider-setup/index.html -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
```

---

## 🐛 Troubleshooting

### Issue: DNS not propagating
**Solution:** Wait 24-48 hours. Check with:
```bash
nslookup craho.dk
dig craho.dk
```

### Issue: HTTPS certificate not available
**Solution:** 
1. Ensure DNS is fully propagated
2. Disable and re-enable "Enforce HTTPS" in GitHub Pages settings

### Issue: Universal links not working (iOS)
**Solution:**
1. Verify Team ID is correct in `apple-app-site-association`
2. Must access via HTTPS (not HTTP)
3. Test with iMessage (Safari may cache)
4. Clear Safari cache and test again

### Issue: App links not working (Android)
**Solution:**
1. Verify SHA256 fingerprint is correct
2. Re-install app after updating `assetlinks.json`
3. Clear app data and test again

### Issue: CORS errors from Railway
**Solution:**
Update `server/index.js` CORS configuration to include:
```javascript
'https://craho.dk',
'https://www.craho.dk',
'https://r13lundorff.github.io'
```

### Issue: Provider setup page shows error
**Check:**
1. Backend is running: `https://craho-production.up.railway.app/health`
2. Token is valid (check database)
3. Browser console for API errors

---

## 🚀 Deployment Checklist

- [ ] Repository created: `r13lundorff/craho-web`
- [ ] All files uploaded to repository
- [ ] GitHub Pages enabled (Settings → Pages)
- [ ] DNS records configured at domain registrar
- [ ] CNAME file in repository root
- [ ] Custom domain configured in GitHub Pages
- [ ] HTTPS enabled (after DNS propagation)
- [ ] Apple Team ID added to `apple-app-site-association`
- [ ] Android SHA256 added to `assetlinks.json`
- [ ] Backend CORS updated to allow `craho.dk`
- [ ] Test provider setup flow end-to-end
- [ ] Universal links tested on iOS device
- [ ] App links tested on Android device

---

## 📞 Support

**Issues with deployment?**
- GitHub Pages docs: https://docs.github.com/en/pages
- DNS help: Contact your domain registrar
- Universal links: https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app

**CraHo Backend Issues:**
- Check Railway logs
- Verify environment variables
- Test API endpoints with Postman

---

## 📝 Notes

- GitHub Pages is **free** for public repositories
- SSL certificate is **automatic** with GitHub Pages
- DNS propagation can take **24-48 hours**
- Universal links require **HTTPS** (automatic with GitHub Pages)
- Test thoroughly on **real devices** (not just emulators)

---

## 🎉 Success!

Once deployed, your provider invitation emails will work perfectly:
1. Admin sends invitation
2. Provider receives email
3. Clicks web link → Opens craho.dk
4. Creates account on web
5. Downloads app or opens existing app
6. Logs in and starts managing bookings

**This is production-ready and follows industry best practices!**
