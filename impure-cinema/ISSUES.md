# Impure Cinema — Issues

A running list, seeded from what's actually outstanding as of the last ship (Trakt integration). Grouped by what kind of attention each needs. Move items to a "Resolved" section (or close them as GitHub Issues) as they're checked off rather than deleting — useful history of what was actually verified vs. assumed.

## Needs live verification (built correctly per docs, never tested against a real response)

- [ ] **Trakt lookup end-to-end.** The two-step flow (search by IMDb ID → resolve Trakt ID → fetch ratings) is built to Trakt's documented API shape but has never hit their actual servers from the build environment. First things to check if ratings look wrong: the `trakt` action in `api.js`, specifically the header format and the shape of the search response.
- [ ] **Standalone file's direct Trakt call may hit CORS.** Unlike the hosted version (which proxies server-side), `ImpureCinema.html` calls Trakt directly from the browser. If Trakt blocks direct browser requests, this fails silently (caught, Trakt data just stays absent) rather than breaking the page — but worth confirming either way.
- [ ] **`metacriticuser` and `rogerebert` score scales.** Best-guess conversions (`.score` preferred, `.value`-based fallback otherwise), never confirmed against a live MDBList response the way `imdb`/`tomatoes`/`metacritic`/`letterboxd` have been. Check a few real films after deploy; adjust `computeVerdict` if a score looks off.

## Needs your visual sign-off

- [ ] **Immiscible and Contested icons never got your eyes on them.** Miscible and Audience Divided went through your Claude Design → review → approve loop. Immiscible and Contested didn't — after round 1 was rejected, they were rebuilt directly from the round-2 diagnosis spec and shipped without a resubmission round. Worth a direct look on-device before considering the icon set fully settled.
- [ ] **Audience Divided v2 vs v1.** v2 (the interlocking-crack version) was flagged as "likely supersedes v1" but the interlocking zigzag detail at 15px was flagged as worth your own check, not just mine.

## Designed but not built

- [ ] **Results-list scores/flags.** Full build plan and mockups exist; the search results list still shows only poster/title/year. This is the single biggest gap between "what's designed" and "what's shipped."
- [ ] **Discourse heat (H) / Wikipedia signal.** Deliberately deferred — P and audience dispersion carried the signal load across all pressure-testing rounds, so this was never load-bearing. Low priority unless real usage shows a gap H would fill.
- [ ] **PWA / "Add to Home Screen" treatment.** Discussed, explicitly deferred at your call — not started.

## Known characteristics, not bugs (worth remembering, not fixing)

- **Films with M near the 40/70 classification boundaries are sensitive to which sources are present.** Uncut Gems moved from M=41 to M=48 depending on whether Trakt data was available for that title, purely from losing one weighted source. This is expected behavior for a weighted average near a threshold, not a defect — but worth remembering if a classification looks like it "flipped" for a borderline film.
- **Critic-side dispersion was tested and deliberately rejected** (it ran backwards — unanimous acclaim scored as maximally "divided"). Documented in the methodology so it doesn't get re-attempted from scratch later.

## Housekeeping

- [ ] **Confirm CHANGELOG.md / SPEC.md merges landed correctly.** I provided paste-ready content earlier but never had visibility into your actual repo files, so there was no real diff — worth a sanity check that nothing got duplicated or misplaced.
- [ ] **"Where to watch" region is hardcoded to US.** Noted early on, never revisited. Low priority unless you have non-US users.
