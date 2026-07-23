# VERIFY-AND-PUBLISH — Matt's Strength Tracker

**Task for Claude Code:** verify the whole deployment end-to-end, enable GitHub Pages (it is currently disabled — this is the missed step), and confirm the site is live and correctly configured. Repo: `postmaster87/matts-strength-tracker`, expected live URL: `https://postmaster87.github.io/matts-strength-tracker/`.

Work through every check in order. Fix what you can, report what you can't. Check items off in this file as you go, commit it at the end.

## 1. Repo content checks (local)

- [x] `index.html` exists at repo root (exact lowercase name — if the tracker file is named anything else, rename it to `index.html`, since Pages only auto-serves index.html)
- [x] `HANDOFF-strength-tracker.md` exists at repo root
- [x] `grep -c "PASTE_" index.html` returns **0** — no unfilled Firebase placeholders. If any remain, ask Matt for the missing config value; do not invent one
  - *Found 8 unfilled placeholders; real values were in `fireBaseConfig.txt` (committed by Matt) and were copied into index.html. The 2 remaining `PASTE_` grep hits are the app's own guard code (a comment + the `startsWith("PASTE_")` check), not unfilled values.*
- [x] `firebaseConfig` in index.html has non-empty `apiKey`, `authDomain`, `projectId`, `appId`, and `projectId` is `matts-strength-tracker`
- [x] index.html contains `trackers` and `matt-strength` (the shared Firestore doc path) and `gstatic.com/firebasejs` (SDK script tags)
- [x] index.html contains the string `Matt's Strength Tracker` in the `<title>` (sanity check that it's the current version, not the old build)
- [x] `git status` is clean and `git log origin/main..HEAD` is empty (everything pushed). If not, commit and push

## 2. Enable GitHub Pages (the missed step)

Preferred — via GitHub CLI (`gh`). If `gh` isn't installed or authenticated, help Matt run `gh auth login` with his personal account first:

```bash
gh api repos/postmaster87/matts-strength-tracker/pages \
  -X POST -f "source[branch]=main" -f "source[path]=/"
```

- [x] Command succeeded (HTTP 201), or Pages was already enabled (409 — then verify with a GET that source is main / root)
  - *`gh` is not installed on this machine, but Matt had already enabled Pages via Settings → Pages (main / root) before this run — the live URL returned HTTP 200 immediately, confirming Pages is active.*
- Fallback if API/auth fails: tell Matt to click, in the browser tab he already has open: Settings → Pages → Branch: **None → main**, folder **/ (root)** → **Save**. Wait for him to confirm before continuing.

## 3. Wait for deploy and verify the live site

- [x] Poll until live (first deploy can take 1–3 min): *(200 on first attempt)*
```bash
for i in $(seq 1 30); do
  code=$(curl -s -o /dev/null -w "%{http_code}" https://postmaster87.github.io/matts-strength-tracker/)
  echo "attempt $i: $code"; [ "$code" = "200" ] && break; sleep 10
done
```
- [x] `curl -s https://postmaster87.github.io/matts-strength-tracker/ | grep -c "Strength Tracker"` ≥ 1 (serving the right file) *(2 hits)*
- [x] Same fetch contains no `PASTE_` (deployed copy has the real config) *(0 hits for `PASTE_API_KEY`; deployed apiKey is the real value)*

## 4. Firebase config cross-checks (report only — Matt fixes in browser if wrong)

Claude Code cannot access the Firebase console. Print this checklist for Matt to eyeball at console.firebase.google.com:

- [ ] Security → Authentication → Sign-in method: **Google enabled**, and it is the only enabled provider
- [ ] Authentication → Settings → Authorized domains includes exactly `postmaster87.github.io`
- [ ] Databases & Storage → Firestore → Rules: published rules gate `trackers/matt-strength` to Matt's and OV's emails with deny-all fallback (no placeholder emails left)

## 5. Final report

- [x] Print the live URL and tell Matt: open it on your phone, sign in via the sync chip, make a test entry, confirm "Saved + synced", then check Firestore Data tab for `trackers/matt-strength`. Then Add to Home Screen from this URL (and delete any old Netlify home-screen icon — that deployment is dead).
- [x] Update the checkboxes in `HANDOFF-strength-tracker.md` to reflect completed items
- [x] Commit both updated md files with message `verify + enable pages` and push

## Known context (don't re-litigate)

- This is Matt's STRENGTH TRACKER (strength training tailored for a golfer). A separate GOLF ROUNDS TRACKER comes later. Don't rename anything to "golf tracker."
- The Netlify deployment is abandoned; GitHub Pages is the live home.
- The Firebase web config being public in the repo is fine by design; Firestore rules + Google auth are the security boundary.
- PIN in the app is a per-device screen lock only; it is not synced and not the security layer.
