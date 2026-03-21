# Buddy-Divers Registration System — Handover Document
**Last Updated:** 21 March 2026 (session 12 — trip 260915 Madagascar Sep-A added to followup.html and HANDOVER)

---

## 🚀 PROMPT FOR NEXT CLAUDE SESSION

Paste this entire block at the start of any new Claude session:

```
You are continuing development of the Buddy-Divers diving liveaboard registration system.
Read the full HANDOVER.md carefully before doing anything.
The user will upload the latest HTML files and HANDOVER.md at the start of each session.

CRITICAL RULES — FOLLOW BEFORE TOUCHING ANY FILE:
1. Always work from the uploaded files — NEVER carry forward edits from a previous session's working copy
2. Verify OTP implementation before delivering any registration HTML (see OTP section)
3. Never add version suffixes to filenames
4. Never use long hyphens (—) in user-facing text — use a regular comma instead
5. Read HANDOVER.md fully before starting work
6. ALL bugs in the Bug Fix History are ALREADY FIXED and LIVE — do not re-investigate them
7. When delivering a new file version, ALWAYS base it on the uploaded file from that session
8. After ANY edit to buddy-divers-registration.html, extract script tags and run `node --check` — a SyntaxError breaks the entire page silently with no visible error
9. Every time buddy-divers-registration.html is uploaded, run a priceNum audit BEFORE any other work:
   - Every pricing option (including roomOptions and addons) must have priceNum as a non-zero integer
   - Negative priceNum is valid (credit addons e.g. credit_1400 has priceNum: -1400)
   - The "default" fallback trip has no priceNum by design — this is acceptable
   - Report any missing values to Ian before proceeding
10. Trip IDs are YYMMDD of the trip start date — flag any mismatch immediately.
    A wrong Trip ID causes Make.com Booking creation to fail silently with "[422] Value null is not a valid record ID"

SYSTEM: Five HTML files on GitHub Pages at register.buddy-divers.com, backed by Airtable and Make.com.
Owner: Ian, operator of Buddy-Divers Israeli liveaboard diving travel agency.
- buddy-divers-registration.html — diver registration form
- trips.html — public trip listing
- followup.html — Ian's internal payment manager (requires Airtable token per session)
- manifest.html — tour leader trip manifest (OTP login, read-only Airtable token hardcoded)
- myanmar.html — Myanmar-only trip listing page

ALL 5 FILES SHARE THE SAME FAVICON:
https://res.cloudinary.com/dccbsfte0/image/upload/v1774016253/logo2_swvqfe.jpg
Added via <link rel="icon" type="image/jpeg" href="..."> in <head>. When creating or editing any HTML file, preserve this favicon line.

CURRENT TRIP IDs — the #1 source of Make.com failures is a mismatch here:
| Trip | ID | Currency |
|---|---|---|
| Madagascar Jun 2026 (new) | 260620 | EUR |
| Madagascar Jul 2026 | 260730 | EUR | ← was wrongly 263007, fixed session 11 |
| Madagascar Aug 2026 | 260815 | EUR |
| Madagascar Sep 2026 (A) | 260915 | EUR |
| Madagascar Sep 2026 (B) | 260925 | EUR |
| Madagascar Oct 2026 | 261009 | EUR |
| Myanmar Dec 2026 | 261211 | USD |
| Philippines Dec 2026 | 261219 | USD | ← was 261215, fixed session 11 |
| Fiji & Tonga Aug 2026 | 260828 | USD |
| Galapagos Oct 2026 | 261022 | USD |
| Galapagos Oct 2027 | 271014 | USD |
| Thailand Mar 2026 | 260301 | USD |
| Philippines Combo (Anilao) | 261122 | USD |
| Mexico Combo | 270104 | USD |
| Myanmar Jan 2027 | 270106 | USD |
| Myanmar Feb 2027 | 270203 | USD |
NOTE: Madagascar Mar 2026 (260328) removed from trips.html but kept in TRIPS object — old links still work.

MULTI-CURRENCY SYSTEM:
- Trips priced in USD or EUR — stored in Airtable Trips.Currency field
- All Madagascar trips = EUR. All others = USD.
- followup.html reads Currency from Trips table → stored as booking.currency
- Both rates fetched from Leumi API: USD currencyID=1, EUR currencyID=49
- applyMarkup() updates all rate labels and symbols on every diver load

RETURNING CUSTOMER DISCOUNT — STORED AS NEGATIVE:
- e.g. -400 not 400. All arithmetic uses + discount.
- Airtable formulas use + {Returning Customer Discount} (not minus)
- populateForm() reads saved Airtable value first; auto-calcs only for new records

AIRTABLE PAYMENTS TABLE:
- Total Cost to Customer = {Safari Price} + {Flight Cost} + {Hotel Cost} + {Extras Amount} + {Returning Customer Discount}
- Total Paid = {Trip Payment} + {Flight Payment} + {Hotel Payment} + {Extras Payment}
- Balance Due = Total Cost to Customer minus Total Paid

CLOUDINARY:
- Payment JPGs: payment_requests/{IsraeliID}_{FirstName}_{LastName}/{TripID}/
- Folder names ASCII-only. booking object fetched BEFORE upload — critical ordering.

WHATSAPP FLOW (test number working — production pending Meta verification):
followup.html "שלח וואטסאפ" →
  1. Fetch booking, build Cloudinary folder
  2. Generate JPG via html2canvas
  3. Upload to Cloudinary
  4. POST {phone, imageUrl, caption:''} to Scenario 4 webhook
  5. If Credit Card → second POST {phone, textMessage}
  6. POST new record to WA Messages table

PERMANENT SYSTEM USER TOKEN:
- Never expires. In Make Scenario 4 HTTP modules Authorization header.
- DO NOT revoke or regenerate.
- Make.com OAuth for WhatsApp is BROKEN — always use HTTP module with Bearer token

PRODUCTION WHATSAPP STATUS:
- +972587232567, Phone Number ID: 1067652216421626, WABA ID: 927883142960562
- BLOCKED by Meta business verification (submitted 13 March 2026) — still pending as of session 11
- Once verified: NO code changes needed — just test from followup.html
- Check: business.facebook.com/settings/info → "Business verification status"

MANIFEST PAGE:
- Tour leader login: Trip ID + phone → OTP in JS → POSTed to Make Scenario 6 → stored in Airtable + emailed
- OTP verified in browser sessionStorage — cleared after successful verification
- Bookings query MUST use field name "Trip" NOT "Trip ID" — wrong name = no divers shown
- Read-only token hardcoded — GitHub flags on upload, select "I'll fix it later" → Allow Secret
- Nitrox column: ✓ green badge = certified, ✗ red X = not certified

MULTI-ADDON PRICING SYSTEM (trip 261219 only — all other trips completely unaffected):
- pricing array: radio (pick one, required) — base package price
- roomOptions array: radio (pick one, required when section visible) — cabin supplement
- addons array: checkboxes (optional) — each priceNum added to total (negative = credit)
- Grand total = pricing.priceNum + roomOptions.priceNum + sum(checked addons.priceNums)
- pricingLabel to Airtable: "full_package_5990 + upper_premium_275 + credit_1400"
- fullPackageOnly: true → addon hidden unless full_package_* selected
- Bidirectional credit/flight logic (261219 only):
  - Tick credit_1400 → hides Buddy-Divers flight, auto-selects self-arrange
  - Select self-arrange → auto-ticks credit_1400
  - Select Buddy-Divers → unticks credit_1400, Buddy-Divers option reappears

PHONE VALIDATION (both fields fixed session 11):
- Main phone and emergency contact phone both require minimum 7 digits
- Emergency phone was previously missing from required fields entirely — now fixed
- Both inputs have real-time blur feedback
- Error: נדרש מספר טלפון תקין / Valid phone number required

Ian's phones:
- +972542323567 = main number, WhatsApp Business app
- +972587232567 = secondary SIM — WhatsApp API only, do NOT install app on it
```

