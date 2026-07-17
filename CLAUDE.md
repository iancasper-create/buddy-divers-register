# CLAUDE.md - Buddy-Divers Registration System

## What this repo is

Customer-facing and admin HTML for Buddy-Divers, an Israeli liveaboard dive travel agency.
Served via GitHub Pages + Cloudflare at **register.buddy-divers.com**. Every file is a
self-contained HTML page (inline CSS + JS, no build step, no framework).

Owner: Ian Caspi. Ian is non-technical: explain platform actions step by step, deliver
complete files (never partial snippets), and confirm before destructive changes.

Backend stack (not in this repo): Airtable (CRM), Make.com (automation, EU1),
Cloudinary (media, cloud `dccbsfte0`), Active Trail (email), WhatsApp Business API.

## Source of truth

The full system history lives in **HANDOVER.md**, kept OFFLINE by Ian (deliberately not
in this public repo). If a task needs history, Airtable schema, Make scenario details, or
past decisions, ask Ian to provide HANDOVER.md before proceeding. Do not guess.

## Coding rules (always apply)

1. **Version tags.** Every edited file gets a new tag: `<!-- BD-[FILE] YYYY-MM-DD[letter] | short change description -->`.
   Location: line 2 for CRM files, line 1 for edit-diver.html, line 45 for the four guest-form
   files (after the anti-translation script). Bump the letter (a, b, c) for same-day edits.
2. **Structural HTML edits:** use assertion-based Python `str.replace()`. Assert the old
   string appears exactly the expected number of times BEFORE replacing. Never sed,
   never regex for structure.
3. **JS validation:** every file whose scripts were touched must pass `node --check` on
   all extracted inline `<script>` blocks before delivery/commit.
4. **Pricing options** in buddy-divers-registration.html: `value`, `price`, and `priceNum`
   must ALWAYS be updated together. House convention: embed the price in the value key
   (e.g. `nondiver_2730`). Known intentional exceptions, do NOT "fix":
   `credit_1750` (priceNum -1750, a credit) and `private_guide_resort_110` (priceNum 0,
   per-day price excluded from totals).
5. **Guest forms** (dune_*, naia_*): never edit the `buildSnapshotHTML()` return-line html
   tag; keep signature compression via `compactSig()` (JPEG). After every deploy of
   naia_electronic_forms.html, re-check that `explore@naia.com.fj` is plain text
   (Cloudflare email-obfuscates it on each deploy).
6. **Style constants:** navy `#1a3a5c`, orange `#d4692a`. No em-dashes anywhere.
   No emojis and no `mailto:` links in email templates (Active Trail renders emojis as `?`).
7. **Hebrew/RTL:** customer-facing content is Hebrew RTL (`lang="he" dir="rtl"`).
   Keep dates LTR inside RTL context via `dir="ltr"`.
8. **Trips listing (trips.html):** when adding/removing a trip card, update the destination
   section's `trip-count-badge` to match. Removing a card does NOT remove the trip's
   config block from buddy-divers-registration.html unless Ian explicitly says so
   (direct `?trip=ID` links stay functional).
9. **Commits:** one logical change per commit, message states the new version tag,
   e.g. `trips.html BD-TRIPS 2026-07-17a: 260725 removed, waitlist badges added`.

## File map

| File | Purpose |
|------|---------|
| buddy-divers-registration.html | Registration form, per-trip via `?trip=ID` (largest file, many trip-config blocks) |
| trips.html | Public trip listing / WhatsApp sharing |
| crm.html | Admin dashboard, links all tools |
| followup.html | Payment manager (multi-currency, WhatsApp payment JPGs) |
| edit-diver.html | Admin diver record editor |
| manifest.html | Tour-leader manifest |
| registration-copy-email.html | "Request Copy" email template used by Make |
| lottery-check.html | Annual lottery registration verification |
| myanmar.html | Destination page |
| dune_aurora_forms.html, dune_black_manta_forms.html, dune_madagascar_forms.html, naia_electronic_forms.html | Guest declaration forms (shared Make webhook, Cloudinary raw upload, PDFBolt) |
| logo.png | Unused asset (all pages load the logo from Cloudinary); keep as-is |

Note: booking-sync.html exists but is intentionally NOT in this repo (local-only admin tool).

## Before finishing any task

- [ ] New version tag in every changed file, correct line position
- [ ] `node --check` passed on all touched script blocks
- [ ] value/price/priceNum consistency if pricing was touched
- [ ] Badge counts consistent if trips.html cards changed
- [ ] Tell Ian the new version tag(s) so he can update HANDOVER.md
