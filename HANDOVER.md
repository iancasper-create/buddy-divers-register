# BUDDY-DIVERS SYSTEM HANDOVER — SESSION 28
**Date:** 2026-05-28  
**Operator:** Ian Caspi (owner, Buddy-Divers)  
**Assistant:** Claude Sonnet 4.6  
**Files modified this session:** followup.html, crm.html (new file)

---

## SESSION SUMMARY

Three things completed this session:
1. **Make.com email Notes fix** — added the `Notes` field to all 8 registration email modules (11, 13, 14, 16, 18, 20, 21, 22) in the Integration Webhooks scenario.
2. **CRM Dashboard** — built a new `crm.html` admin hub page linking all system tools with a professional SaaS-style UI.
3. **ILS currency bug fix** — `followup.html` was showing `$` for ILS trips (260708, 260927). Fixed in 4 places to correctly show `₪`.

---

## CHANGES MADE THIS SESSION

### 1. Make.com — Integration Webhooks scenario (no HTML file change)

Added `{{1.notes}}` (Notes / Special Requests field) to all 8 email modules:

| Module | Type | Change |
|--------|------|--------|
| 11 | Client confirmation | Added Notes box with `{{if()}}` conditional |
| 13 | Client confirmation | Same as 11 |
| 14 | Admin notification | Added Notes row to Booking Details table |
| 16 | Admin notification | Same as 14 |
| 18 | Leader notification | Same as 14 |
| 20 | Leader notification | Same as 14 |
| 21 | Request Copy | Already had Notes — confirmed ✅ |
| 22 | Request Copy | Already had Notes — confirmed ✅ |

Client emails use `{{if(1.notes; "...html..."; "")}}` so the box only appears when Notes is not empty. Admin/leader emails always show the Notes row.

### 2. crm.html (BD-CRM 2026-05-28a) — NEW FILE

New admin dashboard page at `register.buddy-divers.com/crm.html`. Professional SaaS-style UI with:
- **Left sidebar** — navy background, white text, links to all pages, Ian Casper / Buddy-Divers footer
- **Top bar** — "Buddy-Divers CRM" in brand colours (navy / teal / orange), System Live pill
- **Stats row** — Active Trips (22), Platform, Make.com, WhatsApp status
- **Client-Facing section** — Registration Form, Trips Manager, Lottery Verification (teal accent cards)
- **Operations section** — Manifest, Follow-up & Payments, Edit Diver (orange accent cards)
- **Destination Pages section** — Myanmar, NAI'A Forms
- **External Systems row** — quick-launch cards for Airtable, Make.com, Cloudinary, WhatsApp Business
- Colour scheme: light blue-gray background (`#bdd1df`), white cards, navy sidebar, brand orange/teal accents

### 3. followup.html (BD-FOLLOWUP 2026-05-28a) — ILS currency fix

**Bug:** ILS trips (260708 Sinai, 260927 Sinai) were showing `$` instead of `₪` in payment requests.

**Root cause:** All currency symbol logic used a binary `EUR ? '€' : '$'` pattern with no ILS branch.

**Fixed in 4 locations:**
```javascript
// Before (all 4 instances):
currency === 'EUR' ? '€' : '$'

// After (all 4 instances):
currency === 'EUR' ? '€' : currency === 'ILS' ? '₪' : '$'
```

| Line | Context |
|------|---------|
| 1465 | `currSym` — currency badge + input prefix when trip loads |
| 1746 | `currencySymbol` — main payment request image |
| 2027 | `sym` — pro-forma summary display |
| 2074 | `currencySymbol` — pro-forma payment image |

---

## PREVIOUS SESSION SUMMARY (Session 27 — 2026-05-21)

Files changed: buddy-divers-registration.html (BD-REG 2026-05-21a), trips.html (BD-TRIPS 2026-05-21a)

Added new trip **260927 ספארי 5 כוכבים — סיני** (Sep 27 – Oct 1, 2026) to both files. Same pricing as 260708. Flags: `noFlights`, `roomOptional`.

