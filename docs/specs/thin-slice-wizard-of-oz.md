# Wizard of Oz Thin Slice — Operational Methodology

**Timebox**: 2-3일 (실제 elapsed time 최대 7일 — gen2 회수 대기 포함)
**Goal**: plan §1 자인한 mechanic-gap (Wrapped 3 advantages 없이 비교 뷰 mechanic이 substitute 가능한가)을 4주 commit 전 cheap test.
**Gate**: 결과는 `docs/thin-slice-pre-decision.md` 행렬에 따라 plan kickoff 여부 결정.

---

## 산출물 체크리스트

- [ ] 5-10개 sample 지도 (다양한 장르 mix; 손으로 만든 정적 SVG/이미지 + 단일 페이지)
- [ ] 비교 뷰 정적 페이지 템플릿 (sample 지도 × visitor-placeholder)
- [ ] gen2 derivative-request Google Form (입력: playlist URL or 곡 5개 직접 입력, 이름, 연락 방법)
- [ ] gen1 recruitment 스크립트 (neutral 문구)
- [ ] gen1 forwarding 스크립트 (gen1에게 줄 "친구에게 전달할 때 쓸 문구" 가이드)
- [ ] 결과 기록 시트 (Google Sheet — gen1 → gen2 → form 제출 추적)

## 운영 절차

### Day 0 — 준비 (4-6시간)

1. **Sample 지도 5-10개 제작 (수동)**
   - Notion / Figma / 종이 + 사진 어디든. 출력은 단일 정적 이미지 + 한 페이지 설명.
   - 다양성 확보: 장르 mix (K-indie, hip-hop, jazz, J-pop, classical 등), bubble 클러스터 3-5개, 트랙 N=20-40
   - 각 sample에 가짜 사용자 이름 ("Sample A: 인디감성러", "Sample B: 힙합헤드" 등)

2. **비교 뷰 페이지 템플릿**
   - 정적 HTML 또는 Notion 페이지: 왼쪽 = sample 지도, 오른쪽 = "Make yours" 버튼
   - 버튼 → Google Form 링크 (gen2 derivative-request)
   - URL은 musicmapper-thinslice 등 임시 도메인 또는 GitHub Pages

3. **gen2 Form 설계**
   - 필수: playlist URL (Spotify or YouTube) 또는 곡 5개 입력
   - 필수: 이름 + 연락처 (Rae가 회신할 수 있게)
   - 필수: open-ended "이 비교 페이지를 보고 자기 버전을 만들고 싶었던 이유?"
   - 선택: "어떻게 이 페이지에 왔어요?" (gen1 identification)

4. **Pre-decision 문서 최종 확인**
   - `docs/thin-slice-pre-decision.md` 다시 읽고 서명 (작성일 박힌 그대로 두기, 시작 전 수정 금지)

### Day 1 — gen1 발사

5. **gen1 친구 5명 선정 (선정 기준 박음)**
   - Target user 가까운 demographic (20대-30대 음악 적극 청취자) 우선
   - 너무 친한 친구 (5/5 supportiveness 보장) 회피 — knee-jerk PASS bias 위험
   - 다양한 sub-segment (K-pop heavy / indie 중심 / 영화음악 등)

6. **gen1 메시지 발송**
   ```
   안녕 [이름], 음악 취향 관련 뭔가 만들고 있는데 의견 좀 들려줄래?

   이거 한번 보고 (URL): [sample 비교 페이지 URL]

   부탁: 너 친구 1-2명한테도 보내줘. 단, *내가 만든 거라거나 실험이라는 말은 말고*,
   그냥 "재밌는 음악 도구 발견했어" 정도로만 전달해줘. 자연스럽게 반응이 어떤지 궁금해.

   고마워.
   ```
   - 이 문구 그대로 사용. 변형 시 confirmation bias 측정 contamination 가능.
   - 5명에게 동일 sample 사용 OR 서로 다른 sample 분배 — 후자가 robust (sample 효과 분산)

