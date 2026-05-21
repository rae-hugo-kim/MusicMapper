# Thin Slice Artifacts — AI 도구용 프롬프트

**소스 문서**: `docs/specs/thin-slice-artifacts.md`
**대상 도구**: Stitch (Google) / Claude Design (Artifacts) / Figma (First Draft, Magician, Make UI)
**용도**: 디자인 결정을 즉흥적으로 하지 않고 AI 도구에 일관되게 입력해서 thin slice용 artifact를 빠르게 만들기

각 프롬프트는 **그대로 복사-붙여넣기**해서 작동하도록 self-contained로 작성. 한글 카피는 그대로 두되 메타 지시는 영어 — 대부분의 AI 도구 영어 입력 품질이 더 안정적.

---

## 프롬프트 A — Sample Music Map (Claude artifact 권장)

이 프롬프트는 **Claude의 artifact 기능**에 가장 적합 (precise scatter plot + 정확한 색상 + 한글 캡션). Figma 플러그인이나 Stitch에서는 산점도 정확도가 떨어지므로 데이터 시각화는 Claude로 만든 후 SVG export 추천.

### 마스터 프롬프트 (한 번 실행 → 7개 sample SVG 동시 생성)

```
Build a single React + SVG component that renders 7 "music taste map" visualizations side-by-side, one per sample profile below. This is for a Wizard-of-Oz user-testing artifact, so the output must be visually consistent and exportable.

REQUIREMENTS

Each map:
- Canvas: 1200 × 800px SVG (each one)
- Title (h1, 32px, bold, color #111): the sample name
- Subtitle (16px, color #666): a one-line description
- Scatter area: 800 × 600 centered horizontally below the title
- Background: white (#FFFFFF) with a very light grid (gridline #F5F5F5, every 80px)
- NO axis labels, NO numeric ticks
- Each track is a circle, radius 12px, filled with its cluster color, no stroke
- Each cluster has a translucent hull (rounded blob) behind its points: opacity 0.15, same color as the cluster
- Legend at bottom: one row of color-dot + cluster-name + track-count, separated by 24px gap

Cluster color palette (use exactly these 5 colors, assign per cluster):
1. #E63946 (vivid red)
2. #457B9D (teal-blue)
3. #F1A208 (mustard)
4. #2A9D8F (teal-green)
5. #6A4C93 (purple)

If a sample has more than 5 clusters, reuse colors with darker shades (e.g., #B71C2C as a darker red).

Position the circles deterministically: use a seeded layout so the same input produces the same map. Group circles within a cluster into a loose 2D distribution (not a perfect grid). Hulls should wrap each cluster smoothly.

Font stack: Pretendard, Inter, system-ui, sans-serif.

SAMPLE PROFILES (7 maps)

A. "Sample A — 인디감성러"
   Subtitle: "20대 후반 / 평일 늦은 밤 / 한국 인디 + 일본 시티팝 중심"
   Clusters: K-indie (12), 시티팝 (8), 록 (5), 재즈 (2)

B. "Sample B — 힙합헤드"
   Subtitle: "20대 초반 / 출퇴근·운동 청취 / 한국 힙합 + 글로벌 트랩"
   Clusters: K-hip-hop (14), 트랩 (9), R&B (6), 이전세대 힙합 (3)

C. "Sample C — 재즈+클래식 잔잔"
   Subtitle: "30대 / 카페·재택 BGM / 보컬 적은 잔잔한 음악"
   Clusters: 모던재즈 (11), 클래식 솔로 (8), 앰비언트 (5), 보사노바 (4), 시네마틱 (3)

D. "Sample D — K-pop 메인스트림"
   Subtitle: "10대 후반-20대 초반 / 일상 청취 / K-pop 신곡 + 글로벌 팝"
   Clusters: K-pop (18), K-pop ballad (7), 글로벌팝 (6)

E. "Sample E — 장르 잡식"
   Subtitle: "다양한 장르 골고루 / 새 음악 적극 탐색"
   Clusters: 일렉트로닉 (7), 인디록 (6), 힙합 (5), R&B (4), 메탈 (3), 포크 (3), 재즈 (2)

F. "Sample F — 록 + 메탈 진성팬"
   Subtitle: "30대 / 록 페스티벌·콘서트 적극 참여 / 80-90s 클래식 + 현대 록"
   Clusters: 헤비메탈 (9), 90s 얼터너티브 (7), 모던 록 (6), 펑크 (4)

G. "Sample G — 발라드 위주"
   Subtitle: "감정 청취 / 가사 중심 / 한국 발라드 + 글로벌 어쿠스틱"
   Clusters: K-ballad (13), 어쿠스틱 발라드 (7), 인디 포크 (5), OST (3)

DELIVERABLES

1. A single React component that renders all 7 maps stacked vertically with a 48px gap.
2. Each map must be individually exportable as standalone SVG (provide a small "Download SVG" button under each).
3. Code should be clean and self-contained — no external libraries beyond React.

The output is for user testing, not production — visual consistency across the 7 matters more than data accuracy.
```

