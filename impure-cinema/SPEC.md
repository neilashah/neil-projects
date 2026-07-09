# Impure Cinema — SPEC.md
### Complete architecture reference, current as of the Trakt integration ship

This document is written to stand alone — it assumes no context from any prior conversation. It supersedes any earlier spec notes; where something changed (notably: the Miscibility Index went from "designed" to "shipped with Trakt" since the last version of this doc), this reflects the current shipped state only. For a running list of open items, see `ISSUES.md`.

---

## 1. What this is

A minimal iOS-style single-page web app for looking up a movie and seeing an aggregated critic/audience verdict in one number, plus where to stream it. Brand identity is built around a distillation metaphor (a flask that changes shape and fills with color based on the verdict) — the name "Impure Cinema" refers to raw, messy inputs from multiple rating sources being distilled into one score.

## 2. Architecture

- **Frontend:** single static `index.html` (vanilla JS, no framework, no build step).
- **Backend:** one serverless function (`netlify/functions/api.js`) proxying three upstream APIs so keys never reach the browser.
- **Hosting:** Netlify, deployed by dragging the project folder (containing `index.html` at the root) onto Netlify, or via Netlify CLI.
- **Two versions of the client exist:**
  - **Hosted (`index.html` in the Netlify project)** — calls the serverless proxy exclusively. This is the primary, secure version.
  - **Standalone (`ImpureCinema.html`)** — a downloadable single file with `YOUR_API_KEY`-style placeholders, calling TMDB/MDBList/Trakt directly from the browser. Exists for quick preview/testing; **not secure for real deployment** since keys would be exposed client-side if filled in. Both files are kept in sync feature-for-feature.

## 3. Data sources & environment variables

| Source | Purpose | Env var | Free tier |
|---|---|---|---|
| **TMDB** | Search, poster art, movie metadata, watch-provider data (via JustWatch) | `TMDB_KEY` | Generous |
| **MDBList** | Ratings aggregator — one call returns IMDb, Rotten Tomatoes (critic + audience), Metacritic (critic + user), Letterboxd, RogerEbert, Trakt's basic rating, and more | `MDBLIST_KEY` | 1,000 requests/day |
| **Trakt** | Ratings distribution histogram (needed for polarization signal) — not obtainable from MDBList's Trakt field, which only carries the aggregate number | `TRAKT_KEY` | Client ID only, no OAuth needed for this use case |

**OMDb is fully retired** — an earlier version of this app used it; it was replaced by MDBList mid-project and no OMDb code remains.

## 4. Serverless proxy (`netlify/functions/api.js`)

Four actions, all going through the same `?action=` query param pattern:

- **`search`** — TMDB movie search
- **`movie`** — TMDB movie details by TMDb id (this response includes a native `imdb_id` field, used to look the film up on Trakt — no extra TMDB call needed for that)
- **`providers`** — TMDB watch-provider data by TMDb id
- **`ratings`** — MDBList ratings by TMDb id
- **`trakt`** — *(special-cased, not part of the generic single-fetch switch)* — a two-step lookup: `GET /search/imdb/{imdb_id}?type=movie` to resolve the film's Trakt ID, then `GET /movies/{trakt_id}/ratings` for the distribution. Requires custom headers (`trakt-api-version: 2`, `trakt-api-key: <TRAKT_KEY>`), which is why it can't reuse the generic fetch path the other three actions share.

**Caching:** in-memory `Map`, keyed by action + id/query, warm across invocations while the Netlify function instance stays alive. TTLs: search 1h, movie 1d, providers 12h, ratings 1d, trakt 1d. Capped at 500 entries, oldest evicted first.

## 5. Two independent scoring systems

These are separate features with separate purposes. Do not conflate their tier/classification systems — they use different labels on purpose.

### 5a. Impurity Score — a quality score for the film itself

One number, 0–100, answering "how good is this film." Weighted average across up to 5 sources:

