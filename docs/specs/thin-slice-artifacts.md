# Thin Slice Artifacts — Design / Concept / Content Spec

**컴패니언 문서**: `docs/specs/thin-slice-wizard-of-oz.md` (프로토콜) + `docs/thin-slice-pre-decision.md` (결정 기준)
**목적**: 즉흥 디자인이 결과 해석을 오염시키지 않도록 thin slice가 사용할 *모든* artifact의 시각·언어 결정을 사전에 박음.
**작성일**: 2026-05-21

---

## 1. Sample 지도 visual spec

### Layout (단일 정적 이미지, 가로 1200 × 세로 800px)

```
┌──────────────────────────────────────────────────┐
│  [Sample A: 인디감성러]                          │  ← 제목 영역 (h1, 32px, 좌측 정렬)
│  20대 후반 / 평일 늦은 밤 청취 / 한국 인디 + 일본 시티팝 중심  │  ← 한 줄 description (16px, 회색)
│                                                  │
│   ●         ●●                                   │
│        ●  ●●●  ●                                 │  ← 산점도 영역 (800 × 600)
│          ●●●●●●                                  │
│      ●●  ●●●●● ●●                                │
│          ●●●●   ●                                │
│              ●●  ●●●●                            │
│                    ●●●                           │
│                                                  │
│  [● K-indie 12]  [● 시티팝 8]  [● 록 5]  [● 재즈 2] │  ← 클러스터 legend (하단)
└──────────────────────────────────────────────────┘
```

### 시각 단서 결정 (박음)

| 요소 | 결정 | 이유 |
|---|---|---|
| 점 크기 | 일률 (12px 반지름) | 청취 빈도 차이 표현 불가 → "내 데이터" 시각화 환상 약화. 그래도 thin slice scope OK; 진짜 plan에선 크기 = 청취 빈도 |
| 점 색상 | 클러스터별 (4-5색) | 장르 그룹 가시화가 비교 뷰의 핵심 — "내가 한쪽으로 쏠려 있네" 자기 인식 단서 |
| 색 팔레트 | `#E63946 (선명한 빨강) / #457B9D (청록) / #F1A208 (머스타드) / #2A9D8F (틸) / #6A4C93 (보라)` | 색맹 친화 + 서로 충분히 구분 + 인스타 thumbnail에서도 살아남는 saturation |
| 클러스터 시각화 | **반투명 hull** (점들 둘러싼 둥근 영역, opacity 0.15) | 그룹 가시화 강화. 단, hull은 *영역만*, 라벨은 hull 안이 아닌 하단 legend |
| 트랙명 표시 | **표시 안 함 (hover 없음, 정적 이미지)** | thin slice에선 호버 인터랙션 없음. 트랙명은 비교 뷰 페이지에서 "총 27곡 중 표시: 인디 12 / 시티팝 8 / ..." 식 요약만 |
| 배경 | 흰 배경 + 약한 그리드 (`#F5F5F5`) | 깔끔, 인쇄/스크린 모두 호환 |
| 폰트 | 시스템 sans (Pretendard 또는 Inter) | 한글 자연스러움 우선 |
| 좌표축 | **없음** (수치 축 숨김) | "PCA dim 1/2" 표시는 의미 없음 + 사용자에게 인지부담 |

### 만들기 도구

**Figma** 권장 (mockup 5-10개 같은 frame에서 변주 빠름). 대안: 손그림 + 폰 카메라 (실험으로 OK, 단 일관성 유지 — 5장 모두 같은 도구).

---

## 2. 5-8개 Sample 캐릭터 프로필

각 sample은 (a) 캐릭터 이름, (b) 한 줄 description, (c) 클러스터 분포 (장르 + 곡 수), (d) 대표 트랙 5곡으로 구성. 손으로 만들 때 (c)와 (d)가 가이드.

### Sample A — "인디감성러"

- **Description**: 20대 후반 / 평일 늦은 밤 청취 / 한국 인디 + 일본 시티팝 중심
- **클러스터 분포**: K-indie 12 / 시티팝 8 / 록 5 / 재즈 2 (총 27곡)
- **대표 트랙 5곡**:
  - 새소년 — 파도
  - 잔나비 — 주저하는 연인들을 위해
  - 山下達郎 — Sparkle
  - The 1975 — About You
  - 검정치마 — Antifreeze

