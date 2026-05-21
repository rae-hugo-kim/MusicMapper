# Deep Interview Spec: MusicMapper

## Metadata
- Interview ID: musicmapper-2026-05-20
- Rounds: 8 (Round 0 토폴로지 + 8 Q&A)
- Final Ambiguity Score: 13.5%
- Type: greenfield
- Generated: 2026-05-20
- Threshold: 20%
- Initial Context Summarized: no
- Status: PASSED

## Clarity Breakdown

| Dimension | Score | Weight | Weighted |
|-----------|-------|--------|----------|
| Goal Clarity | 0.85 | 0.40 | 0.340 |
| Constraint Clarity | 0.85 | 0.30 | 0.255 |
| Success Criteria | 0.90 | 0.30 | 0.270 |
| **Total Clarity** | | | **0.865** |
| **Ambiguity** | | | **0.135** |

## Topology

| Component | Status | Description | Coverage / Deferral Note |
|-----------|--------|-------------|--------------------------|
| Playlist Ingest | active | YouTube/Spotify 원클릭 import + 수동 곡 입력, 강제 로그인 없음 | AC1, AC2, AC3 충족 |
| Visualization | active | 2D 산점도 + 버블 클러스터 (장르/취향 축), 노드-엣지 모드는 v2 | AC4, AC5 충족 |
| Share | active | "내 지도 vs 공유자 지도" 비교 뷰 URL, 방문자 로그인 불필요 | AC6, AC7, AC8 충족 |
| Adjacent Discovery | deferred | 협업 필터링 기반 인접 추천. K≥0.5 달성 후 v2에서 활성화 | 사용자 결정: cold-start를 Share/초대로 우회. CF는 데이터 임계 도달 후 활성화 |

## Goal

사용자가 자신의 플레이리스트/청취 데이터로 "내 음악 지도"(2D 산점도 + 버블 클러스터)를 만들고, 친구와 비교 뷰로 공유해 자연 전파를 통해 성장하는 도구. v2에서는 누적된 지도 간 유사도를 활용해 사용자가 아직 모르는 인접 아티스트/곡을 추천(Adjacent Discovery)하지만, MVP는 **지도 생성·공유 루프 자체**가 산출물.

## Constraints

- **강제 로그인 금지** — 초기 이탈 방지. 곡 수동 입력, YouTube 플레이리스트 import, Spotify(공개 API 한해) 모두 익명으로 가능해야 함
- **MVP 4주 / 1인 개발** — 솔로 프로젝트 가정, 스코프는 이 시간 안에 끝나야 함
- **공유 URL은 익명 열람 가능** — 방문자 로그인·가입 강제 시 K-factor 루프 깨짐
- **MVP는 비교 뷰의 진입 마찰 최소화** — 방문자가 "나도 만들어보고 싶다" 직후 1-2 클릭으로 생성 가능해야 K≥0.5 가능
- 측정 윈도우: 공유 후 **30일** 내의 파생 지도 수가 K-factor 분자

## Non-Goals

- **Adjacent Discovery / 추천 알고리즘** — v2로 명시 deferred (CF가 작동하려면 사용자/데이터 임계량 필요, MVP에서 검증할 가설이 아님)
- **노드-엣지 그래프 뷰** — MVP는 산점도+클러스터만, 노드-엣지는 v2
- **다중 스트리밍 통합** (Apple Music, Tidal 등) — MVP는 YouTube + Spotify + 수동만
- **회원 시스템 / 프로필 / 팔로우** — MVP는 익명 + URL 기반 공유로만 동작
- **모바일 네이티브 앱** — 웹 우선 (반응형 OK)

## Acceptance Criteria

