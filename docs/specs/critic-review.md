# Critic Review — MusicMapper MVP Plan (RALPLAN-DR Short)

- **Reviewer**: Critic
- **Inputs**: spec `deep-interview-musicmapper.md`, plan `musicmapper-mvp-plan.md`, architect review `architect-review.md`
- **Date**: 2026-05-20
- **Mode**: Started THOROUGH, escalated to ADVERSARIAL after pre-commitment hits + systemic non-code-concern gaps

## 1. Verdict

**REVISE** — Architect's 7 surgical revisions are *necessary but insufficient*. Plan is structurally sound and serves Principles 1–5, but ships with (a) an undefended statistical floor on its single primary success metric, (b) a missing legal/privacy posture that blocks Korean-market soft-launch, and (c) two ambiguity seams in the embedding pipeline. None are architecturally fatal; all are concretely fixable inside the 4-week envelope.

**Blocking issue count: 5** (3 from Architect that I'm endorsing + extending, 2 net-new). Easily fixable → REVISE, not REJECT.

## 2. Blocking Issues (MUST-fix before APPROVE)

### B1. AC8 is unverifiable as currently written, and AC8b alone doesn't close the gap (CRITICAL)
- **Wrong**: AC8 `K-factor ≥ 0.5 over 30 days, N≈20`. At N=20, the 95% CI on K is roughly ±0.3. AC8b leading indicator is necessary but not sufficient — no decision matrix.
- **Threatens**: AC8, Principle 3, Decision Driver 2.
- **Fix**: Reframe AC8 as a triple:
  - AC8a (directional): K-factor point estimate at Day 30, reported with 95% CI.
  - AC8b (leading): share-to-create CTR on `/share/[id]` ≥ 15% by Day 14.
  - AC8c (decision matrix): {K<0.2 kill / 0.2≤K<0.4 iterate viral loop / 0.4≤K<0.6 scale tests / K≥0.6 activate v2}.

### B2. `mm_ref` cookie attribution — 2 edge cases unspecified (CRITICAL)
- **Wrong**: Architect's server-side `mm_sid` lookup is correct but doesn't pin:
  1. Multi-share-visit within 24h → which event becomes parent? (Recommend most-recent unless explicit CTA click.)
  2. Pre-existing map for same `mm_sid` → MUST NOT retroactively reparent.
- **Threatens**: AC8 measurement validity, Principle 3.
- **Fix**: `parent_share_id` set on INSERT only if (a) `mm_sid` has `share_events` in last 24h AND (b) the new map is the FIRST map created by this `mm_sid` after that timestamp. Add unit-test fixture for both cases.

### B3. Option C invalidation contains a factual error (CRITICAL for plan integrity)
- **Wrong**: Plan §2 dismisses Option C with "Supabase Auth biases toward auth-first which fights AC9." Supabase Auth is opt-in. The real disqualifier is umap-js + Deno compatibility.
- **Threatens**: Principle 5, §7 ADR integrity.
- **Fix**: Replace with "Supabase Edge Functions run on Deno; `umap-js` compatibility there is untested and a debugging risk we can't afford in Week 1. Vercel route handlers default to Node — known-good."

### B4. No legal/privacy posture (KR PIPA / EU GDPR) — blocks Week 4 launch (MAJOR)
- **Wrong**: §4 Week 4 step 4 soft-launches to KR audience (KakaoTalk groups). Zero privacy policy, cookie consent, or PIPA disclosure.
- **Threatens**: AC9 audit completeness, Week 4 launch readiness, real-world deployability.
- **Fix**: Add `docs/privacy-policy.md` + `app/privacy/page.tsx` (1-page anonymous-first privacy posture). Add footer link. Update AC9 row to include privacy disclosure presence. ~2 hours.

### B5. Embedding pipeline has 2 ambiguity seams (MAJOR)
- **Wrong**:
  1. PCA/UMAP algorithm flip at 50-track boundary is silent — same playlist different ingest → different map.
  2. `genre_vector` construction unspecified (vocabulary, merge policy, dimensionality).
- **Threatens**: AC4, AC5, Architect's map-immutability fix.
- **Fix**:
  - "Always PCA over normalized genre-tag TF-IDF for MVP. UMAP deferred to v2."
  - Vector recipe: vocab = top 200 MB+Last.fm genres in a precomputed snapshot at `lib/embedding/genre-vocab.json`. Per-track TF-IDF over `(artist genres ∪ track tags)`. Missing → zero → "unclassified".
  - Pin lib versions in `package.json`; add `embedding_algorithm_version TEXT NOT NULL` column for re-embed detection.

## 3. Strong Suggestions (improve quality, not blockers)

1. R9 perf budget for comparison view: Canvas N≥100, first paint ≤200ms on iPhone SE 2020.
2. Choose Neon over Vercel Postgres (Architect §5 item 6).
3. AC5 methodology rewrite to blind self-recognition (Architect §5 item 7).
4. Rate-limit `/api/maps` POST: 10 maps / IP / hour with Vercel KV.
5. Data retention paragraph in ADR: `share_events` 90d, maps indefinite until explicit delete endpoint.
6. OG image immutability: cached at edge keyed on map id, never invalidated.
7. Pre-commit Week 4 distribution channels (don't punt as marketing).
8. Add Day 7 milestone with AC8b CTR + map_created count.

## 4. What's Good (genuine)

- §2 Principles tight, well-ordered. Anonymous-first correctly upstream.
- §2 Decision Drivers correctly identify K-factor measurability as forcing a server.
- §3 AC table with file-paths in "Concrete artifacts" column is excellent.
- §4 Week 1–3 milestone shape (Ingest → Viz → Share) is correct dep order.
- §6 Verification commands are runnable `curl + jq` — bar that most plans miss.
- §5 risk register has 8 entries with likelihood/impact/mitigation triples.

## 5. Architect Review — Sufficiency Check

| Architect Revision | Sufficient? | Critic addition |
|---|---|---|
| 1. Server-side attribution via `mm_sid` | Partially | B2 adds 2 edge cases |
| 2. Map immutability + version pinning | Partially | B5 adds: pin the recipe (vocab, vector construction) |
| 3. Resolve Q1/Q2/Q3 into ADR | Yes | Endorsed |
| 4. AC8b leading indicator | Partially | B1 adds decision matrix (AC8c) |
| 5. R9 perf budget | Yes | Endorsed as S1, not blocking |
| 6. AC5 methodology rewrite | Yes | Endorsed as S3 |
| 7. Option C invalidation rewrite | Insufficient | B3 — write the correct disqualifier |

**Net-new Critic blockers Architect missed**: B4 (PIPA), B5 (vocab seam).

**Architect's package unblocks K-factor at 0.5?** Mathematically no at N=20. Architect's AC8b helps; only AC8c decision matrix (B1) makes the metric actionable.

## 6. Verdict Math

- Blocking issues: 5 (all easily fixable in <1 day)
- Deep structural issues: 0
- → **REVISE**, not REJECT.

## 7. Open Questions (low-confidence)

- Spotify Client Credentials for `/v1/playlists/{id}/tracks` post-Nov-2024 — confirm Week 1 Day 1, not Week 2.
- Vercel Edge OG image + Node `/share/[id]` co-existence — 30-min spike before Week 4.
- KR-locale-first launch? Product call.

## Ralplan summary row

- **Principle/Option Consistency**: Pass with caveat (B3).
- **Alternatives Depth**: Fail — Option C invalidation wrong (B3).
- **Risk/Verification Rigor**: Pass with caveat — R4/R6 under-bounded (B1, B2).
- **Deliberate Additions**: N/A (Short mode).
