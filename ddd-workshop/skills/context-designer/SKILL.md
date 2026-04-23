---
name: context-designer
description: >
  ddd-workshop 파이프라인의 4단계. 분류된 서브도메인에서 Bounded Context를
  설계하고, **BC마다 BC Canvas(1페이지 요약, per-BC Ubiquitous Language
  내장) 파일**과 **Context Map(BC 간 관계)** 을 생성한다. UL은 전역이
  아니라 per-BC이며 각 BC Canvas 안에 "용어 | Code Identifier | 의미"
  이중 표기로 들어간다. Cross-BC 공용 용어·enum·ID 타입만 별도
  published-language.md에 둔다. Slice-first 권장: 선택된 Core
  서브도메인의 BC부터 설계하고 직접 경계하는 이웃 BC만 함께 그린다.
  산출물 폴더는 domains/가 아니라 contexts/. "BC 설계",
  "bounded context", "context map", "BC canvas", "컨텍스트 맵",
  "캔버스", "context designer", "ddd-workshop 4단계" 같은 요청에
  트리거한다.
metadata:
  author: dev-goraebap
  version: "0.3.0"
---

# context-designer 스킬

**서브도메인(문제 공간)** 에서 **Bounded Context(해결 공간)** 를 설계한다. 서브도메인 ↔ BC 매핑은 1:1이 기본이지만, 1:N(큰 서브도메인을 쪼갬)이나 N:1(작은 서브도메인 묶음)도 가능하다.

**이 스킬의 산출물은 3종류:**
1. `docs/shared/context-map.md` — 전역 1장, BC 목록 + BC 간 관계
2. `docs/shared/contexts/<bc>/_canvas.md` — BC마다 1장, **per-BC UL 내장**
3. `docs/shared/published-language.md` — (선택) cross-BC 공용 용어만

UL은 전역이 아니라 per-BC라는 정통 원칙에 따라 Canvas 내부에 박는다. 전역 `ubiquitous-language.md`는 사용하지 않는다.

**Slice-first 권장**: `subdomain-classifier`가 선택한 Core 서브도메인의 BC부터 설계하고, **그 BC와 직접 경계하는 이웃 BC만** 함께 그린다. 전체 BC 맵은 시간이 흐르며 완성.

---

## 대화 UX 규약 (필수)

- **카테고리 선택은 숫자 메뉴**. 자유 입력 금지.
- **1문 1답 기본**. 단순 Yes/No만 2~3개까지 번호 달아 묶기 허용.
- **진행 표시**: `[Q 3/12 · 근거: ...]`
- **"나중에" / "모름" / "스킵" 항상 허용** → `⚠️ 미정`으로 산출물에 기록.
- **자가 진단·평가는 ✅/⚠️/❌ 3단계**. 숫자 점수 금지.
- **UL은 이중 표기 강제**: "용어(사용자 언어) | Code Identifier(영어) | 의미".

---

## 1. 언제 트리거하는가

- `/ddd-workshop:context-designer`
- `subdomain-classifier` 출력 직후
- "BC 설계", "context map", "BC canvas 만들어줘"
- 기존 `context-map.md` 또는 `_canvas.md` 업데이트 → `--update`

---

## 2. 입력

- `docs/shared/subdomain-map.md` — **주 입력**
  - Slice-first 대상 Core 서브도메인
- `docs/shared/event-flow.md` — 이벤트·동음이의어 근거 (BC 경계의 1순위 증거)
- `docs/shared/requirements.md` — 조직·외부 시스템 맥락, UL 기본 언어

---

## 3. Step 0 — Upstream 일관성 스캔

자동 점검:

1. upstream 3개 파일의 존재 확인. 없으면 해당 단계로 돌려보냄.
2. `subdomain-map.md`의 "Slice-first 결정" 섹션 확인.
3. `event-flow.md`의 동음이의어 목록을 **BC 경계의 1순위 증거**로 로드.
4. `requirements.md`의 외부 시스템 목록 → ACL/Conformist 후보로 보관.
5. UL 기본 언어 확인 (미지정이면 질문).

결과 공개:

```
📋 Upstream 스캔
- Slice-first 대상: B. 연차 잔고 회계 (Core)
- UL 기본 언어: 한국어
- 동음이의어 2종 로드 (연차, 취소)
- 외부 시스템: 이메일 공급자(미정), FCM
- ⚠️ 미정: 결재선 범용화 여부 (requirements.md)

진행할까요? (1) 진행 2) 미정 해소 후 진행 3) [3]으로 복귀)
```

---

## 4. 진행 단계

### Step 1 — Slice 범위 확정