- [ ] **AC1** Ingest: YouTube 플레이리스트 URL을 붙여넣으면 곡 목록이 추출되어 지도 생성 입력으로 사용된다 (로그인 없이)
- [ ] **AC2** Ingest: Spotify 공개 플레이리스트 URL/ID로 동일 동작 (로그인 없이 공개 API 한정)
- [ ] **AC3** Ingest: 곡 제목/아티스트를 수동 입력해 외부 메타데이터(MusicBrainz/Last.fm 등) 보강 후 지도에 반영
- [ ] **AC4** Visualization: 입력된 곡들이 2D 산점도로 시각화되고, 인접한 곡들이 버블 클러스터로 묶여 보인다
- [ ] **AC5** Visualization: 사용자가 "오, 이게 내 음악이네"라고 느낄 수 있는 자기 인식 수준의 매핑 (정성 검증, 사용자 테스트 5인 이상 4/5 동의)
- [ ] **AC6** Share: 생성된 지도가 익명 열람 가능한 고유 URL로 공유된다
- [ ] **AC7** Share: 공유 URL을 방문한 사람이 "내 지도 vs 공유자 지도" 비교 뷰를 본다 (방문자도 로그인 없이 자신의 지도 생성 시작 가능)
- [ ] **AC8** Success Metric: 베타 30일 운영 후 K-factor ≥ 0.5 (공유된 지도 1개당 평균 0.5개 이상의 파생 지도가 30일 윈도우 내에 생성)
- [ ] **AC9** Constraint compliance: 어느 흐름에서도 강제 회원가입/로그인이 없다 (감사 가능: 모든 핵심 액션이 익명 세션에서 완결)

## Assumptions Exposed & Resolved

| Assumption | Challenge | Resolution |
|------------|-----------|------------|
| "데이터가 쌓이면 CF로 추천" | Round 4 Contrarian: CF 임계량 도달까지 초기 유저는 쓸모 없을 수도 있다 | "지도 공유" 자체가 cold-start 해결 → 초대 기반 성장. Discovery는 v2 deferred |
| MVP에 Discovery 포함 필요 | Round 6 Simplifier: K-factor가 1차 지표면 MVP는 그것만 측정 가능한 최소 단위여야 | Discovery 제외, MVP = Ingest + Viz(2D 산점도) + Share. Discovery는 K≥0.5 후 v2 |
| 다양한 시각화 형태 모두 MVP에 | Round 6 Simplifier: 노드-엣지는 정보 밀도 높아 입문자 부담 | MVP는 산점도+클러스터만, 노드-엣지는 v2 |
| 공유 URL은 단순 리드 페이지 | Round 7 비교 뷰 선택 | "내 지도 vs 공유자 지도" 비교 뷰 — visitor curiosity가 가장 강력한 viral driver |
| "강제 로그인 OK" | Round 2 사용자 명시 거부: "강제 로그인은 초기 이탈" | 모든 핵심 액션 익명 가능, OAuth는 선택지로만 |

## Technical Context

Greenfield 웹 프로젝트, 1인 4주 MVP. 핵심 외부 의존:
- **YouTube Data API** — 공개 플레이리스트 곡 목록 추출 (API 키 필요, OAuth 불필요)
- **Spotify Web API** — 공개 플레이리스트 read (Client Credentials flow, 사용자 OAuth 불필요)
- **MusicBrainz / Last.fm** — 수동 입력 곡의 메타데이터 보강 (장르, 태그, 유사 아티스트)
- **2D 임베딩 기법** — UMAP/t-SNE/PCA를 곡 메타데이터(장르 벡터, 오디오 특성 if available) 위에 적용
- **공유 URL 저장소** — 지도 직렬화 형태(JSON or compressed string), 익명 영구 링크

기술 스택은 implementation 단계에서 결정 (Next.js+Vercel, SvelteKit 등 4주 1인에 적합한 풀스택 후보).

## Ontology (Key Entities)

| Entity | Type | Fields | Relationships |
|--------|------|--------|---------------|
| User (anonymous) | core domain | session_id, created_maps[] | 1:N Map |
| Map | core domain | id, owner_session, tracks[], embedding_2d[], cluster_assignments[], share_url, created_at | N:1 User, 1:N Track |
| Track | core domain | title, artist, source (youtube/spotify/manual), metadata (genre, tags, audio_features?) | M:N Map |
| Share Event | core domain | map_id, visitor_session, viewed_at, derivative_map_id? | N:1 Map (origin), 0:1 Map (derivative) |
| K-factor | metric | window_days=30, numerator=derivative_maps_count, denominator=source_maps_count | computed from Share Events |

## Ontology Convergence

