# MusicMapper MVP Implementation Plan (RALPLAN-DR Short)

- **Source spec**: `.omc/specs/deep-interview-musicmapper.md` (ambiguity 13.5%, PASSED)
- **Mode**: RALPLAN-DR Short consensus, 1 person / 4 weeks
- **Status**: **FROZEN** at iteration 3.2 — plan is now gated behind a Wizard of Oz thin slice. Engineering layer (iter 1-2): 5 Critic blockers (B1-B5) closed. Product layer (iter 3): hypothesis, AC5.5, gen_depth, AC8c recalibration, (b)/(c) trace. Second-order (iter 3.1): distribution-aware gate, cascade sample floor, Wrapped mechanic gap + KR context. Bet-value gate (iter 3.2): thin slice methodology at `.omc/specs/thin-slice-wizard-of-oz.md`, pre-decision criteria at `docs/thin-slice-pre-decision.md`. **이 plan의 kickoff은 thin slice 결과가 pre-decision 행렬에 통과한 후에만 시작.**
- **Plan owner**: solo developer (nights/weekends OK)

---

## 1. Requirements Summary

MusicMapper is a greenfield **web-only** tool that lets an anonymous visitor turn a playlist (YouTube URL, Spotify public playlist, or manually-typed tracks) into a personal **2D music map** — a scatter-plot of tracks grouped into bubble clusters by genre/taste — and then share that map as a public URL. When a friend opens the share URL, they land on a **comparison view** ("my map vs. shared map"); from there they can build their own map in 1–2 clicks with zero forced login. The MVP's deliverable is *the viral creation/share loop itself*, not a recommender.

The single primary success metric is **K-factor ≥ 0.5 over a 30-day window** (derivative maps / source maps). Adjacent Discovery (collaborative filtering) and node-edge graphs are explicitly deferred to v2. The whole flow MUST be auditable as anonymous (AC9): no signup/login gate on ingest, map creation, share, comparison view, or derivative map creation. Constraint envelope is 1 dev × 4 weeks, so the stack and feature set are chosen for solo throughput, not headroom.

### Product hypothesis (explicit)

MusicMapper bets on **path (a) — "music taste as a sharable social artifact"**, *not* on (b) digging engine or (c) state-based automatic curation. The baseline evidence for (a) is the annual **Spotify Wrapped Instagram-story spike**: people enthusiastically share visual representations of their taste *once a year*. MusicMapper's hypothesis is that the same impulse can be made **year-round** if the artifact (i) generates in seconds, (ii) is visually distinctive enough to feel personal, and (iii) lives on a public URL friends can open without an app.

**Honest baseline-mechanic gap** (the hypothesis carries this burden visibly): Wrapped's virality rests on three *structural* advantages MusicMapper does NOT inherit — (i) Spotify pre-aggregates user data so sharing has zero input cost, (ii) it's a **seasonally-coordinated event** (December social momentum: peers are sharing simultaneously), and (iii) the output is optimized for the **Instagram Story native format** (vertical animated narrative across 10 frames). MusicMapper imposes URL pasting (input cost > 0), runs year-round (no socially-coordinated week), and outputs a single OG card + interactive page (not native Story format). The (a) bet is therefore: *the comparison-view social mechanic ("내 지도 vs 친구 지도" curiosity) substitutes for the three lost Wrapped advantages*. If that mechanic does not substitute, the bet fails — which is what AC8b CTR < 10% would diagnose, ahead of the 30-day AC8 window.

**Korea-specific context**: Spotify's KR market share is small; the closer local reference is **Melon Music Diary**, which has lower share-virality than Wrapped. KR-first soft-launch therefore inherits *less* baseline taste-sharing behavior than the global Wrapped benchmark suggests — the bet is structurally harder in KR than in markets where Wrapped is the default annual ritual.

Path (b) is well-served by existing recommenders; path (c) is a different product (a music-state inbox, not an artifact) and is a legitimate alternative bet we are *not* taking in this build. Calibration: at N=20 testers, K=0.5 is **stretch** (Dropbox referral with 2GB incentive landed K≈0.5; consumer apps without explicit share mechanics average K<0.2), so the decision matrix (AC8c) reads any K≥0.4 as a strong v2 signal, not a "scale tests" middle band.

---

## 2. RALPLAN-DR Summary

### Principles (5)

1. **Anonymous-first.** Every flow MUST be completable in an anonymous session (AC9). Login is an *option*, never a gate.
2. **Ship the viral loop, not the recommender.** The deliverable is K-factor, not embedding quality. Optimize for share-friction reduction over algorithmic sophistication.
3. **MVP must be measurable on K-factor.** Every share, visit, and derivative creation MUST be instrumented before launch — otherwise AC8 is unverifiable.
4. **Minimum-viable embedding.** A "good enough self-recognition" map (AC5, 4/5 user agreement) beats a sophisticated one we can't ship in 4 weeks. PCA/UMAP over genre tag vectors is sufficient.
5. **Solo-throughput stack.** Pick boring, integrated, zero-ops tooling. Every hour spent on infra is an hour stolen from the share-loop.

### Decision Drivers (Top 3)

1. **1-person / 4-week scope** — eliminates anything requiring custom infra, queues, or non-managed databases.
2. **K ≥ 0.5 measurability** — requires server-side event store + shareable persistent URL + cookie-less attribution path. Pure-static can't do this.
3. **No-login constraint (AC9)** — server must accept writes from anonymous sessions (signed cookie or opaque session id), not gated on auth provider.

### Viable Options for Stack/Architecture