```
이번 세션에서 BC 설계 범위를 확정합니다.

Slice-first 대상 서브도메인: B. 연차 잔고 회계
포함할 이웃 BC 후보:
  - A. 조직 관리 (사원 정보 공급)
  - C. 연차 신청·결재 (잔고 차감·복원 호출)
  - D. 알림 (이벤트 구독)

이 범위로 진행할까요?

1) 그대로 진행
2) 이웃 BC 조정
3) 전체 BC 맵 설계 (slice 중단, 모든 서브도메인 일괄)
```

옵션 3은 **요구사항 변동 위험** 경고 후 사용자 재확인.

### Step 2 — 서브도메인 → BC 매핑

각 서브도메인에 대해:

```
[Q · 서브도메인: B. 연차 잔고 회계]

이 서브도메인의 BC 구성을 결정합니다.

1) 1:1 — 서브도메인 전체를 하나의 BC로 (Leave Balance BC)
2) 1:N — 두 개 이상의 BC로 쪼갬
3) N:1 — 인접 서브도메인과 묶음
4) 모름 / 나중에

현재 언어 증거: 동음이의어 "연차"(잔고) vs "종일 휴가"(휴가 종류)
  → 잔고 계산 BC와 휴가 종류 분리 여지
```

### Step 3 — 각 BC의 Canvas 작성 (BC당 1세션)

BC마다 다음 항목을 묶음 질문:

```
[BC Canvas: Leave Balance]

다음을 확정합니다. 1문씩 묶어 질문합니다.

Q1. Purpose (한 문장):
  제안: "연차 잔고의 회계 권위. 부여·사용·조정·소멸 무결성 보장."
  1) 그대로 2) 수정

Q2. Strategic Classification:
  - Domain: Core (subdomain-map 그대로)
  - Why: requirements Why 인용

Q3. Inbound Communication:
  - Commands: consume, restore, adjust
  - Queries: getBalance, getLedger
  - Subscribed Events: EmployeeActivated, EmployeeTerminated

Q4. Outbound Communication:
  - Events: LeaveBalanceAccrued, LeaveBalanceExpired,
            LeaveAdjusted, LeaveConsumed, LeaveRestored
  - Calls: checkPermission (to Access Control)

Q5. Business Decisions (3~5개):
  제안:
  - 잔고 = 원장 누적 합계 (불변식)
  - Accrual은 (employeeId, accrualPeriod) 단위 idempotent
  - 잔고 부족 시 consume 거부
  1) 그대로 2) 추가/수정

Q6. Ubiquitous Language (이중 표기):
  | 용어 | Code Identifier | 의미 |
  | 연차 | AnnualLeave | 사원의 유급휴가 잔고 |
  ...

Q7. Assumptions / Open Questions
```

### Step 4 — BC 간 관계 설계 (Context Map)

BC 쌍마다 관계 패턴을 결정:

```
[Q · 관계: [Leave Request] → [Leave Balance]]

이 두 BC는 어떻게 연결되나요?

1) Customer-Supplier (이벤트 기반)
2) Conformist (하류가 상류 모델 그대로 수용)
3) ACL (하류가 자체 인터페이스, 상류 모델 번역·격리)
4) Shared Kernel (공유 모델, 매우 드물게)
5) Open Host Service + Published Language (상류가 공용 프로토콜 공개)
6) Partnership (두 팀 공동 소유)
7) Separate Ways (통합 안 함)
8) 모름

권장: ACL — Core(Leave Balance) 보호, 차감·복원 로직을 Leave Balance 내부가 책임
```

### Step 5 — Context Map 다이어그램

텍스트 다이어그램으로 관계 시각화. 기존 패턴 유지.

### Step 6 — Published Language 식별 (선택)

**진짜 cross-BC에 공유되는 것**만 여기 분리. 일반 용어는 각 BC Canvas에만.

```
[Q · Published Language 후보]

여러 BC가 **같은 이름·같은 의미**로 공유하는 용어/Enum/ID가 있나요?

예: Permission enum (Access Control이 정의, 모든 BC가 참조)
예: EmployeeId (Organization이 발행, 다른 BC는 ID 타입으로만 참조)

1) 있음 (목록 제공 → published-language.md 생성)
2) 없음 (이 파일 생성 안 함)
```

### Step 7 — 미정 항목과 리스크 정리

다음을 명시:

- 아직 확정 못 한 관계 (⚠️ 미정)
- 고위험 관계 (Shared Kernel, Partnership) → 추후 재검토 제안
- 외부 시스템 실제 인터페이스 미확인 → 확인 필요 표시

---

## 5. 산출물

**Frontmatter 없음.** 본문만.

### 5.1 `docs/shared/context-map.md`