| Round | Entity Count | New | Changed | Stable | Stability Ratio |
|-------|-------------|-----|---------|--------|----------------|
| 1 | 3 (User, Playlist, Genre) | 3 | - | - | N/A |
| 2 | 4 (+ Source) | 1 | 1 (Playlist→TrackInput) | 2 | 75% |
| 3 | 5 (+ Map/Cluster) | 2 | - | 3 | 60% |
| 4 | 6 (+ Adjacency/CF) | 1 | - | 5 | 83% |
| 5 | 7 (+ ShareEvent) | 1 | - | 6 | 86% |
| 6 | 6 (- Adjacency: deferred) | 0 | 1 (CF→K-factor) | 5 | 83% |
| 7 | 6 | 0 | 0 | 6 | 100% |
| 8 | 6 | 0 | 0 | 6 | 100% |

엔티티가 Round 7-8에 완전 수렴 — 도메인 모델 안정.

## Interview Transcript

<details>
<summary>Full Q&A (8 rounds)</summary>

### Round 0 (Topology Gate)
**Q:** MusicMapper 최상위 컴포넌트 3개(Ingest/Visualization/Discovery) 맞나?
**A:** "맞아요, 3개 다 살려주세요"
**Outcome:** 3 components locked active

### Round 1
**Q:** MusicMapper의 "매핑"은 무엇과 무엇을 연결하는 도구?
**A:** "개인의 플레이리스트를 토대로 장르 및 취향을 시각화하고, 미처 몰랐지만 좋아할 것 같은 인접 아티스트, 곡을 디깅하는 데 도움을 주는 툴"
**Ambiguity:** 79% (Goal: 0.45, Constraints: 0.05, Criteria: 0.05)

### Round 2
**Q:** Playlist는 어디서 오는가?
**A:** "선택권을 줘야해 — 곡 입력도 괜찮고, 유튜브 플레이리스트도 읽어오면 좋고, 스포티파이 api가 공개되어있다면 그걸 이용해도 좋고. 강제로 '로그인 꼭 해야함!' 같은걸 넣으면 유저들이 초기에 이탈함"
**Ambiguity:** 72% — **핵심 제약 잠금: 강제 로그인 금지**

### Round 3
**Q:** "내 음악 지도"는 화면에서 어떤 모양?
**A:** "2d 산점도와 노드-엣지 그래프를 둘 다 포함하는 형식이었으면. 기본적으로 산점도에 버블클러스터로 시작해서, 사용자 데이터가 축적되면 비슷한 관계도를 그린 사람의 '이런 음악도 좋아하더라' 식으로 확장해서 추천알고리즘으로 작동"
**Ambiguity:** 69% — Visualization 2단계 진화 모델 잠금

### Round 4 (Contrarian Mode)
**Q:** CF가 작동할 때까지 콜드스타트 구간은 어떻게?
**A:** "지도 공유 자체가 cold-start 해결—초대 기반 성장"
**Ambiguity:** 63% — viral loop 가설 확정

### Round 5
**Q:** 성공의 검수 가능한 합격 조건 1순위?
**A:** "공유 전파율 (K-factor)"
**Ambiguity:** 43% — K-factor 확정 → Share 컴포넌트 1급 격상

### Round 6 (Simplifier Mode)
**Q:** MVP 스코프 + 목표 K값?
**A:** "+ YouTube/Spotify 원클릭 import 추가 — K≥0.5 목표"
**Ambiguity:** 28% — Discovery deferred / Share 추가 / 노드-엣지 v2

### Round 7
**Q:** 공유 URL을 친구가 열었을 때 무엇을?
**A:** "나의 지도 vs 공유자 지도 비교 뷰 (방문자도 자신의 지도 생성 유도)"
**Ambiguity:** 23%

### Round 8
**Q:** MVP 기간 + K 측정 윈도우?
**A:** "1인 · 4주 MVP · K는 30일 윈도우"
**Ambiguity:** **13.5%** ✅ 임계값 통과

</details>

## Pipeline Status

- [x] Stage 1 — Deep Interview (모호도 ≤ 20% 통과)
- [ ] Stage 2 — omc-plan 합의 정제 (선택적)
- [ ] Stage 3 — 실행 (별도 승인 필요)

**Status: pending approval**