---

## 🎯 NEXT SESSION PRIORITIES

1. **⏳ Meta business verification** — submitted 13 March 2026, still pending. Check business.facebook.com/settings/info. When verified, test WhatsApp send from followup.html immediately — no code changes needed.

2. **📨 Scenario 5 — incoming WhatsApp reply forwarding** — build after production number confirmed working. Forwards messages from +972587232567 to Ian's +972542323567.

3. **🗺️ manifest.html testing** — OTP flow built. Test with a real trip + registered divers. Outstanding:
   - Phone normalization (Israeli 05X → +9725X) for leader login
   - WhatsApp OTP delivery — add after production WhatsApp confirmed

4. **🌍 Trip 260620 Airtable row** — still needs manual entry: Trip Id `260620`, Currency `EUR`, Start `20/06/2026`, End `30/06/2026`.

✅ ~~**Favicon added to all 5 HTML files**~~ — DONE session 11
✅ ~~**Phone + emergency phone validation fixed**~~ — DONE session 11
✅ ~~**Manifest nitrox: checkmark / red X**~~ — DONE session 11
✅ ~~**Madagascar Jun 2026 (260620) added**~~ — DONE session 11
✅ ~~**Madagascar Mar 2026 removed from trips.html**~~ — DONE session 11
✅ ~~**Trip ID 263007→260730 (Madagascar Jul)**~~ — DONE session 11
✅ ~~**Trip ID 261215→261219 (Philippines)**~~ — DONE session 11
✅ ~~**Trip 261219 Philippines repriced**~~ — DONE session 11
✅ ~~**Multi-addon pricing system**~~ — DONE session 11
✅ ~~**Bidirectional credit/flight logic**~~ — DONE session 11
✅ ~~**Multi-currency in followup.html**~~ — DONE session 8
✅ ~~**Permanent System User token**~~ — DONE session 4
✅ ~~**Business verification submitted**~~ — DONE session 4