```markdown
# Context Map

## Slice 범위
- 대상 Core BC: Leave Balance
- 포함 이웃 BC: Organization, Leave Request, Notification
- 제외: Access Control (별 슬라이스), Identity (범위 밖)

## 서브도메인 → BC 매핑

| 서브도메인 | 분류 | BC | 매핑 | 비고 |
|---|---|---|---|---|
| B. 연차 잔고 회계 | Core ⭐ | Leave Balance | 1:1 | Slice 대상 |
| A. 조직 관리 | Supporting | Organization | 1:1 | |
| C. 연차 신청·결재 | Supporting | Leave Request | 1:1 | |
| D. 알림 | Generic | Notification | 1:1 | |

---

## Bounded Context 상세

각 BC의 상세는 `contexts/<bc>/_canvas.md` 참조.

### Leave Balance ⭐
- 핵심 책임: 연차 잔고의 회계 권위
- Canvas: `contexts/leave-balance/_canvas.md`

### Organization
- 핵심 책임: 조직·사원 마스터
- Canvas: `contexts/organization/_canvas.md`

(이하 생략)

---

## Context Map (관계 다이어그램)

```
(다이어그램)
```

## BC 간 관계 표

| From | To | 관계 | 근거 |
|---|---|---|---|
| Leave Request | Leave Balance | ACL | Core 보호, 차감·복원 명시 호출 |
| Organization | Leave Balance | Customer-Supplier (이벤트) | EmployeeTerminated 등 수신 |
| ... | ... | ... | ... |

## ⚠️ 경계 판단 애매 지점
## ⚠️ 고위험 관계
## ⚠️ 미확인 사항
## 다음 단계
→ `aggregate-designer`로 Core BC의 Aggregate 설계.
```

### 5.2 `docs/shared/contexts/<bc>/_canvas.md`

BC마다 1개. 언더스코어 접두사로 Aggregate 파일과 구분.

```markdown
# Leave Balance — BC Canvas

## Purpose
연차 잔고의 회계 권위. 부여·사용·조정·소멸의 무결성을 보장.

## Strategic Classification
- **Domain**: Core ⭐
- **Why Core**: 근기법 기반 잔고 계산은 우리 HR의 핵심 차별점.
  오류 시 법적·재무적 리스크가 큼.
- **Business Model**: Compliance + Revenue
- **Evolution**: Product

---

## Inbound Communication

### Commands (from Leave Request BC)
- `consume(employeeId, amount, requestId, reason)`
- `restore(employeeId, amount, requestId, reason)`

### Commands (from Admin UI)
- `adjust(employeeId, delta, reason, adjustedBy)`

### Queries
- `getBalance(employeeId)`
- `getLedger(employeeId, range)`

### Subscribed Events (from Organization BC)
- `EmployeeActivated` → LeaveBalanceInitialized
- `EmployeeTerminated` → 진행 중 정산

---

## Outbound Communication

### Events Published
- `LeaveBalanceAccrued`
- `LeaveBalanceExpired`
- `LeaveAdjusted`
- `LeaveConsumed`
- `LeaveRestored`
- `LeaveLedgerEntryAppended`

### Calls (sync)
- `Access Control.checkPermission(...)` — 모든 관리자 커맨드 앞

---

## Ubiquitous Language

| 용어 | Code Identifier | 의미 |
|---|---|---|
| 연차 | AnnualLeave | 사원의 유급휴가 잔고 |
| 잔여 연차 | LeaveBalance | 현재 사용 가능한 일수 (원장 합계 파생) |
| 원장 | LeaveLedger | 잔고 변동 회계 기록 (append-only) |
| 원장 엔트리 | LeaveLedgerEntry | 원장의 한 줄 |
| 부여 | Accrual | 근기법 공식 기반 자동 적립 |
| 사용 | Consumption | 승인된 신청에 따른 차감 |
| 복원 | Restoration | 취소 시 되돌림 |
| 조정 | Adjustment | 관리자 수기 보정 |
| 소멸 | Expiration | 발생 후 1년 경과분 자동 폐기 |
| 부여 주기 | AccrualPeriod | Accrual 멱등성 키 (YYYY-MM 등) |

### 주의사항 (동음이의어·범위)
- "연차"(여기)는 **잔고 개념**. 휴가 종류의 "종일 휴가"와 별개.
- "신청"은 이 BC의 언어가 **아님** — Leave Request BC 소관.

---

## Business Decisions

1. **잔고 = 원장 누적 합계** (불변식)
2. Accrual은 `(employeeId, accrualPeriod)` 단위 idempotent
3. 잔고 부족 시 `consume` 거부
4. `EmployeeTerminated` 수신 시 진행 중 Accrual 중단 + 미사용분 정산
5. Adjustment는 `adjustedBy` + `reason` 필수, audit log 영구 보존

---

## Assumptions
- 결재·승인 로직은 Leave Request BC가 소유, 이 BC는 결과만 수신
- 이메일 공급자는 Notification BC가 추상화

## Open Questions
- ⚠️ 11개월 근무 후 퇴사 시 1개 적립 여부 (근기법 해석)
- ⚠️ 만료 경계 ±1일 처리

---

## Aggregates (이 BC 소속)

- AnnualLeaveBalance (권위 Aggregate)
- LeaveLedgerEntry (append-only)

각각의 상세 명세는 `./<aggregate>.md` 참조.
```

