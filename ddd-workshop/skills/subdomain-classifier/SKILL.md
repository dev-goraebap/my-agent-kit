---
name: subdomain-classifier
description: >
  ddd-workshop 파이프라인의 3단계. event-storming-explorer가 뽑은
  서브도메인 후보를 Core / Supporting / Generic으로 **분류**한다.
  식별은 이미 끝난 상태이며 이 스킬의 역할은 **분류·투자 판단**에
  집중. Core는 Why(경쟁 차별점)에 근거. 이 분류는 이후 context-designer가
  어느 BC부터 Slice-first로 설계할지 결정하는 근거가 된다.
  "core supporting generic", "서브도메인 분류", "subdomain classifier",
  "어디에 집중할까", "외부 솔루션 쓸지", "slice first 대상 선정",
  "ddd-workshop 3단계" 같은 요청에 트리거한다.
metadata:
  author: dev-goraebap
  version: "0.3.0"
---

# subdomain-classifier 스킬

`event-storming-explorer`가 식별한 서브도메인 후보를 **Core / Supporting / Generic으로 분류**한다. 이 분류는 DDD 전략적 설계의 핵심 — **어디에 최고의 개발자와 시간을 투입할지**를 정한다.

**이 스킬은 식별은 하지 않는다.** 식별은 event-storming-explorer의 일이고, 여기선 이미 식별된 목록을 받아 분류만 한다. 식별과 분류는 서로 다른 사고 모드이기 때문에 분리되어 있다.

완료 후 **Slice-first 결정**(어느 Core 서브도메인을 먼저 [4] context-designer로 밀어볼지)을 사용자와 확정.

> **주의 ⚠️**: Core/Supporting/Generic은 **서브도메인**(문제 공간)의 분류이지 BC(해결 공간)의 분류가 아니다. BC는 [4]에서 분류된 서브도메인을 근거로 설계한다.

---

## 대화 UX 규약 (필수)

- **카테고리 선택은 숫자 메뉴**. 자유 입력 금지.
- **1문 1답 기본**. 단순 Yes/No만 2~3개까지 번호 달아 묶기 허용.
- **진행 표시**: `[Q 3/5 · 서브도메인: 연차 잔고 회계]`
- **"나중에" / "모름" / "스킵" 항상 허용** → `⚠️ 미정`으로 산출물에 기록.
- **자가 진단·평가는 ✅/⚠️/❌ 3단계**. 숫자 점수 금지.
- **정답보다 올바른 질문 생성**이 스킬의 핵심 가치.

---

## 1. 언제 트리거하는가

- `/ddd-workshop:subdomain-classifier`
- `event-storming-explorer` 출력 직후
- "Core 뭐야?", "어디에 집중?", "외부 솔루션 써도 될지"
- 기존 `subdomain-map.md`의 분류 업데이트 → `--update` 모드

---

## 2. 입력

- `docs/shared/subdomain-map.md` — **주 입력** (분류 컬럼이 비어 있어야 함)
- `docs/shared/event-flow.md` — 이벤트 군집으로 서브도메인의 리듬·관심사 파악
- `docs/shared/requirements.md` — **Why·경쟁 차별점은 Core 판정의 1순위 근거**

---

## 3. Step 0 — Upstream 일관성 스캔

자동 점검:

1. `subdomain-map.md`의 분류 컬럼이 **이미 채워져 있으면** 경고 → `--update` 모드로 전환 권유.
2. `requirements.md`의 Why가 `⚠️ 미정`이면 "Core 판정 정확도 낮아짐" 알림 + 먼저 [1]로 돌아갈지 선택.
3. `event-flow.md`에 핫스팟 `⚠️`가 있으면 알림 — 분류 결정에 영향 가능.

결과 공개:

```
📋 Upstream 스캔
- 서브도메인 후보: 4개 (조직, 연차잔고, 연차신청, 알림)
- Why: "근기법 기반 정확한 연차 계산으로 법적 리스크 제거"
- ⚠️ 핫스팟 2개 (퇴사 시 처리, 만료 경계)
- 기존 분류: 비어 있음 ✅

진행할까요? (1) 진행 2) Why 재검토 3) [1]로 복귀)
```

---

## 4. 분류 기준

### Core 서브도메인
- **정의**: 이 프로젝트의 경쟁력 원천. Why와 직결.
- **판정 질문**:
  - `requirements.md`의 Why가 이 서브도메인에 집약되는가? ✅/⚠️/❌
  - 경쟁 차별점이 여기 모이는가? ✅/⚠️/❌
  - 이 서브도메인이 없으면 프로젝트 본질이 바뀌는가? ✅/⚠️/❌