### Sample B — "힙합헤드"

- **Description**: 20대 초반 / 출퇴근/운동 청취 / 한국 힙합 + 글로벌 트랩 중심
- **클러스터 분포**: K-hip-hop 14 / 트랩 9 / R&B 6 / 이전세대 힙합 3 (총 32곡)
- **대표 트랙 5곡**:
  - 빈지노 — Aqua Man
  - Travis Scott — FE!N
  - The Weeknd — Blinding Lights
  - 비프리 — 사이렌
  - Kendrick Lamar — Not Like Us

### Sample C — "재즈+클래식 잔잔"

- **Description**: 30대 / 카페·재택근무 BGM / 보컬 적은 잔잔한 음악
- **클러스터 분포**: 모던재즈 11 / 클래식 솔로 8 / 앰비언트 5 / 보사노바 4 / 시네마틱 3 (총 31곡)
- **대표 트랙 5곡**:
  - Bill Evans Trio — Peace Piece
  - Max Richter — On the Nature of Daylight
  - Antônio Carlos Jobim — Wave
  - Nils Frahm — Says
  - Debussy — Clair de Lune

### Sample D — "K-pop 메인스트림"

- **Description**: 10대 후반-20대 초반 / 일상 청취 / K-pop 신곡 중심 + 글로벌 팝
- **클러스터 분포**: K-pop 18 / K-pop ballad 7 / 글로벌팝 6 (총 31곡)
- **대표 트랙 5곡**:
  - NewJeans — Super Shy
  - aespa — Supernova
  - IVE — Baddie
  - Olivia Rodrigo — vampire
  - Sabrina Carpenter — Espresso

### Sample E — "장르 잡식"