---

## 🌐 Live URLs

| Page | URL |
|---|---|
| Registration Form | https://register.buddy-divers.com/buddy-divers-registration.html |
| Trips Listing | https://register.buddy-divers.com/trips.html |
| Follow-Up Manager | https://register.buddy-divers.com/followup.html |
| Tour Leader Manifest | https://register.buddy-divers.com/manifest.html |
| Myanmar Listing | https://register.buddy-divers.com/myanmar.html |
| Philippines Dec 2026 | https://register.buddy-divers.com/buddy-divers-registration.html?trip=261219 |
| Madagascar Jun 2026 | https://register.buddy-divers.com/buddy-divers-registration.html?trip=260620 |

---

## 📁 GitHub Repository

- **URL:** https://github.com/iancasper-create/buddy-divers-register
- **Branch:** main — PUBLIC (required for free GitHub Pages)
- Upload: Add file > Upload files > Commit
- Verify: Ctrl+F version tag date on live URL after 1-2 min

---

## 🔑 Credentials

### Airtable
- **Base ID:** app2RAeuaXaWrHOan
- **Token:** pat7joAq6LQn9XBT7.ade8e68f31fc96999c0e1642015d163216721e4775050b10c241828d0fde0a83
- NOT hardcoded in followup.html — entered manually per session, stored in sessionStorage (key: buddy-divers-followup-2)
- If rate shows 0.0000 after token entry — reload the page

### Cloudinary
- **Cloud:** dccbsfte0 | **Preset:** buddy_divers_uploads
- **Logo (emails):** https://res.cloudinary.com/dccbsfte0/image/upload/v1773063708/logo_hnmilh.png
- **Favicon:** https://res.cloudinary.com/dccbsfte0/image/upload/v1774016253/logo2_swvqfe.jpg
- Use Cloudinary URL in Make.com emails — never GitHub Pages URL (iPhone Mail blocks it)
- Payment JPGs: `payment_requests/{IsraeliID}_{FirstName}_{LastName}/{TripID}/`

---

## 🗄️ Airtable Tables

| Table | ID |
|---|---|
| Divers | tblE9d3rE4uDv2uxO |
| Trips | tbl8KGVwkIHi26plq |
| Bookings | tblMCB8nCSoRuiTKI |
| Payments | tbl0oLEDoa4PCQOOp |
| WA Messages | tblxcQFC35tOSH8l6 |

> Original handover had Divers and Bookings swapped — the above are correct.