| | Option A: Next.js + Vercel + Vercel Postgres + Vercel Blob | Option B: SvelteKit + Cloudflare Workers + D1/KV | Option C: Static React + Supabase |
|---|---|---|---|
| **Pros** | Single deploy target; route handlers cover ingest+share APIs; Postgres is comfortable for ShareEvent joins; Vercel Analytics for free K-factor instrumentation; widely-known so least research overhead | Cheapest at scale; edge-distributed share URLs feel instant; KV is a natural fit for serialized maps | Supabase Auth+DB+Storage in one; RLS handles anonymous policies; great DX for solo dev |
| **Cons** | Vercel Postgres pricing past free tier; cold starts on serverless functions can hurt first-visit comparison view | D1 still maturing; SvelteKit ecosystem smaller; Workers' 50ms CPU limit may cramp UMAP runtime | Static+Supabase splits ingest logic between client (CORS pain with YouTube/Spotify) and edge functions; harder to keep secrets out of client |
| **4-week fit** | High — most "boring" path | Medium — requires more research time | Medium — anonymous-with-RLS works but cookie session plumbing is fiddly |

**Recommended: Option A (Next.js + Vercel + Vercel Postgres + Vercel Blob).**

**Invalidation rationale for B and C**:
- **B** loses 2–3 days to D1/Workers learning curve and to client-side UMAP workarounds (CPU cap). The cost savings are immaterial at MVP scale (<1k users for K-measurement).
- **C** forces ingest onto Supabase Edge Functions which run on **Deno**; `umap-js` (and ml-pca) compatibility under Deno is untested and a debugging risk we can't afford in Week 1. Vercel route handlers default to Node — known-good for our embedding libraries. **This is the disqualifier**, not Supabase Auth (which is opt-in and can be ignored entirely without "biasing" the architecture).

Both remain viable if Critic flags Vercel lock-in or pricing concerns; the plan does not require A's choice to be locked before Week 1 Day 1.

### ADR — Database: Neon over Vercel Postgres (S2)

Use **Neon serverless Postgres** (not Vercel Postgres). Same Postgres dialect, same Drizzle driver, ~30-min swap. Drivers: (a) reduce Vercel platform lock-in for the data tier (DB is the hardest piece to migrate later), (b) Neon's branching helps preview deploys, (c) generous free tier covers MVP. Trade-off: one extra dashboard. Does **not** change architecture.

---

## 3. Acceptance Criteria (carried forward verbatim + testability notes)

