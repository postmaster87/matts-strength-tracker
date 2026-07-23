# HANDOFF — Matt's Strength Tracker → Firebase Sync Setup

**For Claude Code.** Drop this file in the repo root and tell Claude Code: "Read HANDOFF-strength-tracker.md and pick up where the checklist left off."

---

## Naming — do not mix these up

- This app is **Matt's STRENGTH TRACKER** (strength training tailored for a golfer — NOT powerlifting, NOT bodybuilding, NOT a golf app).
- A completely separate **GOLF ROUNDS TRACKER** will be built later. It does not exist yet. Never rename or reframe this app as a golf tracker.

## What this app is

Single-file web app (`index.html`, no build step) tracking a 12-week strength program written by Matt's trainer OV (United Movement): 3 phases × supervised/independent sessions, free-text entries per exercise per week, NPRS + session notes, and VALD gate check-in criteria. Weeks 1–6 history from the original spreadsheet is seeded into the code.

## Architecture facts Claude Code needs

- **One file.** All CSS/JS inline in `index.html`. No framework, no dependencies except Firebase compat SDK 10.12.2 loaded from gstatic CDN in `<head>`.
- **Storage is three-tier, auto-detected:**
  1. `window.storage` (Claude artifact hosting only — irrelevant on GitHub Pages)
  2. `localStorage` — always the local cache on GitHub Pages
  3. Firebase Firestore — cloud sync layered on top, activates only when `firebaseConfig` placeholders are filled AND user signs in
- **Security model:** the 4-digit PIN is a per-device screen lock only (stored locally, NOT synced). Google Sign-In + Firestore rules are the real security.
- **Shared-document model (differs from Matt's other Firebase project):** ONE doc `trackers/matt-strength`, writable by exactly two Google accounts (Matt + OV) via an email allowlist in the rules. This is NOT the own-document-per-user pattern used in the FlowCode range tracker — do not copy rules between the two projects.
- **Sync mechanics:** local save → debounced `pushCloud()` with `{merge:true}`; `onSnapshot` merges remote keys into local (remote wins per key); render is skipped while an input is focused. Last-write-wins per cell.
- **Data shape:** one JSON blob `{pin, vals:{...}}`. Keys: `e.<phase>.<session>.<groupIdx>.<exIdx>.<weekIdx>` (entries), `d.*` (dates), `n.*` (NPRS), `s.*` (notes), `g.*`/`gm.*` (gates). All values are strings.

---

## Checklist

### Already done ✅

- [x] App built, styled in OV's colors (navy #1F3864 / gold #C9A84C / teal #2E9DAD)
- [x] Weeks 1–6 spreadsheet history seeded
- [x] Dual-mode local storage (artifact vs localStorage) working
- [x] Floating "Done" button drops mobile keyboard
- [x] Auto-slash date entry (519 → 5/19)
- [x] Firebase sync code written into `index.html` (placeholders unfilled — sync currently OFF, app runs local-only)
- [x] Public GitHub repo created; GitHub Pages publishing in progress

### Remaining — GitHub Pages

- [x] Confirm the tracker file is committed as `index.html` in repo root
- [x] Settings → Pages → Deploy from branch → `main` / root → Save
- [x] Live URL loads at `https://postmaster87.github.io/matts-strength-tracker/` and PIN screen works
- [ ] On phone: open live URL in Chrome → Add to Home screen (ALWAYS launch from this icon — a downloaded copy of the file gets a separate localStorage and loses data)

### Remaining — Firebase project

- [x] console.firebase.google.com → Add project (e.g. `matt-strength-tracker`) → disable Analytics
  - Must be a **separate project** from the FlowCode range tracker (different security model)
- [x] Project → `</>` web icon → register app → copy the `firebaseConfig` values *(saved to `fireBaseConfig.txt`, project `matts-strength-tracker`)*
- [x] Paste each value over the `PASTE_...` placeholders in the marked block near the bottom of `index.html`'s script → commit + push *(done by Claude Code 2026-07-23)*
- [ ] Build → Authentication → Get started → Sign-in method → **Google** → Enable → set support email → Save
- [ ] Authentication → Settings → Authorized domains → add `<username>.github.io`
- [ ] Build → Firestore Database → Create database → production mode → us-central1

### Remaining — Firestore rules

- [ ] Rules tab → replace everything with the block below → swap in Matt's and OV's real Google emails → Publish

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /trackers/matt-strength {
      allow read, write: if request.auth != null
        && request.auth.token.email in ['MATT_EMAIL@gmail.com', 'OV_EMAIL@gmail.com']
        && request.auth.token.email_verified;
    }
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

### Remaining — Verify

- [ ] Live site → header chip "Sync off — sign in" is visible (if not, placeholders weren't replaced or SDK failed to load)
- [ ] Sign in with Matt's Google account → chip shows first name
- [ ] Make a test entry → indicator reads "Saved + synced"
- [ ] Firestore console shows `trackers/matt-strength` containing `vals`
- [ ] Incognito window, sign in with a third account → sees "Sync blocked" (rules working)
- [ ] OV's phone: open link → set his own device PIN → sign in with his Google → Matt's full log appears
- [ ] Optional hardening: Firebase console → confirm Google is the only enabled provider, only `<username>.github.io` + `localhost` in authorized domains, budget alert set

---

## Known behaviors / gotchas

- Placeholders unfilled → app is silently local-only by design; no error shown.
- "Sync blocked" toast → the signed-in email isn't in the rules allowlist, or rules weren't published.
- Google popup blocked on live site → `<username>.github.io` missing from Authorized domains.
- PIN is per device. Matt and OV can have different PINs; neither syncs. Forgotten PIN on a device = clear site data for the URL and re-set (cloud log restores after sign-in).
- The Firebase web config in the public repo is not a secret; rules + auth are the boundary.
- Two people editing the exact same cell simultaneously → last write wins. Different cells never conflict.