### Trips table key fields
- **Trip Id** — YYMMDD, must match HTML TRIPS key exactly, digit for digit
- **Currency** — Single select: USD / EUR
- **Leader Name, Leader Phone (E.164), Leader Email** — set when assigning leader
- **Leader OTP, Leader OTP Expiry** — written by Make Scenario 6, do not manually edit
- **Vessel, Capacity** — shown on manifest.html banner

### Bookings table
- **Diver ID** — Linked to Divers
- **Total Price** — Lookup array — always unwrap before use
- **Returning Customer** — "yes" = 5% safari discount
- **⚠️ Zero Price Alert:** `IF({Total Price} = 0, "⚠️ CHECK PRICE", "")`

### Payments table
- **Returning Customer Discount** — NEGATIVE (e.g. -400)
- **Total Cost to Customer** — `{Safari Price} + {Flight Cost} + {Hotel Cost} + {Extras Amount} + {Returning Customer Discount}`
- **Total Paid** — `{Trip Payment} + {Flight Payment} + {Hotel Payment} + {Extras Payment}`
- **Balance Due** — Total Cost to Customer minus Total Paid

### Divers table
- **Payment Method** — `Bank Transfer`, `Credit Card up to 3 payments`, `Standing Order` — matched case-insensitively
- **Phone** — E.164 format

### WA Messages table
- One row per WhatsApp send
- Fields: Message ID, Payment (linked), Sent At, Image URL, Amount, Payment Type, Status, Israel ID, First Name, Last Name

---

## 📱 WhatsApp Business API

| Account | Phone Number ID | WABA ID |
|---|---|---|
| Test (+1 555 189 4715) | 1073799115807988 | 2010287653035398 |
| **Production +972 58 723 2567** | **1067652216421626** | **927883142960562** |

- **App:** Buddy-Divers, ID: 2357403824687608, PUBLISHED
- **Business portfolio:** Buddy-Divers (ID: 671223350061886)
- **System user token:** permanent — DO NOT revoke — in Make Scenario 4 HTTP modules
- **Make.com OAuth:** BROKEN — always use HTTP module with Bearer token

---

## ⚙️ Make.com Scenarios

| # | Function | Webhook URL |
|---|---|---|
| 1 | OTP generation | — |
| 2 | OTP verification | — |
| 3 | Registration + Booking creation | https://hook.eu1.make.com/h3wexi08df15jvkk1uyfw0ix669qv1ms |
| 4 | WhatsApp Payment Request | https://hook.eu1.make.com/yabs6aeja0bz6eoxw29p8t9tpgqi7tio |
| 6 | Manifest OTP | https://hook.eu1.make.com/w626oz6o3f6bp1m1spacjv7t6u1tg1um |

**ALL scenarios have "Has valid payload" filters — NEVER remove these.**

---

## 🖥️ Dev Notes — buddy-divers-registration.html

### Current version tag: BD-REG 2026-03-20

### Favicon
```html
<link rel="icon" type="image/jpeg" href="https://res.cloudinary.com/dccbsfte0/image/upload/v1774016253/logo2_swvqfe.jpg" />
```
Present in `<head>` of all 5 HTML files. Preserve on every edit.

### Phone validation (fixed session 11)
Both phone fields validated at submit — requires minimum 7 digits in the visible input:
```javascript
} else if (name === 'phone' || name === 'emergencyPhone') {
  const digits = val.replace(/\D/g, '');
  ok = digits.length >= 7;
  msg = 'נדרש מספר טלפון תקין / Valid phone number required';
}
```
Emergency phone resolved to `emergencyPhoneNumberInput` in `validateField()`.
Emergency phone is in the required fields array. Both inputs have blur + input event listeners.

### Trip 261219 — Philippines Combo (full structure)

**pricing** (radio, required):
- `full_package_5990` — החבילה המלאה, ים + יבשה, 11 לילות (כולל טיסות) — $5,990
- `boat_only_4390` — ספארי ימי בלבד, 7 לילות (כולל טיסות) — $4,390