### 변형 — 한 번에 1개만 생성 (변경/재시도 빠름)

```
Build a React + SVG component that renders ONE "music taste map" visualization for user testing.

Specs:
- 1200 × 800 SVG canvas, white background, light grid (#F5F5F5 every 80px)
- Title at top: "[SAMPLE_NAME]" — 32px bold, color #111
- Subtitle below title: "[SUBTITLE]" — 16px, color #666
- 800×600 scatter area centered below
- Each track: 12px radius circle, filled with cluster color, no stroke
- Each cluster: translucent rounded hull behind its points (opacity 0.15)
- Bottom legend: row of "● [cluster name] [count]" separated by 24px

Cluster colors (assign in order):
#E63946, #457B9D, #F1A208, #2A9D8F, #6A4C93

Font: Pretendard, Inter, system-ui.
NO axis labels or numeric ticks.

CLUSTERS for this sample:
[CLUSTER_NAME_1] ([COUNT_1])
[CLUSTER_NAME_2] ([COUNT_2])
...

Distribute circles deterministically (seeded) so re-running produces the same output.
Add a "Download SVG" button for export.
```

(`[SAMPLE_NAME]`, `[SUBTITLE]`, `[CLUSTER_NAME_N]`, `[COUNT_N]` 부분만 §2 캐릭터 프로필에서 가져와 채우면 됨)

---

## 프롬프트 B — 비교 뷰 페이지 (Stitch / Figma First Draft / Claude artifact)

이 프롬프트는 **세 도구 모두 동일 입력**으로 작동하도록 작성. Stitch는 자동으로 디자인 변형 3-4개를 제안할 것이고, Figma First Draft는 1개의 디자인, Claude artifact는 작동하는 React 컴포넌트를 출력.

### 마스터 프롬프트

```
Design a single landing page for "MusicMapper" — a tool that turns playlists into 2D music taste maps and lets friends share them. The page is the COMPARISON VIEW that a visitor sees after a friend sends them a share URL.

PAGE PURPOSE
Convert the visitor into someone who creates their own map. The visitor just saw a friend's music taste visualized; now we want them to think "what does mine look like?" and click the CTA.

LAYOUT (desktop, 1200px wide)

Top section (header bar, 80px height):
- Left: "MusicMapper" logo as text — 30px, bold
- Right: empty (no nav for thin slice)

Hero section:
- A large header text in Korean: "누군가의 음악 지도를 받았어요?" (Did someone send you their music map?)
  - 32px, bold, color #111
- Below header, small italic caption: "이 사람은 [CAPTION]" (e.g., "이 사람은 인디 + 시티팝 중심이네요")
  - 16px, color #666

Sample image section:
- A placeholder image area, 800 × 533 (16:10 ratio), centered horizontally
- Placeholder reads "[Sample Music Map Image Here]" — this will be replaced with one of 7 pre-generated PNGs in production
- Subtle 1px border, border-radius 12px

CTA section (the most important section — must be above the fold on iPhone SE 2020 / 375 × 667):
- An h2 above the button: "💭 당신의 음악 지도는 어떻게 생겼을까요?"
  - 24px, bold, color #111
- A primary button: "내 지도 만들기 (30초)"
  - Button: 280px wide, 56px tall, filled background #2A9D8F (teal-green), white text 18px bold, border-radius 12px
  - Hover: slightly darker (#1E7E72)
- Subtext below button (small, gray):
  - "플레이리스트 URL 또는 곡 5개만 알려주세요"
  - 14px, color #888

Footer:
- "Made by MusicMapper · 개인정보 · 연락" — small text 12px, color #999, centered, 32px from bottom

DESIGN GUIDELINES
- White background (#FFFFFF) throughout
- Plenty of vertical breathing room (sections separated by 48-64px)
- Single column layout, content max-width 800px and centered
- Font stack: Pretendard, Inter, system-ui, sans-serif
- No images, illustrations, or icons besides the sample image placeholder
- Avoid corporate/SaaS feel — should feel personal and curious, not promotional

MOBILE BREAKPOINT (≤768px)
- Same vertical order, full-width with 16px side padding
- Sample image: full width (100% - 32px), maintain aspect ratio
- CTA button: full width (100% - 32px), 56px tall (thumb-friendly)
- CRITICAL: on iPhone SE 2020 (375 × 667), the sample image + h2 + CTA button must all fit ABOVE THE FOLD without scrolling

TONE
The visitor is curious, not committed. The page should answer their unspoken question "what is this?" within 3 seconds, and offer them a low-cost way to try it. The "(30초)" suffix on the CTA is doing critical work — it preempts the "this'll take too long" objection.

DELIVERABLE
A single page in the chosen tool's native format (Stitch design / Figma frame / Claude artifact React component). For Claude artifact, make it a functional React + Tailwind component with a hardcoded sample image placeholder.
```