| ID | Criterion (verbatim) | Concrete artifacts | Verification |
|----|-----------------------|---------------------|--------------|
| **AC1** | Ingest: YouTube 플레이리스트 URL을 붙여넣으면 곡 목록이 추출되어 지도 생성 입력으로 사용된다 (로그인 없이) | `app/api/ingest/youtube/route.ts` (POST `{url}` → `{tracks: Track[]}`), `lib/ingest/youtube.ts` (uses `youtube.playlistItems.list` with API key) | E2E: paste a real public playlist URL in incognito, assert ≥1 track persisted into Map |
| **AC2** | Ingest: Spotify 공개 플레이리스트 URL/ID로 동일 동작 | `app/api/ingest/spotify/route.ts`, `lib/ingest/spotify.ts` (Client Credentials token cache in `lib/spotify-token.ts`) | E2E: paste public Spotify playlist URL; assert tracks returned; assert NO Spotify user OAuth redirect |
| **AC3** | Ingest: 곡 제목/아티스트를 수동 입력해 외부 메타데이터(MusicBrainz/Last.fm) 보강 후 지도에 반영 | `app/api/ingest/manual/route.ts`, `lib/enrich/musicbrainz.ts`, `lib/enrich/lastfm.ts`, fallback chain in `lib/enrich/index.ts` | Unit: feed a known-good title/artist pair, assert tags + genre populated; E2E: manual entry of 5 tracks renders into scatter |
| **AC4** | Visualization: 입력된 곡들이 2D 산점도로 시각화되고, 인접한 곡들이 버블 클러스터로 묶여 보인다 | `lib/embedding.ts` (**PCA-only over normalized genre-tag TF-IDF**, recipe v1 — see §4.2 step 2 and `lib/embedding/genre-vocab.json`); `lib/cluster.ts` (k-means, k auto); `components/MapCanvas.tsx`; DB column `maps.embedding_algorithm_version` defaults `'pca-tfidf-v1'` | Visual: load `/map/[id]` for a 30-track playlist; assert ≥2 visually distinct bubble groups; snapshot test of fixture; assert determinism (same input → same coordinates across runs). |
| **AC5** | Visualization: 사용자가 "오, 이게 내 음악이네"라고 느낄 수 있는 자기 인식 수준의 매핑 (정성, 5인 이상 4/5 동의) — **blind self-recognition**. **Note**: AC5 measures *identifiability* of one's own taste, NOT *shareworthiness*. AC5.5 bridges that gap. | User test script `docs/user-test-script.md`; **5 testers each create their map AND a "decoy" map from a stranger's playlist (provided by tester)**; tester is shown both maps with **labels stripped** and asked "which one feels like yours?" before being asked the Likert "이게 내 음악 같다" question on the correctly-identified one. Survey via Google Form. | Qualitative: 4/5 must (a) correctly self-identify their map in the blind A/B and (b) score ≥4 on 5-point Likert. The blind A/B removes friendship/expectation bias. |
| **AC5.5** | **Share intent**: 자기 지도를 본 직후 "이 지도를 친구에게 공유하고 싶다 (1-5)". **N=5 distribution-aware gate** (평균 게이트는 bimodal에 약함): **5명 중 3명 이상이 ≥4** AND **median ≥ 4**. 이 지표가 있으면 AC5 PASS + AC8 FAIL 시 진단 분리 가능 (임베딩 문제 vs 공유 동기 부재). | Same user test session as AC5; 추가 Likert 질문 1개 + open-ended "왜 그 점수인가요?" 1개; survey form 동일. Zero incremental cost. | Qualitative: count ≥3 score ≥4 AND median ≥4. **Bimodal 신호 (예: 2-2-5-5-5) 발생 시 — 평균은 PASS여도 — split-reason 분석 필수**: 환호와 무관심이 갈리는 segment를 open-ended로 진단하고 AC8 launch 전 share CTA/OG image/copy 우선 이터레이션. |
| **AC6** | Share: 생성된 지도가 익명 열람 가능한 고유 URL로 공유된다 | `app/share/[id]/page.tsx` (RSC, no auth middleware), `app/api/maps/route.ts` POST returns `{id, share_url}`; map stored in `maps` table | E2E: create map in browser A, open share URL in browser B incognito, assert map renders identically |
| **AC7** | Share: 공유 URL 방문자가 "내 지도 vs 공유자 지도" 비교 뷰; 방문자도 로그인 없이 자기 지도 생성 시작 가능 | `app/share/[id]/page.tsx` renders dual `<MapCanvas>` (visitor's map if cookie session has one, else CTA panel `<CreateYourMapCTA/>`); `components/ComparisonView.tsx` | E2E: incognito visitor on share URL sees CTA → 1-click to `/create` → submits playlist → lands back on comparison view |
| **AC8a** | 베타 30일 K-factor **점추정치**를 95% Wilson 신뢰구간과 함께 보고 (1세대) + **2세대 K** (derivative-of-derivative / 1세대 derivative) 별도 보고 | `lib/analytics.ts` events: `map_created`, `map_shared`, `share_visited`, `derivative_map_created` (with `parent_share_id`); Drizzle column **`maps.generation_depth INT NOT NULL DEFAULT 0`** (0 = original, 1 = first derivative, 2+ = cascade); `app/admin/k-factor/page.tsx` SQL view with Wilson CI computation + per-generation breakdown | Day 30 SQL: `SELECT generation_depth, derivative_count, source_count, wilson_lower_95, wilson_upper_95 FROM k_factor_30d_by_gen`. Cascade health: K(gen-2) / K(gen-1) ≥ 0.5 → sustainable loop; < 0.5 → "튀고 끝" 패턴. CI must report even if point estimate < 0.4. |
| **AC8b** | **Leading indicator**: `/share/[id]` share-to-create CTR ≥ **15%** by Day 14 (visits → derivative-create-started). **Stretch target — Dropbox referral was K≈0.5 with a 2GB incentive**; consumer apps without explicit share rewards average K<0.2. | Same events table; CTR = `count(derivative_create_started WHERE parent_share_id IS NOT NULL) / count(share_visited)` | Day 14 dashboard checkpoint. If <10% by Day 14, AC8a is almost certainly going to fail — trigger early iteration of share CTA copy/layout. |
| **AC8c** | **Decision matrix** (applied at Day 30 to AC8a result) — **recalibrated** against consumer-app benchmarks: K=0.4 is already extraordinary, not "scale tests" middle band | `docs/k-factor-decision-matrix.md` recording the bands below | `K < 0.15` → **KILL** (below typical consumer baseline; post-mortem, don't pursue v2). `0.15 ≤ K < 0.25` → **iterate viral loop** (share CTA copy, OG image, comparison-view layout) for another 30-day window. `0.25 ≤ K < 0.4` → **scale tests** (broader recruitment, channel diversification, paid probe). `K ≥ 0.4` → **confirmed → activate v2** (Adjacent Discovery, audio features, optional login). **Cascade gate sample-size floor**: cascade caveat (gen2 K < 0.5 × gen1 K → 한 밴드 강등) is **suppressed if gen2 sample < 10** — at MVP scale (N=20 baseline), gen2 ratio CI is too wide to fire reliably and would generate false strong-downgrade signals. In that regime, surface cascade ratio as *informational metric only*; band-downgrade trigger activates only after gen2 sample ≥10 (i.e., from scale-phase recruitment onward). |
| **AC9** | 어느 흐름에서도 강제 회원가입/로그인이 없다 (감사 가능); **개인정보 처리방침 및 쿠키 공시 게재** | No `next-auth` / Clerk middleware on any core route; audit doc `docs/anonymous-flow-audit.md` listing every protected route (should be empty for core flows); `docs/privacy-policy.md` + `app/privacy/page.tsx` + site-wide footer link | Audit: grep for auth gates; manual test in incognito for ingest, create, share, comparison, derivative — all must succeed without login; load `/privacy` and confirm footer link present on every page. |

---

## 4. Implementation Steps (4 weekly milestones)

### Week 1 — Skeleton + Ingest (manual + YouTube)

**Goal**: A user in incognito can paste a YouTube URL OR type tracks manually and see a (placeholder) map at `/map/[id]`.

1. **Project init + persistence layer**
   - Files: `package.json`, `next.config.ts`, `app/layout.tsx`, `lib/db.ts` (Drizzle or Kysely on Vercel Postgres), `drizzle/schema.ts` (tables: `maps`, `tracks`, `share_events`, `anonymous_sessions`)
   - Cookie session: `lib/session.ts` sets HTTP-only signed cookie `mm_sid` on first visit; no auth dependency.
   - Verify: `pnpm dev` → first request sets cookie; DB migration `pnpm drizzle:push` creates 4 tables.

2. **Manual-entry ingest (AC3, partial)**
   - Files: `app/create/page.tsx` (form), `app/api/ingest/manual/route.ts`, `lib/enrich/musicbrainz.ts`, `lib/enrich/lastfm.ts`.
   - Fallback chain: MusicBrainz → Last.fm → bare (title/artist only).
   - Verify: POST with 3 tracks returns enriched `Track[]` with `genres: string[]`.

3. **YouTube ingest (AC1)**
   - Files: `app/api/ingest/youtube/route.ts`, `lib/ingest/youtube.ts` (uses `googleapis` or raw fetch to `playlistItems.list`).
   - Env: `YOUTUBE_API_KEY` in `.env.local`.
   - Title parsing: split "Artist - Title" heuristic in `lib/ingest/parse-yt-title.ts`; fall back to MusicBrainz lookup on raw title.
   - Verify: real public playlist URL → tracks array ≥ 1; quota usage logged.

4. **Map persistence + placeholder render**
   - Files: `app/api/maps/route.ts` (POST create), `app/map/[id]/page.tsx` (server component reads DB).
   - Placeholder: render tracks as a list before embedding lands in Week 2.
   - Verify: end-to-end from `/create` form → DB row → `/map/[id]` page renders.

**Week 1 exit criteria**: AC1 + AC3 ingest paths return persisted maps; AC9 audited so far (no login anywhere).

---

### Week 2 — Visualization + Spotify ingest

**Goal**: AC2 + AC4 done. `/map/[id]` shows a real scatter plot with bubble clusters.

1. **Spotify Client Credentials ingest (AC2)**
   - Files: `app/api/ingest/spotify/route.ts`, `lib/ingest/spotify.ts`, `lib/spotify-token.ts` (in-memory + DB-backed token cache, 1h TTL).
   - Use `/v1/playlists/{id}/tracks` with `Authorization: Bearer <app-token>`.
   - Verify: public playlist URL ingests; no user OAuth redirect; token cache reused across requests.

2. **Genre vector + 2D embedding (AC4) — deterministic PCA-only**
   - **Algorithm**: Always use **PCA** (`ml-pca`, pinned version in `package.json`) over normalized genre-tag **TF-IDF** vectors. **UMAP is deferred to v2** per Principle 4 — removes a non-deterministic, harder-to-debug dependency from the critical path.
   - **Vocabulary**: `lib/embedding/genre-vocab.json` — top **200 genres** ranked by frequency in a precomputed MusicBrainz + Last.fm corpus snapshot. The snapshot file is committed at build time (not fetched at runtime), so the vocabulary is reproducible across deploys.
   - **Per-track vector recipe**: TF-IDF over the multiset `(artist genres ∪ track tags)` projected into the 200-dim vocabulary. Out-of-vocab tags ignored.
   - **Missing-data path**: if a track yields zero in-vocab tags → assign the zero vector → cluster as `"unclassified"` (gray bubble, not error).
   - **Files**: `lib/embedding.ts` (PCA driver), `lib/embedding/tfidf.ts` (vocab loading + vectorization), `lib/embedding/genre-vocab.json` (committed snapshot), `scripts/build-genre-vocab.ts` (one-off; documents how the snapshot was produced).
   - **Schema**: Drizzle column `maps.embedding_algorithm_version TEXT NOT NULL DEFAULT 'pca-tfidf-v1'` — so future algorithm bumps are auditable and old maps don't silently re-render with new coordinates.
   - Verify: feed fixture of 30 tracks twice in a row → identical output coordinates (determinism); fixture with all-OOV tags → all points at origin in `"unclassified"` cluster; snapshot test on `lib/embedding/__tests__/fixture.test.ts`.

3. **Clustering (AC4)**
   - Files: `lib/cluster.ts` (k-means with `k = sqrt(n/2)` capped at 8; use `ml-kmeans`).
   - Verify: fixture produces 3–5 clusters; cluster assignments stored on Map row.

4. **Scatter + bubble cluster UI (AC4)**
   - Files: `components/MapCanvas.tsx` (visx `XYChart` or D3 directly), `components/ClusterHull.tsx` (convex hull or simple radius), `components/TrackTooltip.tsx`.
   - Color by cluster; track name on hover.
   - Verify: load `/map/[id]` for the YouTube playlist from Week 1; eyeball clustering.

**Week 2 exit criteria**: AC1, AC2, AC3, AC4 visible end-to-end. AC5 deferred to Week 4 user testing.

---

### Week 3 — Share, comparison view, K-factor instrumentation

**Goal**: AC6 + AC7 + analytics for AC8.

1. **Share URL (AC6)**
   - Files: `app/share/[id]/page.tsx` (anonymous-readable), `app/api/maps/[id]/share/route.ts` (POST records `share_events` row, returns shareable URL).
   - URL format: `https://musicmapper.app/share/{nanoid}` (short, copy-pasteable).
   - Verify: incognito browser B opens URL from browser A → map renders.

2. **Comparison view (AC7)**
   - Files: `components/ComparisonView.tsx`, `components/CreateYourMapCTA.tsx`.
   - Layout: side-by-side (or stacked on mobile) — left = shared map, right = visitor's map OR CTA card with "Make your own in 30 seconds".
   - Visitor with existing `mm_sid` cookie + ≥1 map → auto-load their most recent map on the right.
   - Verify: incognito → CTA path; cookie-warm session → dual map path.

3. **Derivative-map attribution (AC8 prerequisite) — explicit rules**
   - **Attribution rule (precise)**: On `POST /api/maps`, set `parent_share_id` on the new map row IF AND ONLY IF **both** of these hold for the current `mm_sid`:
     (a) there exists a `share_events` row for this `mm_sid` within the **last 24 hours** (sliding window from `now()`), AND
     (b) this new map is the **first** map created by this `mm_sid` *strictly after* the timestamp of that `share_events` row.
   - **Tie-break on multiple share visits in 24h**: pick the **most-recent** `share_events` row by `viewed_at`, UNLESS there is an **explicit CTA click event** (`share_cta_clicked`) — in which case that share wins regardless of recency.
   - **No retroactive reparenting**: maps created by this `mm_sid` *before* a share visit are NOT updated. The query MUST filter `maps.created_at > share_events.viewed_at`.
   - Files: `lib/attribution.ts` (pure function `resolveParentShareId(mm_sid, now) → string | null`), DB indexes on `share_events(visitor_session, viewed_at DESC)` and `maps(visitor_session, created_at)`.
   - Verify — two required unit-test fixtures in `lib/attribution.test.ts`:
     1. **Multi-share-visit-within-24h**: same `mm_sid` visits share A at t-2h, share B at t-1h (no CTA click); then creates a map. Expect `parent_share_id = B`. Repeat with CTA-click on A → expect `parent_share_id = A`.
     2. **No retroactive reparenting**: same `mm_sid` creates map M1 at t-3h, then visits share A at t-1h. M1's `parent_share_id` MUST remain NULL. A subsequent map M2 after the share visit may attribute to A.
   - E2E spot-check: synthetic flow — open share URL, create map, assert `parent_share_id` populated correctly.

4. **Rate-limiting on map creation (S4 — abuse hardening)**
   - `POST /api/maps` rate-limited to **10 maps per IP per hour** via Vercel KV sliding window (`@upstash/ratelimit` + `@vercel/kv`).
   - File: `lib/rate-limit.ts` (single helper); applied as middleware on `app/api/maps/route.ts`.
   - On limit hit: 429 with friendly copy ("Slow down — try again in N minutes"). Does NOT block `/share/[id]` reads.
   - Verify: hit the endpoint 11x in <1h via curl → 11th returns 429; share-URL reads still 200.

5. **K-factor dashboard (AC8 measurement plumbing)**
   - Files: `app/admin/k-factor/page.tsx` (basic-auth or env-gated), SQL view `k_factor_30d` in a migration — view must compute **point estimate + Wilson 95% CI bounds** (AC8a).
   - Events table: `share_events(id, map_id, visitor_session, viewed_at, derivative_map_id NULLABLE)`; also `share_cta_clicked` events for AC8b CTR.
   - Dashboard surfaces: AC8a (K + CI), AC8b (share-to-create CTR), AC8c decision band (auto-highlight current band).
   - Verify: insert 3 source maps + 2 derivatives via SQL; dashboard reports `K = 2/3 ≈ 0.67` with Wilson CI roughly `[0.21, 0.94]` (sanity).

**Week 3 exit criteria**: AC6, AC7, AC8 instrumentation operational (metric value pending Week 4+ data).

---

### Week 4 — Polish, deploy, soft-launch

**Goal**: AC5 (qualitative), production deploy, recruit 20 testers, start K-factor 30-day window.

1. **Deploy to Vercel production**
   - Files: `vercel.json` (env mapping), `README.md` deployment section.
   - Secrets: `YOUTUBE_API_KEY`, `SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`, `LASTFM_API_KEY`, `MM_COOKIE_SECRET`, `DATABASE_URL` set in Vercel.
   - Verify: production URL serves all 4 ingest types end-to-end.

2. **AC9 audit + privacy/PIPA posture (≈2h)**
   - **Anonymous-flow audit**: `docs/anonymous-flow-audit.md` — list every route, mark auth-required/anonymous-OK. Incognito flows for create / share / derivative all green.
   - **Privacy policy (1-page, anonymous-first)**: `docs/privacy-policy.md` (source of truth) + `app/privacy/page.tsx` (renders the markdown). Cover: (i) only first-party cookie `mm_sid` (purpose, retention), (ii) what we store (playlists, maps, share events) and for how long, (iii) **no third-party tracking beyond Vercel Analytics**, (iv) third-party APIs we call (YouTube, Spotify, MusicBrainz, Last.fm) and that we don't send personal data to them, (v) deletion contact (mailto link).
   - **Cookie disclosure**: a single non-blocking footer note "We use a first-party cookie to remember your maps. Details in [Privacy](/privacy)." — NOT a consent modal (no third-party tracking → no consent gate required, but disclosure is good PIPA hygiene).
   - **Site-wide footer link**: add `<PrivacyFooterLink/>` to `app/layout.tsx`.
   - Verify: load `/privacy` on prod; footer link present on `/`, `/create`, `/map/[id]`, `/share/[id]`.

3. **AC5 user testing**
   - Files: `docs/user-test-script.md`, share Google Form link.
   - Recruit 5 friends; each creates one map; complete Likert survey.
   - Verify: ≥4/5 agree "이게 내 음악 같다" (≥4 on 1–5 scale). If <4/5, iterate on enrichment fallback or embedding parameters within Week 4.

4. **Soft-launch to ~20 users + K-factor window start**
   - Channels: personal Twitter/X, 2 small Discord/KakaoTalk groups.
   - File: `docs/launch-channels.md` (where + when posted).
   - Tag launch with calendar marker; **start of 30-day K-factor measurement window**.
   - Verify: admin dashboard shows incoming `map_created` + `map_shared` events.

5. **Polish backlog (time-permitting)**
   - Loading skeletons on `/map/[id]`.
   - Better track-title parsing for YouTube ("Official Music Video" stripping).
   - Open-Graph image generation for share URLs (`app/share/[id]/opengraph-image.tsx`) — *high leverage on virality*.

**Week 4 exit criteria**: all 9 AC verifiable; K-factor measurement window open; AC8 result lands 30 days post-launch.

---

## 5. Risks and Mitigations

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|------------|
| R1 | **YouTube Data API quota** (10k units/day default; `playlistItems.list` = 1 unit but pagination + retries adds up) | Medium | High (blocks AC1) | Cache playlist→tracks for 24h in DB; surface "try again later" if 403; request quota increase before launch; consider scraping fallback (ToS risk — document and skip for MVP) |
| R2 | **Spotify Client Credentials lifecycle** (1h token, app-level rate limits) | Medium | Medium | Centralized token in `lib/spotify-token.ts` with mutex/lock; exponential backoff on 429; per-IP throttle on `/api/ingest/spotify` to prevent abuse |
| R3 | **2D embedding quality with sparse metadata** (manual entries may have no MB/Last.fm hit) | High | Medium (threatens AC5) | Multi-fallback enrichment chain; if zero genres, place track in "unclassified" bubble (visible as gray cluster, not error); explicit copy: "Add more tracks for better clustering" |
| R4 | **K-factor validity with N≈20** | High | High (AC8 measurement noise) | Set expectations: AC8 is *directional* at N=20; document confidence intervals; complement with leading indicators (share clicks per map, CTA click-through on `/share`) in dashboard |
| R5 | **Cold start for first-time visitor** (must make a map fast — comparison view requires speed) | Medium | High (kills K-factor) | Time-to-first-map target ≤ 30s including ingest; preload sample playlist on `/create`; lazy-load embedding lib; Vercel Edge for `/share/[id]` |
| R6 | **Anonymous session loss** (clearing cookies → lost maps) | High | Medium (annoying but not blocking) | Document explicitly: "Your maps live on shared URLs — bookmark them"; show share URL prominently after creation; *do not* add a login as the fix — that would violate AC9 |
| R7 | **Vercel free-tier limits** (function execution time, DB rows) | Low | Medium | Embedding offloaded to client-side worker for ≥50 tracks; monitor Vercel usage daily during soft-launch; have Option B (Cloudflare Workers) as fallback migration target |
| R8 | **Title-only YouTube tracks lose enrichment** | High | Medium (AC4 quality) | Heuristic parser + MB fuzzy search; mark low-confidence tracks with subtle UI hint |
| **R9** | **Comparison view perf budget on low-end mobile** (two maps rendered side-by-side; visitor's likely on phone — share virality dies on slow first paint) | Medium | High (kills AC8b/AC8c) | **Budget**: Canvas (not SVG) for N≥100 tracks; **first paint ≤200ms on iPhone SE 2020** for an N=100 + N=100 dual map; stacked layout on viewport <640px (don't render two canvases simultaneously on tiny screens); lazy-mount visitor's map below the fold if not in cookie session. Verify with Vercel Speed Insights + a manual run on a real iPhone SE 2020 (or Chrome DevTools "Moto G Power" throttle as proxy). |

---

## 6. Verification Steps

### Per-AC checks (run before Week 4 launch)

```bash
# AC1
curl -X POST localhost:3000/api/ingest/youtube \
  -H "Content-Type: application/json" \
  -d '{"url":"https://www.youtube.com/playlist?list=PLxxx"}' \
  | jq '.tracks | length' # expect ≥ 1

# AC2
curl -X POST localhost:3000/api/ingest/spotify \
  -d '{"url":"https://open.spotify.com/playlist/xxx"}'

# AC3
curl -X POST localhost:3000/api/ingest/manual \
  -d '{"tracks":[{"title":"Bohemian Rhapsody","artist":"Queen"}]}' \
  | jq '.tracks[0].genres' # expect non-empty

# AC4 (visual)
# Open /map/{id} → screenshot → confirm ≥2 visible clusters
# Snapshot test: pnpm test components/MapCanvas.test.tsx

# AC6 (incognito test)
# Browser A: create map; copy share URL
# Browser B (incognito): open share URL; assert map renders, no login prompt

# AC7
# Browser B (incognito) on share URL → click "Make your own" → submit playlist → assert lands on comparison view

# AC8 (post-30d)
psql $DATABASE_URL -c "SELECT * FROM k_factor_30d"

# AC9 (audit)
# Manual incognito sweep: /, /create, /map/[id], /share/[id], derivative flow
# grep -r "auth()" app/ # expect zero hits on core flows
```

### End-to-end smoke (manual, before launch)

1. Incognito → `/create` → paste YouTube URL → see map.
2. Click "Share" → copy URL.
3. Open URL in second incognito window → see comparison CTA.
4. Click "Make your own" → paste Spotify URL → land on comparison view with both maps.
5. Open admin dashboard → confirm `share_events` row + `derivative_map_id` populated.

### K-factor dashboard verification

- Seed DB with 10 source maps + 5 derivatives via SQL fixture (`scripts/seed-k-factor.sql`).
- Load `/admin/k-factor` → expect `K = 0.5` exactly with a populated **Wilson 95% CI** (AC8a).
- Verify 30-day window math by inserting one derivative dated 31 days ago → expect that row excluded.
- Verify AC8b CTR: insert 20 `share_visited` + 4 `derivative_create_started` events with linking `parent_share_id` → CTR = 20%.
- Verify AC8c band highlighting: stub K = 0.15 → "KILL" band; K = 0.3 → "iterate"; K = 0.5 → "scale tests"; K = 0.7 → "v2".

### Post-launch checkpoints (S8 — early-signal triage)

- **Day 7 milestone** (`docs/day-7-checkpoint.md` template): record (a) cumulative `map_created` count, (b) AC8b CTR snapshot, (c) any operational issues (quota, errors, perf). **Hard gate**: if `map_created < 10` OR CTR < 5% at Day 7, hold further channel pushes and diagnose before continuing the 30-day window. This prevents burning the K-factor window on a broken funnel.
- **Day 14 milestone**: AC8b CTR ≥ 15% target evaluation (per AC8b row). Below 10% triggers viral-loop iteration immediately, don't wait for Day 30.
- **Day 30**: AC8a final report (K + Wilson CI) + AC8c decision matrix applied → next-step decision recorded in `docs/k-factor-day-30-report.md`.

---

## 7. ADR (final consensus record)

### Decision

Build the MusicMapper MVP on **Next.js (App Router, Node runtime) on Vercel**, with **Neon serverless Postgres** (Drizzle ORM) and **Vercel Blob** for any binary assets. Use **anonymous cookie session** (`mm_sid`, HTTP-only, signed). Embed tracks with **deterministic PCA over normalized genre-tag TF-IDF (recipe v1, vocab snapshot committed at build time)**; cluster with **k-means**. Instrument K-factor as **AC8a (point + Wilson 95% CI), AC8b (share-to-create CTR leading indicator), AC8c (Day-30 decision matrix)**. Anonymous-first is a hard architectural invariant (AC9), with a 1-page PIPA-shaped privacy policy.

### Drivers

1. 1 dev × 4 weeks — eliminates anything requiring custom infra, queues, or non-managed DB.
2. K-factor must be **measurable with confidence intervals** on N ≈ 20 — requires server-side event store + persistent share URL + cookie-less-friendly attribution.
3. AC9 anonymous-first — server must accept writes from anonymous sessions, never gate on an auth provider.
4. Embedding must be **deterministic** so AC5 blind self-recognition is reproducible and old maps don't silently change.

### Alternatives considered (steelmanned)

- **Option B — SvelteKit + Cloudflare Workers + D1/KV.** Strongest case: cheapest at scale, edge-distributed share URLs would feel instant globally, KV is a natural fit for serialized maps. Honest steelman: if the team were 2+ devs with prior Workers experience, B might beat A. Disqualified for *this* engagement by: (i) 2–3 day learning-curve tax on D1 + Workers patterns the solo dev hasn't used, (ii) Workers' 50ms CPU limit on free tier creates a real risk for `ml-pca` on N ≥ 100 tracks, (iii) cost advantage is immaterial at < 1k MVP users.
- **Option C — Static React + Supabase (DB + Edge Functions + Storage).** Strongest case: Supabase RLS handles anonymous policies elegantly; one vendor for DB+Storage+Functions; great DX for solo work. Honest steelman: Supabase Auth is opt-in — the claim "auth-first bias" in the v1 draft of this ADR was **incorrect** and has been removed. The real disqualifier: **Supabase Edge Functions run on Deno**, and `umap-js` / `ml-pca` Deno-compat is untested. Debugging an embedding-library/runtime incompatibility in Week 1 would consume the entire budget. Vercel route handlers default to Node — known-good. If a future v2 wanted Supabase, it could swap in once embedding stability is proven.

### Why chosen

Option A matches every decision driver with the lowest research cost. **The disqualifier for B is the CPU cap on UMAP/PCA; for C, it is the Deno-runtime risk on embedding libs — not Supabase Auth.** Neon (vs. Vercel Postgres) is chosen for the data tier to reduce platform lock-in on the *hardest-to-migrate* piece, while staying drop-in compatible with Drizzle and the rest of the architecture.

### Consequences

- **Vendor coupling**: compute, blob, KV (rate-limit), and analytics on Vercel. DB on Neon mitigates the most painful piece of lock-in.
- **Cost**: free tier covers MVP. First cost cliff is Vercel Functions invocations and Neon compute hours — to be monitored daily during soft-launch (R7).
- **Cold start**: first-time visitor first-paint depends on serverless cold start. Mitigated by perf budget R9 and Vercel Speed Insights instrumentation.
- **Embedding is intentionally "boring"**: PCA-TF-IDF will under-represent songs whose genre signal lives only in audio features. AC5 (4/5 blind self-recognition) is the gate that decides whether this is good enough; if AC5 fails, the fix is enrichment (more genre tags), not UMAP.
- **No third-party trackers**: simplifies PIPA posture, but means we trade off some analytics depth (mitigated by first-party event store).

### Data retention (S5)

- `share_events`: retained **90 days** rolling (older rows pruned by nightly Vercel Cron). The K-factor 30-day window plus a 60-day buffer for late-arriving derivatives is sufficient; longer retention is unnecessary and increases PIPA surface.
- `maps` and `tracks`: retained **indefinitely until explicit deletion**, exposed via a `DELETE /api/maps/[id]` endpoint and a per-map "Delete this map" UI affordance (post-MVP polish OK, but the endpoint MUST exist by launch so the privacy policy's deletion clause is truthful).
- `anonymous_sessions`: retained **1 year** rolling.
- Backups follow Neon's default; no separate backup of `share_events` beyond Neon's PITR.

### Follow-ups (post-MVP, prioritized by K-factor result — recalibrated bands matching AC8c)

- If **K < 0.15** (KILL band): write post-mortem; do not migrate stack. If AC5.5 share-intent was simultaneously low, post-mortem documents *shareworthiness* failure; if AC5.5 was high but K low, document *distribution / external-site friction* failure (the (a) hypothesis is wounded but not falsified).
- If **0.15 ≤ K < 0.25**: iterate share CTA copy, OG card design, comparison-view layout. *S6 (OG-image immutability link)* and *S7 (channel pre-commit)* land here.
- If **0.25 ≤ K < 0.4**: scale tests, paid-channel probe, consider Spotify audio features for embedding v2.
- If **K ≥ 0.4**: activate v2 (a)-bet expansion — Adjacent Discovery (CF), node-edge graph view, optional login for cross-device persistence. Re-evaluate Option B/C if Vercel/Neon spend > ~$50/mo.

### Alternative product bets NOT taken (explicit trace)

The §1 product hypothesis declares MusicMapper a bet on **(a) sharable social artifact**. Two adjacent product directions are valid but intentionally *not* this build:

- **(b) Digging engine** — surfaces unknown adjacent artists/tracks. Strong signal in user feedback, but the recommendation surface area is already well-served by Spotify/YouTube/Last.fm; differentiation cost is high relative to K-factor leverage. Folded into MusicMapper as the *consequence* of v2 (a)-success (Adjacent Discovery after K ≥ 0.4), not the MVP bet.
- **(c) State-based automatic curation** — generates daily/weekly playlists based on user state (mood/activity/time). Legitimate alternative product, but **a different product** (consumption surface, not artifact surface). NOT a v2 path for MusicMapper; if (a) fails (K<0.15 + AC5.5 low), revisiting (c) means a fresh kickoff, not pivoting this codebase. Documented here so future-self can trace that (c) was considered and explicitly deferred to a separate product decision.

### Explicitly deferred (S6 / S7)

- **S6 (OG-image immutability link)**: cryptographic content-hash on `app/share/[id]/opengraph-image.tsx` URL. Deferred as v1.1 — not required for K-factor measurement, only for share-card cache-busting hygiene.
- **S7 (channel pre-commit)**: pre-registering launch channels with timestamps for attribution cleanliness. Deferred to launch-day decision (`docs/launch-channels.md` will be written on Week 4 Day 1 regardless).

---

## 8. Open Questions for Architect/Critic

- Should `/share/[id]` use Vercel Edge runtime or default Node runtime? (Edge = faster cold start but no Postgres driver — would need REST gateway.) **Resolved in §7 ADR: Node runtime locked.**
- Is k-means with `k=sqrt(n/2)` good enough for AC5 or should we ship HDBSCAN despite added complexity? **Resolved: k-means for MVP, HDBSCAN deferred to v2.**
- Should we ship Open-Graph share-card image generation in Week 4 or as a fast-follow? **Resolved: ship in Week 4 (highest K-factor multiplier).**
- Confirm soft-launch channel list: 20 users is the floor for AC8 validity — is that achievable from personal network alone? Pre-commit channels in `docs/launch-channels.md` on Week 4 Day 1.

---

## 9. Consensus Status

**Status: `pending approval`**

**RALPLAN-DR Short consensus reached** at iteration 2; **iteration 3 product critique applied by user**:
- Iteration 1: Planner draft → Architect (APPROVE WITH CONDITIONS, 7 surgical revisions) → Critic (REVISE, 5 blockers: B1–B5)
- Iteration 2: Planner revision (all 5 blockers + 6 strong suggestions applied surgically) → Critic verify (APPROVE)
- **Iteration 3 (product hypothesis layer)**: user-reviewer flagged that engineering consensus did not validate the *product* hypothesis. Patches: §1 product hypothesis (path (a) explicit, baseline Spotify-Wrapped, K=0.5 stretch acknowledged), AC5.5 share-intent Likert (diagnoses AC5 PASS + AC8 FAIL ambiguity), `maps.generation_depth` (2세대 K / cascade health), AC8c bands recalibrated to consumer-app benchmarks (KILL <0.15 / iterate 0.15-0.25 / scale 0.25-0.4 / v2 ≥0.4), explicit (b)/(c) alternative-bet trace in §7. Engineering surface unchanged; metric framing and product trace tightened.
- **Iteration 3.1 (second-order patches)**: AC5.5 mean-gate replaced with distribution-aware gate (count ≥3 of 5 score ≥4 AND median ≥4) defending against bimodal noise; cascade gate sample-size floor (gen2 N<10 → informational only) so small-N false strong-downgrade signals don't fire at MVP scale; §1 mechanic-gap honest statement (Wrapped's 3 structural advantages MusicMapper does NOT inherit: pre-aggregation, seasonal coordination, IG Story native format — the bet is therefore that the comparison-view social mechanic substitutes for those losses); KR-specific context (Melon Music Diary < Wrapped share-virality, KR soft-launch harder than global Wrapped benchmark implies).
- **Iteration 3.2 (bet-value gate)**: plan FROZEN. Thin slice gate inserted between current state and plan kickoff. Methodology + pre-decision criteria captured as separate artifacts. plan kickoff conditional on thin slice result mapped through `docs/thin-slice-pre-decision.md` decision matrix; possible outcomes are: strong PASS (kickoff as-is) / soft PASS (양가적 — kickoff with AC threshold conservation per pre-decision §"AC 보수화") / FAIL (no kickoff — route to (c) or freeze). gen2-recipient bias control via Rae-anonymized forwarding script; turnaround latency decomposed (form submission is primary measurement, manual map generation is operational only).

**Footer summary (Critic, iteration 2):**
> Plan approved in RALPLAN-DR Short iteration 2 after all five Critic blockers (AC8 triple reframe, mm_sid attribution rules, Option C invalidation rationale, PIPA privacy posture, deterministic PCA-only embedding) were resolved with substance, not cosmetics. Status moves to `pending approval` — ready for implementation kickoff.

**Companion artifacts:**
- Spec: `.omc/specs/deep-interview-musicmapper.md` (13.5% ambiguity, 8 rounds)
- Architect review: `.omc/drafts/architect-review.md`
- Critic review (iteration 1): `.omc/drafts/critic-review.md`