- **권고**: 최고 역량 투입, DDD 전술 패턴 꼼꼼히, 외주·외부 솔루션 금지.

### Supporting 서브도메인
- **정의**: 필요하지만 차별점은 아님. 도메인 지식이 있어 범용 솔루션 대체는 어려움.
- **권고**: 표준 CRUD·패턴, 사내 개발, 코드 리뷰는 Core보다 느슨.

### Generic 서브도메인
- **정의**: 어디나 있는 범용 기능. 외부 솔루션 대체 합리적.
- **권고**: **외부 솔루션 우선 검토** (결제→PG, 인증→Auth0, 이메일→SendGrid, 검색→Algolia 등). 자체 구현은 규제·비용상 필요할 때만.

---

## 5. 진행 단계

### Step 1 — 서브도메인당 분류 판정 (1문 1답)

```
[Q 1/4 · 서브도메인: 연차 잔고 회계]

근거 (requirements.md § Why 인용):
- Why: "근기법 기반 정확한 연차 계산으로 법적 리스크 제거"
- 경쟁 차별점: 대부분 HR 제품이 부정확함 → 정확성이 차별

에이전트 제안: **Core**
평가:
  - Why 직결: ✅
  - 경쟁 차별점: ✅
  - 없으면 본질 바뀜: ✅

이 분류에 동의하시나요?

1) 동의 (Core)
2) Supporting으로 내림 (근거 필요)
3) Generic으로 내림 (외부 솔루션 가능 근거 필요)
4) 모름 / 나중에
```

### Step 2 — Core 수 검증

Core는 보통 **1~2개**. 3개 이상이면 재검토:

```
현재 Core 분류된 서브도메인이 3개입니다. DDD 정론상 Core는 1~2개가 일반적입니다.

1) 유지 (각각 진짜 차별점이라 확신)
2) 우선순위 매김 (Core지만 1순위/2순위 구분)
3) 재검토 (일부를 Supporting으로 내림)
```

### Step 3 — Generic 외부 솔루션 매칭

Generic 분류된 각 서브도메인에 **대체 외부 솔루션 후보 1개 이상** 제시. requirements.md 도메인 브리프의 "주요 플레이어"에서 꺼내 씀.

규제·보안·비용 때문에 외부 솔루션 불가한 경우 이유 명시.

### Step 4 — Slice-first 결정 ⭐

```
✅ 서브도메인 분류 완료.

DDD에서 권장되는 slice-first 접근을 안내드립니다:
→ Core 서브도메인 **1개**를 [4]→[5]로 끝까지 먼저 설계해보기.
→ 그 학습을 바탕으로 나머지 서브도메인을 넓혀가기.

이유: 여러 서브도메인을 병행 설계하면 요구사항 변동 시 재작업 비용이 큽니다.

어떻게 진행할까요?

1) Slice-first (Core 1개 선택 후 [4] context-designer로)
2) 전체 BC 설계 먼저 ([4]에서 모든 서브도메인 병행)
3) 일단 여기서 중단, 나중에 재개
```

선택 결과를 산출물에 기록.

---

## 6. 산출물

**기존 `docs/shared/subdomain-map.md`에 이어서 작성한다.** 새 파일을 만들지 않는다.

업데이트 대상:
- 분류 컬럼 채움 (Core / Supporting / Generic)
- Why 컬럼 채움
- 각 서브도메인 상세 섹션에 판정 근거 + 권고 추가
- 말미에 Slice-first 결정 섹션 추가

**Frontmatter 없음.** 본문만.

최종 파일 구조 예시:

```markdown
# Subdomain Map

> 2단계(event-storming-explorer)에서 후보 식별 완료 →
> 3단계(subdomain-classifier)에서 분류 완료.

## 서브도메인 분류

| # | 서브도메인 | 포함 이벤트 | 분류 | Why |
|---|---|---|---|---|
| A | 조직 관리 | 사원 생애주기, 부서, 직위 | Supporting | 차별 아님, 사내 개발 유지 |
| B | 연차 잔고 회계 | 적립·차감·소멸·조정 | **Core** ⭐ | Why "근기법 정확성" 집약 |
| C | 연차 신청·결재 | 신청·제출·승인·반려·취소 | Supporting | Core 보조 |
| D | 알림 | 이메일·웹푸시 발송 | Generic | 외부 솔루션 가능 |

---

## Core 서브도메인

### B. 연차 잔고 회계 ⭐
- **Why 근거**: requirements § Why — "근기법 기반 정확한 연차 계산으로 법적 리스크 제거"
- **경쟁 차별점**: 대부분 HR 제품이 엣지 케이스에서 부정확함
- **Core 판정 평가**:
  - Why 직결: ✅
  - 경쟁 차별점: ✅
  - 없으면 본질 바뀜: ✅
- **권고**:
  - 최고 역량 개발자 투입
  - Aggregate·Invariant·Policy 철저 분리
  - 원장(Ledger) 패턴 도입, append-only
  - 외주·외부 솔루션 금지
  - 변경 이력 ADR

---

## Supporting 서브도메인

### A. 조직 관리
- 왜 Core 아닌가: Why와 직결되지 않음 (HR 제품 어디나 있음)
- 권고: 표준 CRUD

### C. 연차 신청·결재
- 왜 Core 아닌가: Core(잔고 회계) 보조 역할
- 권고: 상태머신 설계, 결재선 확장 대비

---

## Generic 서브도메인

### D. 알림
- 외부 솔루션 후보: SendGrid, Resend, FCM, NHN Cloud
- 자체 필요 케이스: 없음 (MVP 기준)
- 권고: ACL로 외부 모델 격리

---

## ⚠️ 경계 사례

(있으면 기재. 없으면 "없음")

---

## Slice-first 결정

- **1순위 진행 대상**: B. 연차 잔고 회계 (Core)
- **이유**: Why 집약 + 경쟁 차별점, 엣지 케이스 많아 초기 검증 가치 높음
- **[4] context-designer에서 이 서브도메인부터 BC 설계 시작**
- 나머지는 slice 1 완료 후 확장

---

## 미정
- ⚠️ ...

## 다음 단계
→ `context-designer`로 **"B. 연차 잔고 회계" 서브도메인부터** BC 설계.
  Slice 1 완료 후 나머지로 확장.
```

---

## 7. 업데이트 모드 (`--update`)

1. upstream (`requirements.md`, `event-flow.md`, `subdomain-map.md`) 최신성 확인.
2. 사용자에게 "무엇이 바뀌었나요?" 숫자 메뉴:
   ```
   1) Why / 경쟁 차별점 변경 → Core 재판정
   2) 새 서브도메인 후보 추가 (event-storming-explorer에서)
   3) 기존 서브도메인 분류 변경
   4) Slice-first 대상 변경
   ```
3. 해당 부분만 재질문.

---

## 8. 강제 체크리스트

```
□ Step 0 upstream 스캔 완료
□ 각 서브도메인에 ✅/⚠️/❌ 3문 평가 기록
□ Core 근거에 requirements.md Why 인용 있음
□ Core 수 1~2개 (3+이면 재검토 기록)
□ 각 Generic에 외부 솔루션 후보 1개 이상
□ 경계 사례 있으면 명시 ("경계 없음"도 가능)
□ Slice-first 결정 기록됨 (1/2/3 중 선택)
□ 산출물이 "서브도메인"을 분류함 (BC 분류 아님)
□ subdomain-map.md에 이어 작성됨 (새 파일 생성 안 함)
□ Frontmatter 없음
```

---

## 9. 맥락별 조정

| 맥락 | Core 수 | Generic 외부 솔루션 | Slice-first 강도 |
|---|---|---|---|
| 학습 | 전부 Core OK | 자체 구현 (학습 목적) | 강함 (하나씩 배움) |
| 개인 | 1개 | 적극 외부 | 강함 |
| 사내 | 1~2개 | 사내 정책 따름 | 중간 |
| B2B | 1~2개 | 보안 요구 시 자체 | 강함 (고객 파일럿 슬라이스) |
| B2C | 1~2개 | 적극 외부 | 강함 |

---

## 10. 절대 하지 말 것

- **서브도메인을 식별하기** (event-storming-explorer의 일).
- BC를 Core/Supporting/Generic으로 분류하기 (**서브도메인**이 분류 단위).
- 모든 서브도메인을 Core로 분류 (전략적 집중 실패).
- Why·경쟁 근거 없이 감으로 Core 판정.
- Generic에 외부 솔루션 후보 없이 "알아서" 마치기.
- 분류를 한 메시지에 전부 쏟아 사용자에게 제시 (서브도메인당 1문 1답).
- Slice-first 결정 생략 ([4] 진입 전 필수).
- 사용자가 "모름"이라 답했는데 임의 분류 확정.
- 새 파일 만들기 (기존 `subdomain-map.md`에 이어 작성).
- Frontmatter 추가.
