# 🐾 PawTrail — Lost & Found Pet Reunion Platform

A complete full-stack web app for reporting, searching, and tracking lost/found
pets, built around the 10 differentiators from the pitch: photo-match
suggestions, geofenced alerts, movement heatmaps, QR pet tags, WhatsApp/SMS
reporting, a reward-split system, an anti-fraud live-capture camera flow,
owner-confirmation gating, trust scores, and a full audit trail.

It is a **single self-contained full-stack app**: one Node/Express server
serves both the REST API and the static frontend (plain HTML/CSS/JS — no
build step, no framework, nothing to compile).

---

## 1. Tech stack

| Layer     | Choice | Why |
|-----------|--------|-----|
| Backend   | Node.js + Express | Simple, fast to run anywhere |
| Data store| JSON-file storage (`server/data/*.json`, custom `db.js`) | Zero native dependencies — installs and runs on any machine with Node, no Postgres/Mongo setup required for this demo. Swap-in ready: every call goes through `db.js`'s `find/insert/update` API, so replacing it with a real database later only touches one file. |
| Auth      | JWT (`jsonwebtoken`) + `bcryptjs` | Stateless, simple |
| Uploads   | `multer` | Photo storage to `server/uploads/` |
| Photo matching | `jimp` (pure JS, no native build) | Color-histogram similarity + metadata scoring |
| QR tags   | `qrcode` | Generates PNG QR codes on the fly |
| Frontend  | Vanilla HTML/CSS/JS, multi-page | No build tooling required — open `npm start` and go |

---

## 2. How to run it

**Requirements:** Node.js 18+ (any recent LTS works).

```bash
cd petreunion
npm install
npm start
```

Then open **http://localhost:3000** in your browser.

That's it — the JSON data files and uploads folder are created automatically
on first run. To reset all data, stop the server and delete the contents of
`server/data/` and `server/uploads/`.

To change the port: `PORT=4000 npm start`.

---

## 3. Feature map (where each pitch item lives)

| Pitch feature | Implementation |
|---|---|
| Report lost/found + photo/location/species | `POST /api/cases`, `report.html` |
| Searchable/filterable feed | `GET /api/cases` (type, species, status, text search, near-me radius), `index.html` |
| User accounts + history | `routes/auth.js`, `dashboard.html` |
| **Photo-Match Suggestions** | `utils/imageSimilarity.js` — color histogram (Jimp) + metadata scoring, ranked in `GET /api/cases/:id/matches`, shown on `case.html` |
| **Geofenced Sighting Alerts** | `utils/geo.js` haversine distance; `routes/sightings.js` notifies the owner + watchers and flags `nearLastKnown` |
| **Movement Heatmap / Trail** | All sightings for a case (`GET /api/sightings/case/:caseId`), rendered as a custom lightweight SVG trail in `case.html` (no external map API key needed) |
| **QR Pet Tags** | `routes/pets.js` — `GET /api/pets/:id/qrcode` returns a PNG linking to a public, no-login-required profile (`pet.html`) with contact + medical info |
| **Low-Tech Reporting (WhatsApp/SMS)** | `routes/whatsapp.js` — a Twilio-webhook-shaped endpoint. See §5 below to wire a real number. |
| **Reward-Split Incentive System** | `routes/cases.js` — owner sets a reward on a "lost" post; split 20% (nearby sighting reporter) / 80% (physical returner), tracked in the `rewards` collection |
| **In-App Geotag-Only Camera** | `sighting.html` uses `<input capture="environment">` to force the live camera on supported mobile browsers, plus server-side freshness check on the capture timestamp (see limitation in §6) |
| **Owner-Confirmation Gate** | `POST /api/cases/:id/resolve` — only the original reporter can trigger this, and it's the only thing that releases reward funds |
| **Trust Score + Complaint Tracking** | `utils/trust.js` + `routes/complaints.js` — scores gate posting/claiming ability, and drop when a complaint is upheld |
| **Full Case Audit Trail** | `utils/audit.js` logs every action with a timestamp; viewable per-case on `case.html` and system-wide on `admin.html` |

---

## 4. Walking through the app

1. **Sign up** (`register.html`) as a pet owner.
2. **Report a lost pet** (`report.html`) with a photo, last-known location, and
   an optional reward.
3. Open **`register-pet.html`** to create a permanent pet profile and download
   a printable QR tag — anyone who scans it (`pet.html`) sees your contact
   info and medical notes, no login required.
4. From another account, **report a sighting** (`sighting.html`) with a live
   camera photo and current location. If it's near the pet's last known spot,
   the owner gets a geofenced alert and that reporter becomes eligible for the
   finder's reward share.
5. Someone who physically returns the pet **submits a recovery claim**
   (`case.html`).
6. The **owner confirms the reunion** on `case.html` — this is the only thing
   that releases the reward, split automatically between the sighting
   reporter and the returner.
7. Check **`dashboard.html`** for wallet balance, trust score, and
   notifications, and **`admin.html`** for the community-wide trust/audit
   transparency view.

---

## 5. Wiring the WhatsApp/SMS channel to a real number

`POST /api/whatsapp/webhook` already expects the same field names Twilio
sends (`From`, `Body`, `Latitude`, `Longitude`, `MediaUrl0`), so no code
changes are needed to go live:

1. Create a Twilio account and enable the WhatsApp Sandbox (or a real WhatsApp
   Business / SMS number).