### 5.3 `docs/shared/published-language.md` (선택)

```markdown
# Published Language — 전사 공용 프로토콜

이 파일의 용어는 **여러 BC가 동일 의미로 공유**하는 것만 담는다.
특정 BC 내부 언어는 각 BC Canvas의 UL 섹션에.

## 공용 Enum

### Permission
Access Control BC가 정의하고 모든 BC가 참조.

| 값 | Code | 의미 |
|---|---|---|
| 휴가 관리 | LEAVE_MANAGE | 타인의 연차 조정 |
| 휴가 승인 | LEAVE_APPROVE | 휴가 신청 결재 |

## 공용 Event 이름 규약

- 이벤트 네이밍: `<주어>_<과거형 동사>` / `<Subject><PastVerb>`
- 예: `사원_초대됨` / `EmployeeInvited`

## 공용 ID 타입

- `EmployeeId` — Organization BC가 발행, 다른 BC는 ID로만 참조 (강결합 금지)
```

---

## 6. 업데이트 모드 (`--update`)

1. upstream `updated_at` 비교 (git) → stale 경고.
2. "무엇을 업데이트?" 숫자 메뉴:
   ```
   1) Slice 범위 변경
   2) 서브도메인 → BC 매핑 수정
   3) 특정 BC Canvas 수정
   4) BC 간 관계 재설계
   5) 새 외부 시스템 추가
   6) Published Language 항목 추가/제거
   ```
3. 해당 부분만 delta 질문.

---

## 7. 강제 체크리스트

```
□ Step 0 upstream 스캔 완료
□ Slice 범위 명시됨 (대상 BC + 이웃 BC)
□ 서브도메인 → BC 매핑 표 완성
□ 각 BC마다 _canvas.md 파일 생성
□ 각 Canvas에 Purpose / Classification / I-O / UL / Decisions / Questions
□ UL 테이블이 이중 표기 (용어 / Code / 의미)
□ 동음이의어가 Canvas의 "주의사항"에 명시됨
□ BC 쌍마다 관계 패턴 결정 (미정은 ⚠️)
□ 외부 시스템은 ACL 또는 Conformist (Partnership 희귀)
□ Context Map 다이어그램 포함
□ Published Language는 진짜 공용만 (없으면 파일 생성 안 함)
□ 산출물 폴더: domains/ 가 아니라 contexts/
□ Frontmatter 없음
```

---

## 8. 맥락별 조정

| 맥락 | BC 수 | Slice 강도 | 관계 주의 |
|---|---|---|---|
| 학습 | 1~2 | 매우 강함 | 단순 |
| 개인 | 2~3 | 강함 | 과분리 금지 |
| 사내 | 3~6 | 강함 | 조직 경계 존중 |
| B2B | 3~6 | 강함 | 멀티테넌시 경계 별도 |
| B2C | 3~7 | 중간 | 외부 연동 많음, ACL 중심 |

---

## 9. 절대 하지 말 것

- Core/Supporting/Generic을 BC 분류로 다시 쓰기 (서브도메인 분류임).
- 동음이의어 증거 무시하고 기술 레이어(UI/Service/DB)로 BC 자르기.
- 모든 BC를 한 번에 설계 (Slice-first 무시).
- 외부 시스템과 Partnership/Shared Kernel 남용 (대부분 ACL/Conformist).
- UL을 전역 `ubiquitous-language.md` 한 파일에 몰아넣기 (**per-BC 원칙 위반**).
- Published Language를 "혹시 몰라" 미리 만들기 (진짜 공용만).
- 산출물을 `domains/` 에 배치 (contexts/ 사용).
- 사용자가 "모름"이라 답한 관계를 임의 확정.
- Frontmatter 추가.

---

## 10. 왜 per-BC UL인가

Evans/Vernon 정통: **UL은 Bounded Context 안에서만 일관된다.**
- 같은 단어라도 BC마다 의미가 다름 (동음이의어 = BC 경계 신호)
- 전역 UL은 이 원칙을 흐림 — 모든 BC가 "한 국어로 얘기해야 한다"는 착시

그래서 이 스킬은:
- UL을 **BC Canvas 안 섹션**으로 고정
- 전역 `ubiquitous-language.md`를 **만들지 않음**
- 진짜 cross-BC 용어만 `published-language.md`에 분리