```
BASE_WEIGHTS = { imdb: 1.0, rt: 1.1, mc: 1.4, letterboxd: 1.05, tmdb: 0.7 }
confidence(votes) = min(1, log10(votes+1) / log10(50000))
weight(source) = BASE_WEIGHTS[source] × confidence(votes)   // confidence=1 for critic-only sources
Impurity = round( Σ(score·weight) / Σ(weight) )
```

Displayed as a color-coded ring plus a **shape-changing flask icon** — the flask's liquid literally fills with the tier color, and the flask's *shape itself* changes by tier (not just its color):

```
SCORE_TIERS: Panned (<50, cracked flask) · Mixed (50–69, calm flask)
           · Liked (70–84, lightly bubbling flask) · Acclaimed (85+, overflowing flask)
```

Tapping the score opens a breakdown panel showing each source's contribution, sorted by weight. Tier colors are a single-hue neon-magenta ramp (`--tier-panned` through `--tier-acclaimed`) — deliberately not the traffic-light red/yellow/green scheme, because meaning riding on brightness rather than hue keeps it colorblind-safe.

### 5b. Miscibility Index — a concordance score (do critics and audiences agree?)

A completely separate number/classification answering "how much do critics and audiences agree about this film, and is that agreement trustworthy." Named for the chemistry term — whether two liquids blend into one solution (miscible) or separate into layers (immiscible).

**Formula:**
```
z_s = clamp((x_s − μ_s) / σ_s, −3, 3)     // standardize each source against ITS OWN historical distribution
C = Σ(w·z)/Σ(w)  over critic sources (Rotten Tomatoes, Metacritic, RogerEbert)
A = Σ(w·c·z)/Σ(w·c)  over audience sources (IMDb, Letterboxd, TMDB, RT-audience, Metacritic-users, Trakt)
R = √(R_C · R_A)                           // reliability, from source coverage
D = R · (A − C)                            // signed divergence
M = 100 · exp(−(D / 0.85)²)                // Miscibility Index, 0-100
```

Calibration table and weights (verbatim from shipped code):
```
MISC_CAL = {
  imdb: {μ:63,σ:10}, letterboxd: {μ:64,σ:11}, tmdb: {μ:66,σ:9},
  popcorn: {μ:60,σ:18}, mcuser: {μ:64,σ:12}, trakt: {μ:72,σ:8},
  tomatoes: {μ:61,σ:26}, metacritic: {μ:56,σ:17}, rogerebert: {μ:62,σ:22}
}
MISC_W_CRITIC   = { metacritic:1.4, tomatoes:1.1, rogerebert:0.6 }
MISC_W_AUDIENCE = { letterboxd:1.05, imdb:1.0, popcorn:0.9, mcuser:0.8, trakt:0.8, tmdb:0.7 }
```

**Classification:**
```
R < 0.35                    → Unmeasured (no badge shown at all — absent evidence claims nothing)
M ≥ 70                      → Miscible (green-ish; "Widely loved"/"Widely panned" if both camps ≥|1|)
40 ≤ M < 70                 → Partial (no badge — ambiguous middle stays silent)
M < 40, D < 0                → Critic-Darling ("Critics' film")
M < 40, D > 0                → Crowd-Pleaser ("Audience's film")
```

**Two independent context notes** (either, both, or neither can attach to any classification above — they answer different questions):

- **Contested** — `M < 55` AND (audience dispersion ≥0.55 OR polarization P ≥0.6). Claims the *critic/audience gap itself* looks possibly manufactured (e.g., review-bombing).
- **Audience Divided** — dispersion ≥0.55 OR P ≥0.6, **no M gate at all**. Claims the *audience disagrees with itself*, independent of what critics said — can fire even on a Miscible film.

Where the two signals come from:
- **Dispersion** = weighted standard deviation of the audience camp's own z-scores. Catches manipulation concentrated on specific platforms (bombing usually hits RT-audience/Metacritic-user harder than Letterboxd/Trakt) and catches organic cinephile-vs-general-audience splits.
- **Polarization (P)** = from Trakt's 1–10 rating distribution, using a **two-sided** formula: `balanced = 2 × min(lowShare, highShare)` where low = ratings 1–3, high = 9–10. A one-sided version (just "lots of extreme ratings") was tried and rejected during design — it misread unanimous acclaim as more "polarizing" than an actual review-bombing case, since a pile of 9s and 10s alone is agreement, not division. Requires mass in *both* tails.