---

## CHANGES MADE THIS SESSION

### 1. buddy-divers-registration.html (BD-REG 2026-05-21a)

Added trip **260927** — ספארי 5 כוכבים — סיני (Sep 27 – Oct 1, 2026):
- Venue: Renaissance Sharm El Sheikh Golden View Beach Resort
- Currency: ILS | Flags: `noFlights: true`, `roomOptional: true`
- Pricing identical to 260708: certified/OW/Advanced all ₪3,980 | non-diver ₪2,890 | single supp ₪760 | 3rd person −₪1,000
- Room upgrades: sea view double ₪175pp | sea view single ₪350 | suite double ₪395pp | suite single ₪790
- Guide: TBA (note in form: מלווה יפורסם בהמשך)

### 2. trips.html (BD-TRIPS 2026-05-21a)

Added trip 260927 card to the Sinai destination section with WhatsApp share and registration link.

---

## PREVIOUS SESSION CHANGES (Session 27 — 2026-05-21)

Added trip 260927 ספארי 5 כוכבים — סיני to registration.html and trips.html. Same pricing as 260708. Flags: `noFlights`, `roomOptional`. Renaissance Sharm venue.

---

## REMAINING CLEANUP (optional, not urgent)

- [ ] Delete empty `divers/` folder shells from Cloudinary Media Library manually
- [ ] Review and delete 62 no-trip test files (555555, 123456789, 900000050, 900000100, 900000150 etc.)
- [ ] Verify a few Airtable diver records that the photo URLs display correctly in manifest.html and edit-diver.html

---

## DEFINITIVE FILE STATUS (end of session)

| File | Version | Changed this session |
|------|---------|---------------------|
| buddy-divers-registration.html | BD-REG 2026-05-21a | No |
| trips.html | BD-TRIPS 2026-05-21a | No |
| followup.html | BD-FOLLOWUP 2026-05-28a | ✅ Yes — ILS currency symbol fix |
| crm.html | BD-CRM 2026-05-28a | ✅ Yes — new file |
| edit-diver.html | BD-EDIT 2026-04-04g | No |
| manifest.html | BD-MANIFEST 2026-04-13a | No |
| myanmar.html | BD-MYANMAR 2026-03-20 | No |
| lottery-check.html | BD-LOTTERY 2026-04-18a | No |

---

## OUTSTANDING ITEMS

- [ ] Payment Requests Airtable views: create Active (filter Paid=unchecked) and All views
- [ ] Backfill last 2 days of payment JPG sends manually into Payment Requests table
- [ ] Make.com Scenario 3: re-activate after customer (261219) re-submits
- [ ] Meta business verification (pending)
- [ ] WhatsApp Scenario 5 reply forwarding
- [ ] Blank Pricing field fixes in Airtable
- [ ] Scenario 6 recurring deactivation issue
- [ ] edit-diver.html debug panel removal
- [ ] Tests pending: requestCopy email, Payments row creation
- [ ] ILS note: followup.html Payment Requests table will receive Sinai prices already in ILS — exchange rate field irrelevant for trip 260708 rows
- [ ] Delete empty divers/ folder shells in Cloudinary (optional cleanup)

---

## SYSTEM CONSTANTS (for reference)

**Airtable:**
- Base ID: app2RAeuaXaWrHOan
- Divers: tblE9d3rE4uDv2uxO
- Bookings: tblMCB8nCSoRuiTKI
- Trips: tbl8KGVwkIHi26plq
- Payments: tbl0oLEDoa4PCQOOp
- WA Messages: tblxcQFC35tOSH8l6
- Payment Requests: tblqqgpdp5BOhSAH9

**Cloudinary:** cloud dccbsfte0 | upload preset: buddy_divers_uploads | logo: logo_hnmilh
- API Key: 789894644819389 (Root key)
- API Secret: BIHev-_geBBhIRcnwubf9qQcf0E
- New folder structure: {tripId}/passports/ and {tripId}/certificates/