2. Deploy this app somewhere with a public URL (Render, Railway, Fly.io, a VPS, etc.).
3. In Twilio's console, set the incoming-message webhook to
   `POST https://<your-domain>/api/whatsapp/webhook`.
4. Done — incoming messages become sightings (if they reference a case, e.g.
   "CASE-abc123 seen near the park") or new unmatched "found" reports for
   triage.
5. To have the app *reply* to the sender, add a call to the Twilio REST API
   inside `routes/whatsapp.js` using an account SID/auth token stored in
   environment variables (not included here, since it needs your own Twilio
   credentials).

---

## 6. Known simplifications (by design, for a runnable demo)

These are called out so nothing is presented as more solved than it is:

- **Data store**: JSON files, not a production database. Fine for a demo or
  small pilot; swap `db.js` for a real DB before scaling.
- **Photo similarity**: a lightweight color-histogram comparison, not a
  trained visual-embedding model (e.g. CLIP). It's good enough to *rank*
  candidates for a human to review — which is the actual UX goal — but it's
  not a guaranteed match.
- **Rewards are an internal wallet balance**, not a real payment. Releasing a
  reward credits `walletBalance` in the data store. Plugging in a real payout
  (Stripe Connect, PayPal Payouts, etc.) means adding a payment call where
  `routes/cases.js`'s `/resolve` endpoint currently does `walletBalance += ...`.
- **"Geotag-only camera"**: on the web, we can't *guarantee* a photo wasn't
  pulled from the gallery the way a native app can. This build approximates
  it with `capture="environment"` (forces the live camera on supporting
  mobile browsers) plus a server-side check that the reported capture
  timestamp is recent. A native mobile app (or a WebView with camera
  permissions) would be needed to fully close this gap.
- **WhatsApp/SMS**: the webhook logic is real and Twilio-payload-compatible,
  but no Twilio account is wired up here — see §5.
- **Trust-score moderation**: complaints are logged immediately, and the demo
  exposes a direct "uphold" action so the trust-score effect is visible
  end-to-end. In production, an admin/mod review step would sit in between.

---

## 7. Project structure

```
petreunion/
├── package.json
├── server/
│   ├── index.js              # Express app entry point
│   ├── db.js                 # JSON-file data layer
│   ├── middleware/
│   │   ├── auth.js           # JWT sign/verify
│   │   └── upload.js         # multer config
│   ├── routes/
│   │   ├── auth.js
│   │   ├── pets.js           # profiles + QR tags
│   │   ├── cases.js          # lost/found, feed, matches, rewards, resolve
│   │   ├── sightings.js      # geotagged sighting reports
│   │   ├── notifications.js
│   │   ├── complaints.js
│   │   ├── whatsapp.js       # Twilio-style webhook
│   │   └── admin.js          # trust scores + audit trail views
│   ├── utils/
│   │   ├── geo.js            # haversine distance
│   │   ├── imageSimilarity.js
│   │   ├── trust.js
│   │   ├── audit.js
│   │   └── notify.js
│   ├── data/                 # created at runtime (JSON "database")
│   └── uploads/              # created at runtime (uploaded photos)
└── public/                   # static frontend, no build step
    ├── index.html            # landing + feed
    ├── register.html / login.html
    ├── report.html           # report a lost/found pet
    ├── register-pet.html     # create a pet profile + QR tag
    ├── pet.html              # public finder-facing profile (QR destination)
    ├── sighting.html         # live-capture sighting report
    ├── case.html             # case detail: matches, trail, claims, resolve, audit
    ├── dashboard.html        # my cases, wallet, trust score, notifications
    ├── admin.html            # transparency dashboard
    ├── css/style.css
    └── js/app.js             # shared fetch/auth helpers
```

---

## 8. API quick reference

```
POST   /api/auth/register              { name, email, password, phone? }
POST   /api/auth/login                 { email, password }
GET    /api/auth/me                    (auth)

POST   /api/pets                       (auth, multipart: name, species, breed, medicalInfo, emergencyContact, photo)
GET    /api/pets/mine                  (auth)
GET    /api/pets/:id/public
GET    /api/pets/:id/qrcode            -> PNG

POST   /api/cases                      (auth, multipart: type[lost|found], species, breed, color, size, description, lastLat, lastLng, locationLabel, rewardAmount, photo)
GET    /api/cases?type=&species=&status=&q=&nearLat=&nearLng=&radiusKm=
GET    /api/cases/mine                 (auth)
GET    /api/cases/:id
GET    /api/cases/:id/audit
GET    /api/cases/:id/matches
POST   /api/cases/:id/watch            (auth)
POST   /api/cases/:id/claim-recovery   (auth) { note }
POST   /api/cases/:id/resolve          (auth, must be case owner)

POST   /api/sightings                  (auth, multipart: caseId, lat, lng, capturedAt, note, photo)
GET    /api/sightings/case/:caseId

GET    /api/notifications              (auth)
POST   /api/notifications/:id/read     (auth)

POST   /api/complaints                 (auth) { againstUserId, caseId?, reason }
POST   /api/complaints/:id/uphold      (auth)
GET    /api/complaints                 (auth)

POST   /api/whatsapp/webhook           (Twilio-style form fields)

GET    /api/admin/users                (auth)
GET    /api/admin/audit                (auth)
GET    /api/admin/stats                (auth)
```