### 변형 — 7개 sample 캡션 변주만

페이지 디자인은 1개로 통일하되 sample별 caption만 다르게. 다음 7개 caption을 페이지 hero section의 "이 사람은 ___" 자리에 차례로 갈아 끼움:

| Sample | Caption |
|---|---|
| A | 인디 + 시티팝 중심이네요 |
| B | 한국 힙합 + 트랩 중심이네요 |
| C | 잔잔한 재즈와 클래식을 좋아하네요 |
| D | K-pop을 광범위하게 즐기네요 |
| E | 장르를 가리지 않고 다양하게 듣네요 |
| F | 록과 메탈에 진심이네요 |
| G | 발라드를 중심으로 감성 청취하네요 |

---

## 프롬프트 C — Google Form은 AI 도구 불필요

§4 정확 wording을 그대로 Google Form에 손으로 입력. AI 도구로 자동화할 가치 < 손으로 5분 작업.

---

## 도구별 사용 팁

### Stitch (Google)
- 마스터 프롬프트 B 그대로 입력 → 자동으로 3-4개 디자인 변형 제안
- 가장 마음에 드는 것 선택 → Figma export 가능
- Sample image는 placeholder로 유지, 실제 사용 시 프롬프트 A의 출력 PNG로 교체

### Figma First Draft (Make Designs)
- 마스터 프롬프트 B 입력 → 1개 디자인 출력
- "Refine" 기능으로 색상 / 폰트 미세 조정
- Sample image placeholder는 frame으로 처리해 나중 일괄 교체

### Claude Design (artifacts)
- 프롬프트 A 마스터로 7개 sample SVG 일괄 생성 → "Download SVG" 버튼으로 export
- 프롬프트 B로 React + Tailwind 컴포넌트 생성 → 정적 export 또는 GitHub Pages 직접 배포
- **추천 순서**: Claude로 A 먼저 (data viz 정확도) → Stitch나 Figma로 B (UI 디자인 완성도)

### Magician / Figma AI Plugins
- 프롬프트 B 입력, "단일 화면 디자인" 모드
- 결과물 그대로 사용 가능 / Refine 필요 시 "make CTA more prominent" 같은 작은 지시로 조정

---

## Day 0 통합 워크플로 (이 프롬프트들 활용 시 1-2시간 단축)

1. **Claude artifact**에 프롬프트 A 마스터 입력 → 7개 sample SVG 생성 + PNG로 export. **30-45분**.
2. **Stitch 또는 Figma First Draft**에 프롬프트 B 마스터 입력 → 페이지 1개 생성. **15-20분** (변형 검토 + 선택).
3. 선택한 디자인을 Figma에서 sample별로 duplicate (7 페이지) → caption만 §B 표 따라 갈아 끼움. **15분**.
4. Figma export → 정적 HTML 또는 그대로 PNG로 GitHub Pages 호스팅. **20-30분**.
5. Google Form §C 따라 손 작업. **10분**.
6. **총 1.5-2.5시간** (마음에 드는 디자인 한 번에 나오면 더 빨리).

원래 §thin-slice-artifacts.md의 "4-6시간" 추정의 디자인 부분이 AI 도구로 절반 이하로 압축됨. 실제 작업 비용보다 *결정 즉흥성 제거*가 더 큰 이득.
