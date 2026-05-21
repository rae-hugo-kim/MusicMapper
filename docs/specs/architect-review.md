# Architect Review — MusicMapper MVP Plan (RALPLAN-DR Short)

- **Reviewer**: Architect
- **Inputs**: `.omc/specs/deep-interview-musicmapper.md`, `.omc/drafts/musicmapper-mvp-plan.md`
- **Date**: 2026-05-20

---

## 1. Verdict

**APPROVE WITH CONDITIONS.**

The plan is architecturally sound for a 1-person/4-week MVP and faithfully serves the K-factor goal. The stack choice (Next.js + Vercel + Postgres) is defensible. However, three under-bounded risks must be addressed before Critic can sign off: (a) cookie-loss vs share-link integrity contract is implicit, (b) embedding/cluster determinism is asserted but not specified, (c) K-factor measurement validity at N≈20 needs explicit leading-indicator commitment, not just a footnote in R4.

---

## 2. Strongest Steelman Counterargument (Antithesis)

**The stronger choice is Option C: Supabase + a thin Next.js client, not Option A.**

The Planner dismisses C with "auth-first bias fights AC9" (§2). This is backwards: Supabase Auth is opt-in — you simply don't call it. What Supabase actually buys you is:

1. **Anonymous sessions as a first-class primitive** via RLS + `auth.uid() IS NULL` policies. The Planner's plan reinvents this with a hand-rolled `mm_sid` cookie (§4.1) + a custom `anonymous_sessions` table, which is exactly the plumbing AC9 is most likely to leak through.
2. **Realtime out of the box** — when v2 needs "see your friend's map update," you already have it.
3. **No vendor lock to a serverless runtime** with Edge/Node split (the Planner's own Open Question #1 in §8 evaporates if Postgres is just a Postgres connection from anywhere).
4. **Storage + Postgres + Auth in one dashboard** is *measurably* fewer hours for a solo dev than Vercel Postgres + Vercel Blob + custom session.

The CORS argument the Planner uses against C is wrong: ingest still runs server-side in Supabase Edge Functions, identical to Next.js route handlers. The real risk with C is the Edge Functions Deno runtime + UMAP-js compatibility — a real concern, but solvable by running embedding client-side (which the Planner *already* proposes in R7 mitigation, §5).

I still narrowly prefer A because Next.js RSC for `/share/[id]` SSR is cleaner than Supabase + a separate Next.js frontend. But the Planner's invalidation of C is uncharitable and should be hardened or softened.

---

## 3. Tradeoff Tensions

**Tension 1 (primary): Anonymous-first vs durable share URL integrity.**
The plan correctly makes the *share URL* the durable artifact (R6, §5). But §4.3 step 3 attributes derivative maps via a `mm_ref` cookie set on share visit. If the visitor clears cookies between visiting the share URL and creating their map (mobile Safari ITP, incognito tab closure), the derivative is silently un-attributed and K-factor under-counts. This is a **measurement validity bug, not just a UX wart** — and the plan does not name it.

**Tension 2: Embedding stability vs UX expectations.**
The Planner says "deterministic seed" for UMAP (§4.2 step 2). But UMAP-js stability across (a) added tracks to the same playlist and (b) library version bumps is non-trivial. If a user re-ingests the same playlist next month and sees a visibly different map, the "이게 내 음악이네" trust (AC5) breaks. The plan does not specify whether maps are immutable snapshots or re-computed on view.

**Tension 3: K-factor measurement vs deploy-and-iterate ethos.**
N≈20 users / 30 days yields a K-factor with confidence interval roughly ±0.3. The plan acknowledges this in R4 but still treats K≥0.5 as a binary pass/fail in AC8. Either the metric needs Bayesian framing or AC8 should be reframed as "directional evidence + leading-indicator package."

---

## 4. Synthesis Path

**For Tension 1 (cookie loss):** Persist `parent_share_id` attribution *server-side* at the moment of share-visit, keyed on a fresh `mm_sid` (which is HTTP-only and survives the same session). On map creation, look up the most recent `share_event` for the active `mm_sid` within last 24h and attribute. Drop the separate `mm_ref` cookie. This holds anonymous-first (no login) AND durable attribution within a single browser session.

**For Tension 2 (stability):** Make maps **immutable**. Once embedded, store the 2D coordinates in the `maps` row. Re-ingest → new map row, new URL. State this explicitly. This also means share URLs are forever-stable (no "the map I shared yesterday looks different today"). Trade-off: storage cost grows linearly — fine at MVP scale.

**For Tension 3 (statistical floor):** Keep AC8's K≥0.5 as the *directional* north star, but add a parallel AC8b lightweight: "share-to-create CTR on `/share/[id]` ≥ 15%." This leads K-factor by 7–14 days and is statistically valid at N=20.

---

## 5. Architecture-Specific Risks Planner Missed or Under-Bounded

1. **Session-loss attribution bug** (under-bounded R6, §5). See Tension 1. Severity: directly threatens AC8 measurement validity. **Must fix before launch.**

2. **Edge vs Node decision (§8 Q1, unresolved).** Vercel Postgres now ships `@vercel/postgres` which works on Edge. Use **Node runtime for `/share/[id]`** anyway because: (a) embedding pre-render may need Node-only libs, (b) Edge cold-starts on Vercel are not meaningfully faster than Node for a DB-bound RSC route, (c) eliminates a class of "works locally, fails in prod" bugs. Plan should remove this as an open question and lock Node.

3. **Embedding/cluster determinism (§4.2).** "deterministic seed" is asserted but the plan does not pin (a) `umap-js` version, (b) seed source, (c) input-vector canonical ordering. Without these, "same playlist → same map" is luck. Specify in §4.2 or move map immutability into the schema.

4. **Comparison view rendering perf (§4.3 step 2).** Two D3/visx scatter plots side-by-side with N=200 tracks each = ~400 DOM nodes + ~6 cluster hulls. This is fine on desktop, **questionable on mobile Safari** where the comparison view is most viral. No perf budget is set. Recommend: Canvas (not SVG) for ≥100 tracks, OR cap visible tracks per map at 150 with a "show all" toggle.

5. **K-factor validity at N≈20.** Already covered in Tension 3.

6. **Vendor lock / migration path.** R7 mentions Cloudflare Workers as fallback but Vercel Postgres → Cloudflare D1 is a non-trivial migration (different SQL dialect, no Drizzle parity). If lock-in matters, choose **Neon (Postgres) over Vercel Postgres** — same DX, portable, free tier comparable. This is a 30-minute switch *now* and a 3-day switch *later*.

7. **AC5 user-testing methodology gap (§4 Week 4).** "5 friends, 4/5 agree" is a friendship test, not a self-recognition test. Specify: testers should not know the project goal, should be shown map without context, asked "Do you recognize this as your taste?" — otherwise social-desirability bias floors the metric.

---

## 6. Open Questions Resolution (Planner §8)

**Q1: Edge vs Node for `/share/[id]`?**
**Resolve: Node runtime.** Reasoning in §5 item 2. Lock this; remove from open questions.

**Q2: k-means vs HDBSCAN?**
**Resolve: k-means with k=auto.** The Planner's `k=sqrt(n/2) capped at 8` is fine for AC5 self-recognition. HDBSCAN's main win (noise points, varying density) is a *researcher's* win, not a "이게 내 음악이네" win at N≤200 tracks. Defer HDBSCAN to v2 alongside Adjacent Discovery, where density-aware neighborhoods actually matter.

**Q3: OG image generation in Week 4 or fast-follow?**
**Resolve: Week 4, MUST-ship.** OG images are the single highest-leverage K-factor multiplier — Twitter/KakaoTalk share previews drive 2-3x click-through over text-only links. Cut the "polish backlog" loading skeletons (R5 covers that) and ship `app/share/[id]/opengraph-image.tsx`. Vercel's `@vercel/og` makes this ~3 hours, not 0.5–1 day.

**Q4: N=20 achievable from personal network?**
**Resolve: Defer with rationale.** This is a marketing question, not an architecture question. Architect declines. *However*, the plan should add a contingency: if N<10 at Day 7, expand channels (Reddit r/Music, HackerNews Show HN, Korean dev communities) — name the list now so launch day isn't blocked on this decision.

---

## 7. Required Revisions (Surgical)

The Planner MUST apply these before Critic review:

1. **§4.3 step 3 — Fix attribution to use server-side `share_events` lookup keyed on `mm_sid` (HTTP-only cookie), not a separate `mm_ref` cookie.** Update §3 AC8 testability row to match. Acknowledge cookie-loss as a known measurement floor.

2. **§4.2 step 2 — Specify map immutability.** Add to schema: `maps.embedding_2d` is written once at creation, never recomputed. Re-ingesting produces a new `maps` row + URL. Pin `umap-js` version in `package.json`. Document deterministic seed source (e.g., hash of sorted track IDs).

3. **§8 — Resolve Q1 (Node runtime), Q2 (k-means), Q3 (OG image MUST-ship in Week 4).** Remove from open questions; move into ADR §7.

4. **§3 AC8 row — Add AC8b leading indicator: share-to-create CTR ≥ 15% on `/share/[id]` measurable at Day 7.** This makes the metric package statistically defensible at N=20.

5. **§5 add R9: comparison view perf budget.** "≤200ms first paint on mid-tier mobile for ≤150 tracks per side; canvas rendering for N≥100." Without this, mobile viral loop dies silently.

6. **§4 Week 4 step 3 — Rewrite AC5 user-test methodology to remove confirmation bias.** Testers should not know the goal; question should be blind self-recognition.

7. **§2 — Either harden or soften the Option C invalidation.** Current language ("auth-first bias fights AC9") is wrong on the merits. Either drop it or add the real argument: "Supabase Edge Functions on Deno create a UMAP-js compatibility risk we don't want to debug at hour 60 of week 1."

---

## References

- `.omc/specs/deep-interview-musicmapper.md:38-43` — constraint envelope (anonymous-first, 4 weeks)
- `.omc/specs/deep-interview-musicmapper.md:54-62` — AC1–AC9 verbatim
- `.omc/drafts/musicmapper-mvp-plan.md:42` — Recommended Option A
- `.omc/drafts/musicmapper-mvp-plan.md:44-48` — invalidation rationale for B/C (concern in §2 and §7 here)
- `.omc/drafts/musicmapper-mvp-plan.md:107-122` — Week 2 embedding step (concern in §5 item 3, §7 item 2)
- `.omc/drafts/musicmapper-mvp-plan.md:140-144` — derivative attribution via `mm_ref` cookie (concern in §3 Tension 1, §7 item 1)
- `.omc/drafts/musicmapper-mvp-plan.md:188-200` — Risks table (R4, R6 under-bounded per §5 items 1, 5)
- `.omc/drafts/musicmapper-mvp-plan.md:269-275` — Open Questions (resolved in §6)
