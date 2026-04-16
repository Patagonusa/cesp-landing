# California Energy Savings Program — Landing Page

**Live:** https://californiaenergysavingsprogram.com
**Render:** https://cesp-landing.onrender.com
**Render Service ID:** `srv-d7giabrbc2fs73aog0ug`

---

## Architecture

```
                    +---------------------------+
                    |       GoDaddy DNS         |
                    |  californiaenergy...com   |
                    |  A → 216.24.57.1         |
                    |  CNAME www → onrender.com |
                    +------------+--------------+
                                 |
                                 v
                    +---------------------------+
                    |     Render Static Site     |
                    |     (Global CDN + SSL)     |
                    |     cesp-landing           |
                    |     Auto-deploy from       |
                    |     GitHub main branch     |
                    +------------+--------------+
                                 |
                    +------------+--------------+
                    |     Static HTML/CSS/JS     |
                    |                            |
                    |  index.html    (main page) |
                    |  luna-chat.js  (chat)      |
                    |  privacy.html              |
                    |  terms.html                |
                    |  terms-and-conditions.html |
                    |  images/                   |
                    +--+---------------------+--+
                       |                     |
            Form Submit (POST)        Luna Chat (API)
                       |                     |
                       v                     v
              +------------------+   +------------------+
              |  Patagon CRM     |   |  Patagon CRM     |
              |  /api/webhooks/  |   |  /api/chat/      |
              |  lead            |   |  session + message|
              +--------+---------+   +------------------+
                       |
                       v
              +------------------+
              |    Supabase      |
              |   (leads table)  |
              |  source: CESP Web|
              |  stage:          |
              |  pre_confirmed   |
              +------------------+
```

## How It Works

### Lead Flow
1. Visitor fills out the 4-step eligibility form (Homeowner? → ZIP → Upgrades → Contact)
2. Form submits to Patagon CRM webhook (`POST /api/webhooks/lead`)
3. CRM detects `form_name: "CESP Eligibility Check"` or `source: "CESP Web"` → routes to CRM (not QuickBase)
4. Lead created in Supabase `leads` table with `stage: pre_confirmed`
5. Notification SMS sent to sales team
6. Auto-welcome SMS + Luna AI follow-ups scheduled

### Luna Chat
- Chat widget appears after 15s, auto-opens with energy savings greeting after 20s
- Connects to `/api/chat/session` and `/api/chat/message` on Patagon CRM
- Green-themed to match CESP branding
- Supports English and Spanish

### Callback Number
- **Phone:** (888) 480-4286
- **Twilio SID:** PN3ac78b71e6b81b1eac479a1d125d3cb5
- **Type:** Toll-free, voice-only inbound (no SMS, no outbound dialing)
- **Purpose:** Callback number for customers from the Program campaign
- **Voicemail:** "Thank you for calling the California Energy Savings Program..."
- **Routing:** Inbound calls route to the last agent who called the customer, or to the assigned campaign agent

## File Structure

```
cesp-landing/
├── index.html                  # Main landing page (single-page, all inline CSS/JS)
├── luna-chat.js                # Luna AI chat widget (green CESP theme)
├── privacy.html                # Privacy Policy (TCPA, CCPA compliant)
├── terms.html                  # Terms of Service
├── terms-and-conditions.html   # Terms & Conditions + government disclaimer
├── favicon.svg                 # Green lightning bolt favicon
├── robots.txt                  # SEO robots file
├── sitemap.xml                 # SEO sitemap
├── images/
│   ├── logo.png                # CESP program logo (400x400)
│   ├── hero-bg.jpg             # Hero background (1920x1072)
│   ├── about-team.jpg          # Team / assessment photo (800x597)
│   ├── project-01.jpg          # Solar panel installation (600x448)
│   ├── project-02.jpg          # HVAC system upgrade
│   ├── project-03.jpg          # Window replacement
│   ├── project-04.jpg          # Roofing project
│   ├── project-05.jpg          # Insulation installation
│   ├── project-06.jpg          # Water heater installation
│   ├── project-07.jpg          # Exterior painting
│   └── project-08.jpg          # Complete home upgrade
└── README.md
```

## Form — 4-Step Eligibility Check

| Step | Question | Fields |
|------|----------|--------|
| 1 | Are you the homeowner? | Yes/No buttons |
| 2 | Where is your home located? | ZIP code (with CA area auto-detect) |
| 3 | Which upgrades interest you? | Multi-select: Solar, HVAC, Insulation, Windows, Roofing, Water Heater, Exterior Paint, Landscaping |
| 4 | Contact info | First name*, Last name, Phone*, Email, Address, TCPA consent checkbox (required) |

### TCPA Compliance
- Explicit opt-in checkbox (not pre-checked) required before submit
- Consent text references calls, texts, autodialer, prerecorded voice
- Consent timestamp (`tcpa_timestamp`) sent with form payload
- Privacy Policy and Terms of Service linked
- Submit button disabled until consent checked

### Form Payload → CRM
```json
{
  "first_name": "...",
  "last_name": "...",
  "phone": "(555) 123-4567",
  "email": "...",
  "zip": "91335",
  "is_homeowner": "Yes",
  "project_type": "Solar Panels, HVAC, Insulation",
  "source": "CESP Web",
  "form_name": "CESP Eligibility Check",
  "campaign": "CESP Web",
  "tcpa_consent": true,
  "tcpa_timestamp": "2026-04-16T18:30:00.000Z"
}
```

## CRM Integration

### CORS Origins (in Patagon CRM `/api/webhooks/lead`)
- `https://cesp-landing.onrender.com`
- `https://californiaenergysavingsprogram.com`
- `https://www.californiaenergysavingsprogram.com`

### Lead Detection
CRM uses `isCESPLead()` to detect CESP leads by:
- `form_name` containing "cesp"
- `campaign` starting with "cesp"
- `source` containing "cesp"

CESP leads are routed to the CRM path (not QuickBase).

## Deployment

- **Hosting:** Render Static Site (Global CDN)
- **Auto-deploy:** Every push to `main` triggers a deploy
- **Build:** None required (static HTML)
- **Publish path:** `.` (root)
- **Domain:** californiaenergysavingsprogram.com (GoDaddy → Render)
- **SSL:** Auto-provisioned by Render (Let's Encrypt)

## UTM Tracking

The form captures UTM parameters from the URL for ad attribution:
- `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`
- `gclid` (Google Click ID — persisted in sessionStorage)

Example ad URL:
```
https://californiaenergysavingsprogram.com/?utm_source=google&utm_medium=cpc&utm_campaign=CESP+Energy+LA
```

## Legal Disclaimer

> California Energy Savings Program helps homeowners determine their eligibility for energy savings programs available in California. We are not a government agency and are not affiliated with, endorsed by, or connected to any federal, state, or local government office.
