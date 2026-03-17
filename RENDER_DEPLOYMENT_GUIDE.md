# Render Deployment Guide for Jenisha Project

## Overview
Your project has multiple components that can be deployed on Render:
1. **Admin Panel** (React/Vite app) - Frontend
2. **Cloud Functions** (Node.js) - Backend services
3. **Static Files** (HTML/CSS/JS)

---

## Step 1: Prepare Your Project for Render

### 1.1 Create a `.env.render` file in your project root:
```
# Add your environment variables here
VITE_API_URL=https://your-render-backend-url.onrender.com
FIREBASE_API_KEY=your_firebase_key
FIREBASE_AUTH_DOMAIN=your_domain
FIREBASE_PROJECT_ID=your_project_id
FIREBASE_STORAGE_BUCKET=your_bucket
FIREBASE_MESSAGING_SENDER_ID=your_id
FIREBASE_APP_ID=your_app_id
```

### 1.2 Update the admin panel build configuration
In `admn/vite.config.ts`, ensure the build output is correct.

---

## Step 2: Deploy Admin Panel Frontend

### 2.1 Create a new Render Service:
1. Go to [https://render.com](https://render.com)
2. Click **"New +"** → **"Web Service"**
3. Connect your GitHub repository:
   - Select **GitHub** as repository source
   - Choose `sayaliileapforge-art/jenisha.final`
   - Click **Connect**

### 2.2 Configure the service:
- **Name:** `jenisha-admin-panel`
- **Environment:** `Node`
- **Build Command:** 
  ```
  cd admn && npm install && npm run build
  ```
- **Start Command:** 
  ```
  cd admn && npm run preview
  ```
- **Publish directory:** `admn/dist`

### 2.3 Add environment variables:
- Click **Environment** tab
- Add all variables from your `.env.render` file

### 2.4 Deploy:
- Click **Deploy**
- Wait for deployment to complete (usually 5-10 minutes)

---

## Step 3: Deploy Backend/Cloud Functions

### 3.1 Create a new Render Service for backend:
1. Click **"New +"** → **"Web Service"**
2. Connect repository again
3. Configure:
   - **Name:** `jenisha-backend`
   - **Environment:** `Node`
   - **Build Command:**
     ```
     cd admn/functions && npm install
     ```
   - **Start Command:**
     ```
     cd admn/functions && node index.js
     ```

### 3.2 Add environment variables:
- Add Firebase credentials
- Add database URLs
- Add API keys

### 3.3 Deploy:
- Click **Deploy**

---

## Step 4: Deploy Static Files (PHP → Node Server)

Since Render doesn't support PHP directly, convert your PHP scripts to Node.js or use a different service.

### Option A: Deploy as Static Site (simple files)
1. Click **"New +"** → **"Static Site"**
2. Connect repository
3. Configure:
   - **Name:** `jenisha-static`
   - **Build Command:** (Leave empty if no build needed)
   - **Publish directory:** `public_html`

### Option B: Convert PHP to Node.js
Create a simple Express server in `server.js`:
```javascript
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors());
app.use(express.json());
app.use(express.static('public_html'));

// Convert your PHP upload logic to Node.js endpoints
// Example:
app.post('/api/upload', (req, res) => {
  // Handle file upload
  res.json({ success: true });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## Step 5: Configure Database & Firebase

### 5.1 Firebase Setup:
1. Keep using your existing Firebase project
2. Update CORS settings in Firebase Storage
3. Add Render domains to Firebase authorized domains:
   - Go to Firebase Console
   - Settings → Authorized Domains
   - Add:
     ```
     your-service.onrender.com
     ```

### 5.2 Firestore Rules:
Update your `firestore.rules` for Render services:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

---

## Step 6: Connect Services Together

### 6.1 Update API endpoints:
In your frontend code, update API calls to point to Render services:
```typescript
// admn/src/services/api.ts
const API_BASE_URL = import.meta.env.VITE_API_URL || 
  'https://jenisha-backend.onrender.com/api';

export const apiClient = axios.create({
  baseURL: API_BASE_URL
});
```

### 6.2 Update CORS settings:
In your backend (`admn/functions/index.js`):
```javascript
const cors = require('cors');

app.use(cors({
  origin: [
    'https://jenisha-admin-panel.onrender.com',
    'https://your-domain.com',
    'http://localhost:3000'
  ]
}));
```

---

## Step 7: Connect Custom Domain (Optional)

### 7.1 For Admin Panel:
1. In Render dashboard, select `jenisha-admin-panel`
2. Go to **Settings** → **Custom Domains**
3. Add your domain
4. Update DNS records as instructed

### 7.2 Update Firebase settings:
- Add custom domain to Authorized Domains in Firebase Console

---

## Step 8: Configure Cron Jobs (If needed)

If you need scheduled tasks:
1. Create a separate Cron Job service
2. Set frequency (e.g., daily at 2 AM UTC)
3. Point to your backend endpoint

---

## Step 9: Monitoring & Logs

1. **View Logs:**
   - Each Render service has a **Logs** tab
   - Monitor for errors and performance

2. **Set up Alerts:**
   - Enable email notifications
   - Monitor for failed deployments

---

## Troubleshooting

### Issue: Port already in use
**Solution:** Render automatically assigns a port via `process.env.PORT`
```javascript
const PORT = process.env.PORT || 3000;
```

### Issue: Missing dependencies
**Solution:** Ensure `package.json` includes all deps in the correct directory

### Issue: Timeout during build
**Solution:** Increase build timeout in Render settings (up to 30 minutes)

### Issue: Firebase connection fails
**Solution:** 
- Verify Firebase credentials in environment variables
- Check Firestore Rules allow read/write
- Add Render IP to Firebase authorized networks

---

## Security Checklist

- [ ] Environment variables don't contain secrets in code
- [ ] `.env` files added to `.gitignore`
- [ ] Firebase Rules restrict unauthorized access
- [ ] CORS configured to allow only trusted domains
- [ ] SSL/HTTPS enabled on Render (automatic)

---

## Next Steps

1. **Test each service** after deployment
2. **Monitor logs** for errors
3. **Set up CI/CD** for automatic deployments on git push
4. **Configure custom domain** if you have one
5. **Set up monitoring** for uptime and performance

---

## Useful Links

- Render Docs: https://render.com/docs
- Firebase Deployment: https://firebase.google.com/docs
- Vite Build Guide: https://vitejs.dev/guide/build.html

