# PetConnect

A community platform for reporting, searching, and tracking lost or found pets — built to help neighbourhoods reunite pets with their owners faster.

**Live demo:** _add your GitHub Pages / hosting link here once deployed_

## Problem

Communities regularly encounter stray or lost pets, but there is no organised way to report a sighting, search existing reports, or track a case until it's resolved. Information is scattered across WhatsApp groups and word of mouth, and gets lost quickly.

## Solution

PetConnect gives a community a shared, searchable board:

- **Report** — log a lost or found pet with species, colour/markings, last-seen location, date, description, an optional photo, and contact details.
- **Search** — filter the community board by status (lost / found / reunited), species, or free-text keyword.
- **Track** — update a case's status as it progresses, and mark it reunited once the pet is home.

## Features

- Dual report flow: "I lost a pet" / "I found a pet"
- Live search and filtering (status, species, keyword)
- Status lifecycle: `lost → found → reunited`
- Per-report detail view with reporter contact info
- "My reports" view scoped to the reporting device
- Photo attachment support
- Responsive layout, keyboard-accessible forms

## Tech stack

| Layer | Choice | Notes |
|---|---|---|
| Frontend | HTML, CSS, vanilla JavaScript | No build step — open `src/index.html` directly |
| Storage | Browser `localStorage` | Prototype-level persistence; see [Roadmap](#roadmap) for a real backend |
| Fonts | Oswald (display), Source Sans 3 (body) | Loaded via Google Fonts CDN |

This repo currently ships a fully working **frontend prototype**. It's structured so a backend (Node/Express, Django, or Firebase) can be dropped in behind the same data shape without a UI rewrite.

## Getting started

```bash
git clone https://github.com/<your-username>/petconnect.git
cd petconnect/src
```

Open `index.html` in any browser — no build tools or server required.

Optional (serves it over http:// instead of file://, needed for some browser APIs):

```bash
python3 -m http.server 8000
# visit http://localhost:8000
```

## Project structure

```
petconnect/
├── src/
│   ├── index.html      # App shell and views (browse / report / my reports)
│   ├── style.css        # Design tokens and layout
│   └── app.js            # State, rendering, form handling, storage
├── docs/
│   └── ARCHITECTURE.md   # Data model and how to swap in a real backend
├── .gitignore
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## Data model

Each report is stored as:

```json
{
  "id": "r_1699999999_ab12",
  "status": "lost",
  "name": "Bruno",
  "species": "Dog",
  "breed": "Indie",
  "colour": "Brown with white chest patch",
  "location": "Pollachi Bus Stand",
  "date": "2026-07-20",
  "description": "Wearing a red collar, friendly.",
  "photo": "data:image/jpeg;base64,...",
  "reporter": "Anita",
  "contact": "9876543210",
  "deviceId": "dev_xxxxx",
  "createdAt": 1699999999000
}
```

See `docs/ARCHITECTURE.md` for the fuller schema notes and backend migration plan.

## Roadmap

- [ ] Replace `localStorage` with a real backend (Node/Express + PostgreSQL, or Firebase) so reports are shared across devices
- [ ] Add authentication so users can manage only their own reports securely
- [ ] Add geolocation-based search ("pets found near me")
- [ ] Push notifications when a new report matches a saved search
- [ ] Image similarity matching between lost and found reports
- [ ] Admin/moderation view for community organisers

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

## License

Released under the [MIT License](LICENSE).