7. **gen1 전달 추적 시작**
   - 24-48h 내 "보냈어?" 한 번만 reminder
   - gen1이 "어색해서 못 보냈어" 응답 시 — 그 자체를 기록. gen1 전달 friction은 bimodal 신호 (pre-decision 문서 참조)

### Day 2-4 — gen2 회수

8. **Form 제출 추적**
   - Day 4까지 gen2 form 제출 0건이면 reminder 발송 권한 없음 — 첫 사흘 organic 결과가 primary signal
   - Day 4 이후 form 제출 들어오면 → 24h 내 manual 지도 생성 후 derivative 비교 뷰 페이지 회신
   - 회신 후 그 사람이 *또* 다른 친구에게 공유하는지 별도 추적 (gen3 — informational only, decision 영향 없음)

9. **수동 지도 생성 절차 (turnaround 분해)**
   - **Primary measurement는 form 제출 자체** — 이미 끝남
   - 후속 manual 생성은 데이터 품질이 아닌 친구 약속 이행 차원
   - 생성 시간 30분-1시간 (Spotify/YouTube URL 손으로 파싱 → 간단한 클러스터링 → 정적 이미지)

### Day 5-7 — 마감 + 결정

10. **결과 집계 (`docs/thin-slice-results.md`)**
    - gen1 전달 비율 (5명 중 몇 명이 실제 전달 행위 했는가)
    - gen2 form 제출 비율 (전체 gen2 도달자 중)
    - gen2 form 제출자의 open-ended 답변 sentiment 분석 (수동)
    - bimodal 패턴 여부 (강한 PASS + 강한 거부가 동시 출현하는가)

11. **Pre-decision 행렬 적용**
    - `docs/thin-slice-pre-decision.md`의 결정 행렬에 결과 매핑
    - 양가적 케이스 시 30분 self-interview ("왜 더 보수화하지 않는가") 기록 후 plan AC 자동 patch

## 측정 변수 — 명시적 분리

| 변수 | 측정 시점 | Primary / Secondary | 사용처 |
|---|---|---|---|
| gen1 전달 비율 | Day 2-4 | Secondary | bimodal 신호 |
| gen2 도달 수 | Day 3-5 | denominator (form 제출률 계산) | 결정 행렬 |
| **gen2 form 제출 수** | Day 3-7 | **Primary** | 결정 행렬 |
| gen2 form open-ended 답변 | Day 3-7 | Qualitative | 양가적 케이스 진단 |
| gen3 자발 공유 | Day 5+ | Informational only | 결정 영향 없음 (cascade gate sample <10 floor 적용) |
| Manual 지도 생성 latency | Day 4-7 | Operational only | 친구 약속 이행 metric, decision 영향 없음 |

## 정직성 가드 (operational)

- **Sample 지도 quality는 진짜 plan보다 의도적으로 *낮춰* 둠**. 좋게 보이려고 디테일 갈리는 시간 1시간 이상 쓰지 말 것. plan의 인터랙티브 지도가 정적 sample보다 더 매력적이라는 가정이 (a) 가설의 일부 — 정적 sample이 작동하면 인터랙티브는 더 작동할 거라는 lower-bound 추론.
- **Sample 지도에 친구 본인 데이터 절대 쓰지 말 것**. 자기 데이터 보면 self-recognition 발동 → PASS bias.
- **gen1에게 "어땠어?" 후속 질문 금지**. gen1 의견은 decision input 아님. 그들의 *전달 행위*가 신호.

## 종료 트리거

다음 중 하나 발생 시 thin slice 종료, pre-decision 행렬 적용:

- Day 7 도달 (timebox)
- gen2 form 제출 ≥12 (강한 PASS 영역, 추가 데이터 limit utility)
- gen1 5/5 전달 거부 (Day 4까지 0건 전달) — 강한 FAIL의 변형, gen2 hop 자체가 안 일어남
