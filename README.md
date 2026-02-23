
# LNI Global — Lehem Network International
### Official Website & Network Portal

**Live Site:** https://LehemNetwork.org/
**Ministry:** Leemm Network International  
**Tagline:** Lagos · Global · Every Nation

---

## Overview

This repository contains the complete codebase for the LNI Global website and the LNI Network Leaders Portal. The site is built in plain HTML, CSS, and JavaScript — no frameworks, no build process, no dependencies. Every file is self-contained and deploys automatically to Netlify when pushed to this repository.

---

## File Structure

```
/
├── index.html              → Main public website (homepage)
├── contact.html            → Contact page
├── teachings.html          → Teachings archive
├── prayer.html             → Public prayer wall
├── partner.html            → Partnership / giving page
├── pioneer-toolkit.html    → Pioneer network toolkit (public)
├── privacy.html            → Privacy policy
├── total-man.html          → The Total Man 100-Day Curriculum page
├── leaders-hub.html        → Network Leaders Portal (login required)
├── lni-admin.html          → HQ Admin Command Centre (admin only)
└── README.md               → This file
```

---

## Page Descriptions

### `index.html` — Main Homepage
The primary public-facing website. Includes hero section, about LNI, pillars of the network, global network heatmap, testimonies, locations, partnership CTA, and footer. Powered by Firebase for real-time testimonies and the prayer wall.

### `contact.html` — Contact
Contact form routed through Web3Forms to HQ email. Also includes WhatsApp and location details.

### `teachings.html` — Teachings Archive
Video and audio teaching library. Embeds from YouTube/external sources.

### `prayer.html` — Prayer Wall
Public-facing prayer wall. Visitors can submit prayer requests which are stored in Firebase Firestore and displayed in real time.

### `partner.html` — Partnership
Partnership and giving page. Includes giving tiers, bank details, and online giving options.

### `pioneer-toolkit.html` — Pioneer Toolkit
Resource hub for people starting or growing an LNI network. Public-facing.

### `privacy.html` — Privacy Policy
Standard privacy policy covering data collection and use across all LNI digital properties.

### `total-man.html` — The Total Man
Landing page for the 100-Day Discipleship Curriculum. Links to the curriculum handbook and workbook PDFs.

### `leaders-hub.html` — Network Leaders Portal ⭐
**Restricted — requires individual login.**  
The private portal for all 35+ LNI network leaders worldwide. Each leader has their own Firebase Authentication account and sees only their own data.

**Leader features:**
- Personal dashboard — member count, gatherings, testimonies, reports submitted
- Monthly report submission — numbers and full narrative (testimonies, challenges, support needed, next month vision) with file attachment
- Full report history — view past reports and read HQ feedback on each
- Direct messaging with HQ — real-time thread
- Announcements from HQ — colour-coded by type
- Network Prayer Wall — shared prayer across all network leaders
- Network Goals tracker — set targets, track progress via progress bars
- Toolkit & Resources — links to key LNI documents
- Profile management — update personal and network details, change password

### `lni-admin.html` — HQ Admin Panel 🔐
**Restricted — HQ administrators only. Not linked publicly anywhere.**  
Access at: `lni.netlify.app/lni-admin` — bookmark this privately.

**Admin features:**
- Network Overview — live stats: total leaders, active this month, total members, salvations, pending reports, unread messages
- All Networks table — every leader, location, member count, last report at a glance
- Leader management — search, filter, view full profiles, activate/deactivate accounts
- Add New Leader — creates Firebase account, leader receives password setup email
- All Reports — read every section of every report, write feedback, mark as reviewed or needs-action
- Messages — grouped by leader, unread highlighted, reply directly from HQ
- Post Announcements — one post reaches all leaders instantly
- Prayer Requests — see all requests including private HQ-only submissions
- Activity Log — full audit trail of all admin actions

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Pages | Plain HTML / CSS / JS | No build step needed, anyone can edit, instant Netlify deploy |
| Database | Firebase Firestore | Real-time, free tier generous, scales to hundreds of leaders |
| Authentication | Firebase Authentication | Email/password login per leader, secure, free |
| Forms | Web3Forms | Contact and partnership forms route to email without a backend |
| Fonts | Google Fonts (Cinzel, Lato) | Loaded via CDN |
| Hosting | Netlify (via GitHub) | Auto-deploys on every push, free, fast CDN |

---

## Design System

All pages share a consistent design language:

| Token | Value | Usage |
|---|---|---|
| `--gold` | `#C9922A` | Primary accent, buttons, borders |
| `--gold-light` | `#F0C060` | Hover states, highlights |
| `--gold-pale` | `#FAE8B0` | Headings on dark backgrounds |
| `--deep` | `#0A0604` | Page background |
| `--dark` | `#1A1208` | Card backgrounds |
| `--mid` | `#2A1F0E` | Input backgrounds |
| `--text` | `#EDE0C8` | Body text |
| `--muted` | `#9A8870` | Secondary text, labels |

**Fonts:** Cinzel Decorative (display/logo) · Cinzel (headings) · Lato (body)

---

## Firebase Setup

### Project
Use the existing LNI Firebase project. Do not create a new one.

### Firebase Config
The config block appears in `index.html`, `leaders-hub.html`, and `lni-admin.html`. All three must use the identical config:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

**⚠️ Never commit real API keys without first restricting them in Google Cloud Console.**  
See the API Key Security section below.

### Firestore Collections