**roomOptions** (radio, required — section header: תוספות שדרוג עבור חדרים בספינה):
- `lower_twin_0` — מפלס תחתון, חדר טווין (מיטת קומתיים) / Lower deck, twin room (Bunk Bed) — $0
- `lower_triple_90` — מפלס תחתון, חדר לשלושה בתפוסה זוגית בלבד / Lower deck, triple room for Double Occupancy Only — $90
- `main_front_140` — מפלס ראשי, חדר פרמיום קידמי / Main deck, front premium cabin — $140
- `main_premium_205` — מפלס ראשי, חדר פרמיום / Main deck, premium cabin — $205
- `upper_premium_275` — מפלס עליון, חדר פרמיום / Upper deck, premium cabin — $275

**addons** (checkboxes, optional — section header: תוספות ואפשרויות):
- `garden_suite_185` — שדרוג לחדר Garden Suite בריזורט / Resort Garden Suite upgrade — $185 — `fullPackageOnly: true`
- `credit_1400` — זיכוי כספי עבור רכישת טיסות עצמאית / Cash Credit for Self Flight Purchase — $1,400 display, priceNum: **-1400**
- `nitrox_boat_180` — נייטרוקס על הסירה / Nitrox Boat — $180
- `nitrox_boat_resort_240` — נייטרוקס על הסירה ובריזורט / Nitrox Boat & Resort — $240

**Grand total:** mainPriceNum + roomPriceNum + addonTotal

### Trip 260620 — Madagascar Jun 2026
Same pricing as other Madagascar trips (twin_2890, twin_2690, single_supplement €690).
Has `note: "⚠️ בכפוף למינימום נרשמים"` shown on the form.

### OTP Implementation — CRITICAL, verify before every delivery

**CORRECT:**
```html
<div id="otpDigitsDisplay" style="display:flex;gap:10px;justify-content:center;cursor:text;">
  <div class="otp-slot" id="otpSlot0"></div>
  <div class="otp-slot" id="otpSlot1"></div>
  <div class="otp-slot" id="otpSlot2"></div>
  <div class="otp-slot" id="otpSlot3"></div>
</div>
<input type="number" id="otpSingleInput" inputmode="numeric" autocomplete="one-time-code"
  maxlength="4" style="position:absolute;opacity:0;width:1px;height:1px;" />
```

### All validated mandatory fields:
idNumber, firstName, lastName, email, **phone** (min 7 digits), dob, nationality, passportNumber, passportIssue, passportExpiry, gender, certLevel, medicalNotes, emergencyName, emergencyEmail, **emergencyPhone** (min 7 digits), notes, diveComputer (radio), pricing (radio), flights (radio), paymentMethod (radio), room_option (radio — only when roomOptionsSection visible), terms (checkbox)

---

## 🖥️ Dev Notes — followup.html

### Current version tag: BD-FOLLOWUP 2026-03-20

### Trip names (tripNames):
```javascript
'260301':'תאילנד — מרץ 2026',      '260828':'פיג׳י / טונגה — אוגוסט 2026',
'261022':'גלפגוס — אוקטובר 2026',  '271014':'גלפגוס — אוקטובר 2027',
'260328':'מדגסקר — מרץ 2026',      '260620':'מדגסקר — יוני 2026',
'260730':'מדגסקר — יולי 2026',     '260815':'מדגסקר — אוגוסט 2026',
'260915':'מדגסקר — ספטמבר 2026 א׳', '260925':'מדגסקר — ספטמבר 2026 ב׳', '261009':'מדגסקר — אוקטובר 2026',
'261211':'מיאנמר — דצמבר 2026',    '270106':'מיאנמר — ינואר 2027',
'270203':'מיאנמר — פברואר 2027',   '261219':'פיליפינים — דצמבר 2026',
```

### Trip countries (tripCountries):
```javascript
'260301':'תאילנד',  '260328':'מדגסקר', '260620':'מדגסקר', '260730':'מדגסקר',
'260815':'מדגסקר', '260828':'פיג׳י / טונגה', '260915':'מדגסקר', '260925':'מדגסקר',
'261009':'מדגסקר', '261022':'גלפגוס', '271014':'גלפגוס',
'261211':'מיאנמר', '270106':'מיאנמר', '270203':'מיאנמר',
'261219':'פיליפינים',
```

