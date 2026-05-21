# Wizard of Oz Thin Slice — Pre-Decision Criteria

**작성 시점**: thin slice **시작 전** (post-hoc rationalization 방어 목적)
**작성자**: Rae
**작성일**: 2026-05-21
**서명 효력**: 이 문서는 thin slice 결과를 보기 전 작성되었다. 결과 본 이후 thresholds 조정은 *명시적으로 새 row 추가*로만 허용 (기존 row 덮어쓰기 금지).

---

## 측정 단위

- **gen1** = Rae가 직접 share 링크를 보낸 친구 5명. K-factor measurement에는 포함되지 않음 (attribution chain source일 뿐).
- **gen2** = gen1이 자기 1-2명에게 *Rae 언급 없이 neutral 문구로* 전달한 friends-of-friends. **이게 primary measurement 모집단**.
- **gen2 target N**: 5 (gen1 × 1) ~ 15 (gen1 × 3, 자연 상한)
- **Primary metric**: gen2 중 "derivative 지도 요청 form 제출" 비율
- **Secondary**: gen2 form 제출자의 open-ended 답변 ("왜 만들고 싶었나요?")

## 결정 행렬 (사전 박음)

| gen2 derivative request | 분류 | 행동 |
|---|---|---|
| **0/15 (0%)** | 강한 FAIL | plan freeze 유지. post-mortem 작성. (c) 라우팅 또는 보존 결정. plan kickoff 금지. |
| **1-2/15 (7-13%)** | 약한 FAIL | 동일. 1-2건은 noise / friendship support 가능성 압도. plan kickoff 금지. |
| **3-7/15 (20-47%)** | 양가적 (가장 흔할 결과) | plan kickoff하되 **AC 임계 보수화**: K 타깃 0.4 → 0.25, AC8b CTR 15% → 10%. AC8c KILL band 0.15 → 0.20으로 상향. open-ended 답변에서 "왜 그만큼만 작동했는가" 분석 → share CTA/OG 우선순위 상향 후 kickoff. |
| **8-11/15 (53-73%)** | 강한 PASS | plan kickoff. AC 임계 그대로. thin slice 결과를 `docs/thin-slice-results.md`에 첨부. |
| **12+/15 (80%+)** | 매우 강한 PASS | plan kickoff + AC 임계 상향 검토 (K 타깃 0.4 → 0.5). 이 결과는 의심해야 함 (selection bias?) — gen1 친구 demographic이 target user에 너무 잘 맞춰져 있을 가능성 점검. |

## Bimodal 신호 처리

gen2 형태가 binary("0건 OR 다건")로 splits되면 — 예: 2명의 gen1 친구가 전달조차 안 함, 다른 3명은 5명씩 전달 — gen1 단계에서 이미 friction이 있음. 이 경우:
- **gen1 전달율 < 60%** (5명 중 3명 미만 전달) → form 제출 결과와 무관하게 양가적 분류
- 이유는 open-ended로 추적: gen1이 전달 안 한 이유 ("어색해서" vs "쓸 만한데 친구한테 보낼 정도는 아님" vs "이런 거 친구한테 보내면 이상하게 볼 거 같아서")
- gen1 전달 friction이 *공유 동기 자체의 약함* 신호. plan §1 (a) 가설의 진짜 challenge.

## 결과 본 이후 *해서는 안 되는* 합리화 패턴 (사전 명시)

다음 표현을 결과 보고 사용하면 self-honesty 위반 — 결정행렬 그대로 적용:

- ~~"샘플이 작아서"~~ → 사전에 N=15 floor 인정하고 시작했음
- ~~"친구들이 음악에 관심 없어서"~~ → gen2는 친구들이 아니라 그들의 친구임. demographic 분리 필요했으면 gen1 선택 단계에서 통제했어야 함
- ~~"수동 처리 latency 때문에 mechanic이 죽었음"~~ → primary measurement는 form 제출이지 완성된 지도 보기가 아님. latency는 secondary metric에만 영향
- ~~"한 번 더 해보자"~~ → 본 문서가 *한 번의 thin slice*에 결정 강제력 부여. 두 번째 thin slice는 (다른 design, 다른 가설로) 새 pre-decision 문서 필요
- ~~"plan이 더 풍부하니까 실제로는 작동할 것"~~ → plan §1이 이미 자인했음: 비교 뷰 mechanic이 *최소* 작동해야 plan의 추가 mechanic이 lift 가능. 최소 작동 안 하면 lift 가능성 0에 수렴

## 양가적 케이스의 AC 보수화 — 구체 변경 (사전 박음)

thin slice 결과 양가적이면 plan은 다음과 같이 자동 patch (재논의 없이):

| Plan 위치 | 원래 값 | 양가적 케이스 값 |
|---|---|---|
| AC8b CTR target | ≥15% by Day 14 | ≥10% by Day 14 |
| AC8c KILL band 상한 | K < 0.15 | K < 0.20 |
| AC8c iterate band | 0.15 ≤ K < 0.25 | 0.20 ≤ K < 0.30 |
| AC8c scale tests band | 0.25 ≤ K < 0.40 | 0.30 ≤ K < 0.40 |
| AC8c v2 confirmed band | K ≥ 0.40 | K ≥ 0.40 (그대로) |
| Week 4 step 5 (OG image) 우선순위 | "ship if time permits" | **mandatory ship** (양가적 케이스에서 share CTA + OG 강화가 lift driver) |

## 결정 권한

이 문서의 행렬에 따른 결정은 Rae 본인이 자동 적용. 외부 합의 불필요. 단, **양가적 케이스에서 plan kickoff을 선택한 경우**, 위 "AC 보수화" 표를 plan 본문에 자동 반영하기 전에 30분 timebox로 "왜 더 보수화하지 않는가?"를 self-interview하고 답을 같은 폴더에 `docs/thin-slice-results.md`로 기록. 답이 "그냥 하고 싶어서"이면 plan kickoff을 1주 보류하고 다시 결정.