**Deliberately not built:** a symmetric "critic dispersion" signal was designed, tested, and rejected — found to run backwards (unanimous critical acclaim scored as *maximum* disagreement, because Rotten Tomatoes' pass-rate math and Metacritic's continuous average diverge most exactly when critics agree but only mildly, and converge when critics genuinely split). This is a permanent decision, not a gap — don't re-attempt it without re-reading why it failed.

## 6. Icons

Two separate icon families, different conventions — keep them straight:

- **Flask-tier family** (Impurity Score): `viewBox="0 0 64 64"`, `stroke-width="2.4"`. Symbol IDs: `flask-panned`, `flask-mixed`, `flask-liked`, `flask-acclaimed`, `flask-unmeasured`. Rendered via `<use href="#id">`.
- **Verdict family** (Miscibility Index badges): `viewBox="0 0 24 24"`, `stroke-width="2"`, rendered at 15×15px in a text pill. Symbol IDs: `verdict-miscible` (two circles, Venn overlap filled — replaces a generic handshake), `verdict-immiscible` (two-layer separated container — replaces a generic arm-wrestle), `verdict-contested` (gauge with a bent/erratic needle), `verdict-divided` (two jagged interlocking half-circles with a gap between them).

Both sprites are defined once as hidden `<svg style="display:none">` blocks near the top of `<body>`, referenced via `<use>` everywhere they're needed — not duplicated inline per instance.

**Open item:** Immiscible and Contested were built directly from a design spec rather than going through the same design → review → approve loop the other two icons did. Worth a direct visual check before considering the set fully settled (see `ISSUES.md`).

## 7. UI structure (single page, two views)

- **Search view:** search bar, debounced TMDB search, results list showing poster/title/year only *(no per-row score yet — designed, not built; see §9)*.
- **Detail view:** poster, title, year/runtime, Impurity Score (ring + flask + tap-to-expand breakdown), Miscibility Index badges (0–2 pills, stacked), "Where to watch" (small grayscale provider logos, quiet label), footer attribution.

Design language: near-black background, single neon-magenta ramp reserved exclusively for score tiers ("color means verdict"), cyan (`#2FE6D8`) as the one brand/interactive accent, monochrome everything else (source rings, provider logos).

## 8. Known unverified items (built correctly per documentation, never tested live)

- Trakt's two-step lookup has never hit their actual API from the build environment — built to their documented shape, not confirmed.
- The standalone file's direct-from-browser Trakt call may hit CORS blocking (the hosted version avoids this by proxying server-side).
- `metacriticuser` and `rogerebert` score-scale conversions are best-guess (MDBList's own normalized `.score` field preferred when present, `.value`-based fallback otherwise).

Full detail and more items in `ISSUES.md`.

## 9. Designed but not built

- **Results-list per-row scores/flags.** A full plan exists: fetch scores lazily (IntersectionObserver, capped to the first 6–8 visible rows, rest on scroll) to avoid bursting API limits on every search. This is the largest gap between what's been designed and what's shipped.
- **Discourse heat (H)** — an attention-spike signal (Wikipedia pageviews + Trakt comment intensity) that was part of the original Miscibility Index design but never wired in. Deliberately deprioritized — P and dispersion carried the signal load in all validation testing; H never independently drove a classification.
- **PWA "Add to Home Screen" treatment** — discussed, explicitly deferred at the project owner's call.

## 10. Deploy checklist

1. Unzip the project — confirm `index.html`, `netlify.toml`, and `netlify/` sit directly inside the folder you're about to deploy (a past incident: a double-nested zip export caused Netlify to serve `README.md` as the homepage instead of the app).
2. Drag that folder onto Netlify (or `netlify deploy --prod` via CLI).
3. Set `TMDB_KEY`, `MDBLIST_KEY`, `TRAKT_KEY` in Site settings → Environment variables.
4. Redeploy once more — environment variables only apply to deploys that happen *after* they're saved.