**WhatsApp:** Production WABA: 927883142960562 | Phone Number ID: 1067652216421626

**GitHub Pages:** register.buddy-divers.com

**Key colleagues:** Amir Fireman (co-admin) | Amit Liber (guide) | Itay Grisaro | Dror Gilat | Oren Kedron | Yarin Shitrit | Daphna Stern | Ian Caspar

---

## PREVIOUS SESSION SUMMARY (Session 25 — 2026-05-19)

Files changed: buddy-divers-registration.html (BD-REG 2026-05-19e), trips.html (BD-TRIPS 2026-05-19b)

- Added new trip 260708 ספארי 5 כוכבים (Sinai) — first ILS currency trip
- Introduced `noFlights` and `roomOptional` flags
- Added Sinai destination section to trips.html

## CURRENT TRIP INVENTORY (registration.html)

| Trip ID | Name | Currency | Notes |
|---------|------|----------|-------|
| 260328 | Coral Triangle / Philippines | USD | |
| 260615 | Lembeh / Halmahera / Raja Ampat | USD | FULL |
| 260620 | Madagascar Jun 20 | EUR | |
| 260708 | ספארי 5 כוכבים — סיני | ILS | ירין שיטרית, noFlights, roomOptional |
| 260725 | Maldives Black Manta Jul 25 | USD | ללא ליווי |
| 260730 | Madagascar Jul 30 | EUR | |
| 260801 | Maldives Black Manta Aug 1 | USD | עמית ליבר |
| 260815 | Madagascar Aug 15 | EUR | |
| 260915 | Madagascar Sep 15 | EUR | |
| 260925 | Madagascar Sep 25 | EUR | FULL — waiting list |
| 260927 | ספארי 5 כוכבים — סיני | ILS | Renaissance Sharm, מלווה TBA, noFlights, roomOptional |
| 260828 | Fiji & Tonga | USD | FULL |
| 261009 | Madagascar Oct 9 | EUR | |
| 261022 | Galapagos Oct 2026 | USD | FULL |
| 261211 | Myanmar Dec 2026 | USD | |
| 261219 | Philippines Combo | USD | |
| 270104 | Mexico Combo | USD | |
| 270106 | Myanmar Jan 2027 | USD | |
| 270118 | Raja Ampat Jan 2027 | USD | FULL |
| 270203 | Myanmar Feb 2027 | USD | |
| 270422 | Tubbataha Apr 2027 | USD | |
| 271014 | Galapagos Oct 2027 | USD | |
| 270814 | Komodo/Indonesia Flagship | USD | |

---

## PROMPT FOR NEXT SESSION

Paste this at the start of the next session:

---

**SESSION 29 OPENING PROMPT:**

Hi Claude — continuing Buddy-Divers system development (Session 29).

Please read the attached HANDOVER.md carefully before doing anything else. Key context:

- I am Ian, owner of Buddy-Divers, an Israeli liveaboard diving agency
- The system runs on GitHub Pages (register.buddy-divers.com), Airtable, Make.com, Cloudinary, WhatsApp Business API
- I am non-technical — always give me step-by-step instructions for any platform actions
- Last session (28, 2026-05-28): added Notes field to all 8 Make.com registration emails; built new crm.html admin dashboard; fixed ILS currency symbol bug in followup.html (was showing $ instead of ₪ for Sinai trips 260708 and 260927).

**Coding rules (always apply):**
- All JS edits must pass `node --check`
- Use Python str.replace() for HTML changes — never regex for structural edits
- Use assertion-based Python scripts (assert old in html) to catch mismatches early
- Version tags format: BD-[FILE] YYYY-MM-DD[letter]
- Hebrew RTL Word docs: always python-docx with set_rtl_para() / set_rtl_run()
- No em-dashes anywhere in email campaigns; no emojis; soft tone; navy #1a3a5c; orange #d4692a

Please confirm you've read the handover and are ready to proceed.