| Collection | Description | Who Writes | Who Reads |
|---|---|---|---|
| `leaders` | Leader profile documents (one per leader, keyed by UID) | Leaders (own doc), Admin | Leaders (own doc), Admin |
| `leader_reports` | Monthly reports submitted by leaders | Leaders | Leaders (own), Admin |
| `leader_goals` | Network goal targets set by each leader | Leaders (own doc) | Leaders (own doc), Admin |
| `messages` | Direct messages between leaders and HQ | Leaders + Admin | Leaders (own thread), Admin |
| `announcements` | HQ announcements visible to all leaders | Admin only | All authenticated leaders |
| `prayer_requests` | Prayer requests from leaders | All authenticated | All authenticated |
| `activity_log` | Admin action audit trail | Admin only | Admin only |
| `pending_leaders` | Leaders added before Firebase account created | Admin | Admin |

### Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isAdmin() {
      return request.auth != null &&
        request.auth.uid in ['PASTE_YOUR_ADMIN_UID_HERE'];
    }

    // Admin can read and write everything
    match /{document=**} {
      allow read, write: if isAdmin();
    }

    // Leaders can read/write their own profile
    match /leaders/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }

    // Leaders can read/write their own goals
    match /leader_goals/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }

    // Leaders can create reports and read/update their own
    match /leader_reports/{id} {
      allow create: if request.auth != null;
      allow read, update: if request.auth != null &&
        request.auth.uid == resource.data.leaderUid;
    }

    // Leaders can read/write messages in their own thread
    match /messages/{id} {
      allow read, write: if request.auth != null &&
        request.auth.uid == resource.data.leaderUid;
    }

    // All authenticated leaders can read announcements
    match /announcements/{id} {
      allow read: if request.auth != null;
    }

    // All authenticated leaders can read/write prayer requests
    match /prayer_requests/{id} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Replace `PASTE_YOUR_ADMIN_UID_HERE` with your actual Firebase UID (found in Firebase Console → Authentication → Users → your account row).

---

## Adding a New Network Leader

### Step 1 — Create their Firebase Auth account
1. Firebase Console → Authentication → Users → **Add user**
2. Enter their email and a temporary password: `LNI-Change-Me-2025`
3. Copy the **User UID** that is generated

### Step 2 — Create their Firestore profile
1. Firestore → `leaders` collection → **Add document**
2. Set the **Document ID** to their User UID (from Step 1)
3. Add these fields:

| Field | Type | Value |
|---|---|---|
| `uid` | string | their UID |
| `email` | string | their email |
| `firstName` | string | their first name |
| `lastName` | string | their last name |
| `networkName` | string | e.g. LNI Abuja |
| `city` | string | e.g. Abuja |
| `country` | string | e.g. Nigeria |
| `phone` | string | their WhatsApp number |
| `networkStarted` | string | e.g. March 2024 |
| `status` | string | `active` |

### Step 3 — Send them access
Tell the leader:
- Portal URL: `lni.netlify.app/leaders-hub`
- Their email and temporary password
- Ask them to log in and go to **My Profile → Change Password** immediately

> As the network grows, the **Add New Leader** button in the Admin Panel automates Steps 1 and 2.

---

## Removing or Deactivating a Leader

**To deactivate** (keeps data, removes access):
- Admin Panel → All Leaders → View → **Deactivate Account**
- This sets `status: inactive` in Firestore
- Also go to Firebase Console → Authentication → find their email → **Disable account**

**To permanently delete:**
- Firebase Console → Authentication → delete their account
- Firestore → `leaders` collection → delete their document
- Their reports remain in `leader_reports` for HQ records

---

## API Key Security

The Firebase API key is visible in the HTML source. This is normal and expected for Firebase web apps. Security comes from **restricting the key**, not hiding it.

### Restrict the key
1. Google Cloud Console → APIs & Services → Credentials
2. Click the API key → Edit
3. Under **Application restrictions** → HTTP referrers
4. Add:
```
lni.netlify.app/*
*.netlify.app/*
lehemnetwork.org/*
*.lehemnetwork.org/*
www.lehemnetwork.org/*
localhost/*
```
5. Under **API restrictions** → Restrict key → select only Firebase APIs
6. Save

### If a key is ever exposed without restrictions
1. Go to Credentials → Delete or Regenerate the key immediately
2. Apply restrictions to the new key before using it
3. If it was committed to GitHub history — delete and recreate the repository, then re-upload files

---

## Deployment

### How it works
Every time a file is pushed or uploaded to this GitHub repository, Netlify automatically detects the change and redeploys the site within 60–90 seconds. No build command is needed. No configuration required.

### To update any page
1. Open the file on GitHub
2. Click the pencil icon (Edit)
3. Make your changes
4. Click **Commit changes**
5. Netlify redeploys automatically

### To upload a new file
1. Go to the repository on GitHub
2. Click **Add file** → **Upload files**
3. Drag the file in
4. Click **Commit changes**
5. Netlify redeploys automatically

---

## Domain

Current: `lni.netlify.app`  
Future: `lehemnetwork.org` (to be connected via Netlify domain settings)

When the domain is ready:
1. Netlify → Site configuration → Domain management → Add custom domain
2. Point DNS at Netlify's nameservers as instructed
3. Add `lehemnetwork.org/*` to the Firebase API key restrictions (if not already done)

---

## PDF Resources

These files are not in this repository. Distribute them directly or via Google Drive:

| Document | Description |
|---|---|
| `total_man_curriculum.pdf` | The Total Man — 100-Day Discipleship Handbook (49 pages) |
| `total_man_workbook.pdf` | The Total Man — 100-Day Companion Workbook (76 pages) |

---

## Contact & Ownership

**Ministry:** Lehem Network International  
**Website:** [lni.netlify.app](https://lni.netlify.app)  
**Email:** leaders@lehemnetwork.org
*לֶחֶם — House of Bread · A Living Network Across Every Nation*


# lni
Lehem Network International Website