- **Description**: 다양한 장르 골고루 / 새 음악 적극 탐색 / 평균 청취 다양성 높음
- **클러스터 분포**: 일렉트로닉 7 / 인디록 6 / 힙합 5 / R&B 4 / 메탈 3 / 포크 3 / 재즈 2 (총 30곡)
- **대표 트랙 5곡**:
  - Fred again.. — Marea (we've lost dancing)
  - Black Midi — Sugar/Tzu
  - Tyler, The Creator — DOGTOOTH
  - SZA — Snooze
  - Nine Inch Nails — Hurt

### Sample F — "록 + 메탈 진성팬"

- **Description**: 30대 / 록 페스티벌·콘서트 적극 참여 / 80-90s 클래식 + 현대 록
- **클러스터 분포**: 헤비메탈 9 / 90s 얼터너티브 7 / 모던 록 6 / 펑크 4 (총 26곡)
- **대표 트랙 5곡**:
  - System of a Down — Toxicity
  - Pearl Jam — Even Flow
  - Idles — Mother
  - 잠뱅이 — Last (가상의 한국 록 밴드 — 진짜 thin slice 시 실제 곡으로 교체)
  - Tool — Schism

### Sample G — "발라드 위주"

- **Description**: 감정 청취 / 가사 중심 / 한국 발라드 + 글로벌 어쿠스틱
- **클러스터 분포**: K-ballad 13 / 어쿠스틱 발라드 7 / 인디 포크 5 / OST 3 (총 28곡)
- **대표 트랙 5곡**:
  - 폴킴 — 모든 날, 모든 순간
  - 아이유 — 이런 엔딩
  - Adele — Easy On Me
  - 죠지 — boomerang
  - Joshua Bassett — Lie Lie Lie

### Sample 활용 규칙 (gen1 분배)

- 5개 gen1 친구에게 **서로 다른 sample** 배포 (sample 효과 분산). N=5이면 A, B, C, D, E 분배.
- gen1 친구 demographic이 sample profile에서 *멀게* 배치 (예: K-pop 안 듣는 친구에게 Sample D). 이유: thin slice는 "*자기 데이터처럼* 느껴지는가"가 아니라 "*공유할 가치가 있게* 느껴지는가" 검증.
- 한 sample을 두 명에게 배포하면 sample 효과 분리 불가 → 절대 회피.

---

## 3. 비교 뷰 페이지 — wireframe + copy

### 페이지 구조 (단일 페이지, scroll OK)

```
┌──────────────────────────────────────────────────┐
│  MusicMapper  (logo, h1 30px, 좌측)              │
│  ── 누군가의 음악 지도를 받았어요?              │  ← 헤더 카피
├──────────────────────────────────────────────────┤
│                                                  │
│      [Sample A 이미지 — section 1 결과]          │  ← sample 지도, 좌측 또는 상단
│      "이 사람은 인디 + 시티팝 중심이네요"        │
│                                                  │
├──────────────────────────────────────────────────┤
│                                                  │
│      💭 당신의 음악 지도는 어떻게 생겼을까요?   │  ← 핵심 카피, h2 24px
│                                                  │
│      [ 내 지도 만들기 (30초) ]   ← CTA 버튼     │
│                                                  │
│      플레이리스트 URL 또는 곡 5개만 알려주세요   │  ← subtext, 14px
│                                                  │
├──────────────────────────────────────────────────┤
│  Made by MusicMapper · privacy · contact         │  ← footer
└──────────────────────────────────────────────────┘
```

### Copy 결정 (박음)

| 위치 | 한글 (KR 소프트런치 기본) | 영어 (필요 시 fallback) |
|---|---|---|
| 헤더 | 누군가의 음악 지도를 받았어요? | Received someone's music map? |
| Sample 캡션 | "이 사람은 인디 + 시티팝 중심이네요" (sample별 변주) | "This person leans indie + city pop" |
| 핵심 카피 (h2) | 💭 당신의 음악 지도는 어떻게 생겼을까요? | 💭 What does your music map look like? |
| CTA 버튼 | **내 지도 만들기 (30초)** | **Make your own (30 sec)** |
| Subtext | 플레이리스트 URL 또는 곡 5개만 알려주세요 | Just paste a playlist URL or list 5 songs |
| Footer | Made by MusicMapper · 개인정보 · 연락 | Made by MusicMapper · privacy · contact |

### 디자인 결정

- **CTA 버튼은 페이지 fold 안 (스크롤 없이 보임)**. 모바일 fold는 더 짧으니 sample 이미지를 65% 높이로 제한.
- **Sample 캡션 1문장으로 짧게** — 길어지면 분석적 거리감 발생 → "내 차례"의 emotional pull 약화.
- **CTA에 "(30초)" 명시** — Wrapped와 달리 input cost 0이 아님을 알면서 *낮은 비용*임을 강조. 친구들 피드백 "외부 사이트 접속 자체가 진입장벽" 정면으로 다룸.
- **"플레이리스트 URL 또는 곡 5개" 명시** — 30초 약속의 신뢰성을 입력 옵션의 가벼움으로 뒷받침. 5개라는 작은 숫자는 "이 정도면 할 만하다"의 mental commitment 단서.

### 모바일 분기

- 가로 폭 768px 이하: section 1과 section 2 세로 스택. Sample 이미지 화면 너비 100%, 좌우 16px padding.
- CTA 버튼: 화면 너비 - 32px, 높이 56px (thumb-friendly).
- iPhone SE 2020 (375 × 667)에서 sample 이미지 + h2 + CTA 모두 fold 안에 들어와야 함 — Day 0 검증 항목.

### 호스팅 (정적 페이지)

- **GitHub Pages** 또는 **Vercel** (계정 있으면 5분).
- URL 형식: `https://musicmapper-thinslice.vercel.app/sample-a` (sample별 1 페이지 = 5-8개 URL).
- 진짜 도메인 (`musicmapper.app`) 사용 금지 — thin slice는 실험 단계이고 도메인 인지가 결과를 contaminate 가능.

---

## 4. gen2 Form — Google Form 정확 wording

### Form 구조 (3 페이지)

#### Page 1: 환영 + 입력 옵션

**제목**: 당신의 음악 지도 만들기

**Description**:
> 좋아하는 음악으로 당신의 "음악 지도"를 만들어드려요.
> 플레이리스트 URL을 붙여넣거나, 좋아하는 곡 5개를 알려주시면 됩니다.
> 만든 지도는 3일 안에 이 이메일로 보내드릴게요.

**질문 1 (필수)**: 어떤 방식으로 알려주시겠어요?
- [○] Spotify 플레이리스트 URL
- [○] YouTube 플레이리스트 URL
- [○] 좋아하는 곡 5개를 직접 입력
- [○] Apple Music 플레이리스트 (현재 미지원 — 곡 직접 입력으로 도와드릴 수 있어요)

**질문 2 (조건부)**:
- (Spotify/YouTube 선택 시) URL을 붙여넣어 주세요: [text input]
- (직접 입력 선택 시) 좋아하는 곡 5곡을 알려주세요 (제목 - 아티스트 형식): [paragraph text, 5줄 권장]

#### Page 2: 연락 정보

**질문 3 (필수)**: 만든 지도를 어디로 받으실 건가요?
- 이메일: [text input, email validation]

**질문 4 (선택)**: 이름이나 닉네임 (지도에 적힐 이름)
- [text input]

#### Page 3: 동기 (핵심 — primary qualitative signal)

**질문 5 (필수, open-ended)**:
> 이 페이지를 보고 "내 지도를 만들고 싶다"고 생각한 이유가 있나요?
> (한 문장이라도 좋아요)

- [paragraph text]

**질문 6 (선택)**: 이 페이지를 어떻게 알게 되셨어요?
- [○] 친구가 보내줌
- [○] SNS / 메신저에서 발견
- [○] 직접 검색
- [○] 기타: [text input]

**질문 7 (선택, AC5.5 proxy)**: 만약 자기 지도가 마음에 든다면, 친구한테 공유하고 싶을까요?
- [1 - 절대 안 함] [2] [3] [4] [5 - 무조건 공유]

**제출 후 안내 페이지**:
> 감사합니다! 3일 안에 이메일로 지도를 보내드릴게요.
> 자기 지도가 마음에 들면, 친구에게도 한 번 보내주세요. 🎵
> (이 페이지 URL: `[페이지 URL]`)

### Form 동작 결정

- **Google Form** 사용 (제일 빠름, response sheet 자동 생성).
- **이메일 수집**: 필수. 회신 채널 + 중복 방지.
- **응답 저장**: Google Sheet에 자동 sync → `docs/thin-slice-results.md`로 수동 export.
- **회신 약속**: "3일 안에" — 약속 위반 시 측정 신뢰도 무관(primary는 form 제출), 단 친구 약속 차원에서 지킬 것.
- **Form 자체에서 attribution 측정 시도 안 함** — gen1 식별은 sample URL × 이메일 도메인 정도로만 추정. 결정행렬 input에는 미사용.

---

## Day 0 작업 체크리스트 (이 문서 기반)

- [ ] **Figma 또는 종이로 Sample 5-8개 mockup 제작** — §1 visual spec + §2 캐릭터 프로필 따라
- [ ] **각 sample을 png/jpg로 export** — 1200×800, 파일명 `sample-a.png` 등
- [ ] **정적 HTML 페이지 5-8개 작성** — §3 wireframe + copy 그대로 (sample별 변주는 caption 한 문장만)
- [ ] **GitHub Pages 또는 Vercel 배포** — sample별 1 URL
- [ ] **Google Form 생성** — §4 정확 wording 그대로, 3 페이지
- [ ] **Form 응답 → Google Sheet 연결 확인**
- [ ] **gen1 5명 후보 명단 + sample 분배 매핑** (예: 친구 A → Sample C, 친구 B → Sample A...)
- [ ] **gen1 메시지 + gen2 forwarding 스크립트 final 확인** (`docs/specs/thin-slice-wizard-of-oz.md` §6, §7 참조)
- [ ] **`docs/thin-slice-pre-decision.md` 재확인 + 시작 시점 박기**

총 예상 시간: **4-6시간** (Sample mockup 2-3h + 페이지 작성·배포 1-1.5h + Form 0.5h + gen1 명단 0.5h)