### Leumi API currency IDs: USD=1, EUR=49, GBP=19, JPY=12, CHF=3

---

## 🖥️ Dev Notes — manifest.html

### Current version tag: BD-MANIFEST 2026-03-20

- Read-only token: `patlTOY5CIZQiCwX6.128e3422e74128f058aa8b3f51fe39ec5483eb7b6f23dc761c7c9a259bf82979`
- GitHub flags on upload — "I'll fix it later" → Allow Secret
- **Bookings query: `{Trip}` NOT `{Trip ID}`** — wrong name = no divers shown
- Nitrox: ✓ green badge (certified), ✗ red X (not certified)
- OTP generated in JS, stored in sessionStorage, cleared after verification

---

## 🖥️ Dev Notes — trips.html

### Current version tag: BD-TRIPS 2026-03-20

Static content — update manually when trips change.

**Madagascar section (7 trips):** Jun 260620, Jul 260730, Aug 260815, Sep-A 260915, Sep-B 260925, Oct 261009 + one more.
**Philippines card (261219):** 19/12/2026 – 30/12/2026, $4,390 – $5,990.
**Red note pattern:** `<div class="trip-note" style="color:#c0392b;font-weight:700;font-size:12px;margin-top:3px;">` — used for 260620 "בכפוף למינימום נרשמים".

---

## 🖥️ Dev Notes — myanmar.html

### Current version tag: BD-MYANMAR 2026-03-20

Standalone page — Myanmar trips only (261211, 270106, 270203). Same design as trips.html with destination badge header and cabin breakdown per card.

---

## 🧪 Test Diver
**ID:** 324127513 | **Email:** iancasper1@gmail.com | **Phone:** +972542323567

---

## ✍️ Writing Style Rules
- Bilingual: Hebrew first (RTL), English second (LTR)
- Inclusive Hebrew: `.י` suffix (e.g. `תרצה.י`)
- **NEVER use long hyphens (—) in user-facing text — use comma instead**
- Hints: red bold (#c0392b) | Errors: red

---

## 🗺️ Trip Management Guide

### 4 touchpoints — always keep in sync:

| # | Where | What |
|---|---|---|
| 1 | Airtable Trips table | Trip Id (YYMMDD), Name, Destination, Currency, Start/End Date |
| 2 | buddy-divers-registration.html | TRIPS object — pricing, priceNum, currency |
| 3 | trips.html | Trip card — dates, price range, optional note |
| 4 | followup.html | tripNames + tripCountries in generatePaymentImage() |

### Critical rules:
- **Trip ID = YYMMDD of start date** — must match Airtable and all HTML files exactly
- **priceNum mandatory on every option** including roomOptions and addons (negative OK)
- **Do NOT delete TRIPS entries** — leave so old registration links still work
- **node --check after every edit** to buddy-divers-registration.html

### Standard trip entry:
```javascript
"XXXXXX": {
  name: "🤿 שם — Name",
  dates: "DD/MM/YYYY – DD/MM/YYYY",
  currency: "USD",
  note: "optional note shown on form",
  pricing: [
    { value:"option_value", he:"...", en:"...", price:"$X,XXX", priceNum:XXXX },
    { value:"single_supplement", he:"תוספת שימוש יחיד", en:"Single room supplement",
      price:"€690 תוספת", priceNum:690, supplement:true }
  ]
}
```

### Extended trip (261219 pattern): see Dev Notes above for full structure.

### Pre-upload checklist:
1. Trip ID = YYMMDD — matches Airtable digit for digit
2. priceNum on every pricing/roomOptions/addons entry
3. `node --check` passes
4. Favicon line present in `<head>`
5. Upload to GitHub, wait 1-2 min, verify version tag live
6. Open `?trip=XXXXXX` — name, dates, pricing correct
7. Submit test registration — Airtable: Total Price not 0, Trip linked

### Removing a trip:
- trips.html: remove card, update badge count
- registration.html: **leave TRIPS entry** — old links must still work
- Airtable: leave row — linked to existing Bookings
- followup.html: no action needed

---

## 🐛 Complete Bug Fix History

| Bug | Root Cause | Fix |
|---|---|---|
| OTP autofill broken on iOS/Android | 4 separate inputs break autocomplete="one-time-code" | Single hidden input + 4 visual slots |
| OTP keeps reverting each session | New sessions used uploaded source file as base | Always verify OTP in output before delivering |
| Total Price = 0 in Airtable | Missing priceNum on multiple trips | Added priceNum to all — audited on every upload |
| Bookings search returning 0 results | ARRAYJOIN formula returns display names | Fetch all bookings, filter in JS |
| Diver names "Unknown Diver" | Wrong table ID | Read lookup fields from Booking record directly |
| Safari Price $0 | Lookup field returns array | Unwrap arrays before use |
| Table IDs swapped | Original handover had Divers/Bookings reversed | Corrected IDs documented |
| Booking creation failing | Field renamed to "Diver ID" | Updated Make.com steps |
| Trip 270107 null record | Typo in Airtable | Fixed Trip ID |
| Photos sideways | EXIF orientation not applied | formData.append('angle', 'auto_right') |
| Android camera/gallery | Single input incompatible | Dedicated separate capture inputs |
| Mexico Combo pricing wrong | La Paz as main option | Swapped — Quino Del Mar is main |
| Airtable token in git | Token hardcoded | Rotated, followup uses sessionStorage |
| Phone numbers inconsistent | Various local formats | cleanPhone() normalizes to E.164 |
| Logo not in iPhone Mail | GitHub Pages URL blocked | Cloudinary URL in emails |
| WhatsApp no automation | Manual wa.me | Full flow: Cloudinary → Make → WhatsApp API |
| Make.com WhatsApp OAuth broken | OAuth callback fails | HTTP module with Bearer token |
| Permanent token SMS not arriving | Meta Access Block | Resolved via Meta Business Support |
| Production number not sending | Unverified WABA | App published, permanent token, verification submitted |
| Exchange rate wrong | Frankfurter deprecated | Switched to Leumi live API |
| Payment image layout | flex inconsistencies | Strict CSS grid |
| Returning customer discount to all | Unconditional 5% | Reads Bookings.Returning Customer |
| Returning Customer Discount not saving | Missing from saveRecord() | Added to saveRecord() |
| Returning Customer Discount resets | populateForm always recalculated | Reads saved Airtable value first |
| EUR rate null | currencyID 2 not 49 | Fixed to 49 |
| Rate shows USD on EUR trip | boiRateDisplay set once | applyMarkup() updates on every diver load |
| Trip Link not populated | tripRecordId not stored | Added to booking object and saveRecord() |
| Balance box not refreshing | _existingPayments not updated | Updated immediately after save |
| Negative balance format | toLocaleString minus sign | Accounting format (€1,400) in red |
| Cloudinary send failing | booking declared after use | Moved to top of openWhatsApp() |
| Cloudinary folder rejected | Hebrew chars in path | Sanitize strips all non-ASCII |
| Fiji Trip ID mismatch | HTML 260828 vs Airtable 260829 | Corrected Airtable to 260828 |
| Manifest Bookings wrong field | "Trip ID" vs "Trip" | Fixed filterByFormula to use {Trip} |
| Manifest OTP as text | Make generated formula as string | OTP generated in JS, sent as value |
| GitHub blocked manifest upload | Read-only token detected | "I'll fix it later" → Allow Secret |
| JS SyntaxError broke page | Python replacement duplicated content | node --check on every delivery |
| Philippines Trip ID wrong | 261215 vs correct 261219 (YYMMDD) | Replaced all occurrences, Airtable updated |
| Madagascar Jul Trip ID wrong | 263007 vs correct 260730 (YYMMDD) | Replaced all occurrences, Airtable updated |
| Phone field accepted empty | cleanPhone() returned country code only | Added 7-digit minimum check in validateField() |
| Emergency phone not validated | Missing from required list entirely | Added to required array + validateField() + blur listeners |
| Manifest nitrox showed "Yes" text | Hard-coded "✓ Yes" string | Changed to ✓ green badge / ✗ red X |
| Favicon was base64 embedded in registration | Large inline blob, inconsistent across files | Replaced with Cloudinary URL, added to all 5 files |
