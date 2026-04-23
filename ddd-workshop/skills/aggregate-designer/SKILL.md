---
name: aggregate-designer
description: >
  ddd-workshop 파이프라인의 5단계 (최종 산출물, 전술적 설계). 한 BC 내부의
  Aggregate를 1개씩 설계한다. Design-Level Event Storming(이벤트·커맨드·
  액터·정책·핫스팟)을 내부 도입부에서 상세화하고, 그 흐름을 근거로
  Aggregate 구조(Root/Entity/VO), 불변식(Invariant), 정책(Policy 위치),
  그리고 **Exposed Queries(읽기 경로)** 를 결정한다. UL은 BC Canvas의
  per-BC UL을 이어받아 "용어 | Code Identifier | 의미" 이중 표기로
  참조·확장. Exposed Queries는 screen-inventory의 Query 이름과 크로스체크.
  산출물은 docs/shared/contexts/<bc>/<aggregate>.md 1개. BC 내 여러
  Aggregate는 여러 번 호출. "aggregate 설계", "애그리거트", "invariant",
  "불변식", "domain policy", "event storming 상세", "exposed queries",
  "read path", "aggregate designer", "ddd-workshop 5단계" 같은 요청에
  트리거한다.
metadata:
  author: dev-goraebap
  version: "0.3.0"
---

# aggregate-designer 스킬

ddd-workshop 파이프라인의 **최종 산출물**. 한 BC에서 Aggregate 1개를 완성된 `.md` 파일로 설계한다.

이 스킬은 **Design-Level Event Storming** (상세한 이벤트·커맨드·정책·핫스팟)을 내부 단계로 수행한 뒤, 그 흐름에서 Aggregate 경계·불변식·정책·**조회 경로**를 끌어낸다.

**v0.3 주요 변경:**
- 산출물 경로: `docs/domains/...` → `docs/shared/contexts/<bc>/<aggregate>.md`
- **Exposed Queries 섹션 신설** — read-path를 1급 산출물로 (screen-inventory의 Query와 매칭)
- UL은 BC Canvas의 per-BC UL을 이어받는 **이중 표기** (용어 / Code Identifier / 의미)
- YAML frontmatter 제거 (선택적 body 뱃지 `> **Status**: draft` 만 사용)

## 레퍼런스

| 파일 | 용도 |
|---|---|
| `references/template.md` | Aggregate 문서 템플릿 (v0.3 형식) |
| `references/example-order.md` | 주문 Aggregate (B2C) |
| `references/example-organization.md` | 조직 Aggregate (사내) |

---

## 대화 UX 규약 (필수)

- **카테고리 선택은 숫자 메뉴**. 자유 입력 금지.
- **1문 1답 기본**. 단순 Yes/No만 2~3개까지 번호 달아 묶기 허용.
- **진행 표시**: `[Q 3/15 · 근거: ...]`
- **"나중에" / "모름" / "스킵" 항상 허용** → `⚠️ 미정`으로 산출물에 기록.
- **자가 진단·평가는 ✅/⚠️/❌ 3단계**. 숫자 점수 금지.
- **UL은 이중 표기**: 용어(사용자 언어) | Code Identifier | 의미.

---

## 1. 언제 트리거하는가

- `/ddd-workshop:aggregate-designer`
- `context-designer` 출력 직후 (또는 `screen-inventory` 직후)
- "Order Aggregate 설계", "이벤트 스토밍 상세", "Invariant vs Policy 구분", "Exposed Queries 정리"
- BC와 Aggregate를 지정하며 호출 ("Leave Balance BC의 AnnualLeaveBalance")
- 기존 `.md` 업데이트 → `--update`

**호출 단위**: 한 세션에서 **여러 Aggregate를 연속 진행**한다. Step 10의 "다음 행동 메뉴"로 같은 세션에서 이어가기. 매번 재호출 강요 금지.

**Bulk draft 모드** (옵션): Supporting/Generic BC에 한해, 사용자가 명시 요청하면 여러 Aggregate를 짧은 초안으로 연이어 생성. **Core BC 금지**. §10 참조.

---

## 2. 입력

- `docs/shared/context-map.md` — 대상 BC 정보 + BC 간 관계
- `docs/shared/contexts/<bc>/_canvas.md` — **주 입력**
  - per-BC UL (Aggregate가 이어받음)
  - Inbound/Outbound (이 BC의 커맨드·쿼리·이벤트 계약)
  - Business Decisions (불변식 후보 포함)
- `docs/shared/contexts/<bc>/_screens.md` — (있으면) Query 이름 매칭 대상
- `docs/shared/requirements.md` — 엣지 케이스·제약
- `docs/shared/subdomain-map.md` — Core/Supporting/Generic (상세도 결정)
- `docs/shared/published-language.md` — (있으면) cross-BC 공용 용어

---

## 3. Step 0 — Upstream 일관성 스캔

자동 점검:

1. `_canvas.md` 존재 확인. 없으면 "먼저 `context-designer`로 Canvas 생성" 안내.
2. 대상 BC 분류 확인 → 상세도 조정.
   - Core → 이벤트 스토밍 철저 + Invariant/Policy 엄격 분리
   - Supporting → 표준 CRUD 수준
   - **Generic → 이 스킬 실행 만류. 외부 솔루션 인터페이스로 가라고 제안.**
3. Canvas의 UL을 이 BC 맥락으로 고정 (동음이의어 해소).
4. `_screens.md` 있으면 Query 목록 보관 → Step 9 Exposed Queries와 대조.
5. Canvas의 Business Decisions를 불변식 후보로 보관.

```
📋 Upstream 스캔
- 대상 BC: Leave Balance (Core)
- Canvas UL 로드: 10개 용어 (연차, 원장, 부여, 사용, ...)
- _screens.md 없음 — Exposed Queries는 Canvas의 Inbound Queries에서 시드
- Canvas Business Decisions 5개 → 불변식 후보 시드
- BC 내 예상 Aggregate 후보: AnnualLeaveBalance, LeaveLedgerEntry
- ⚠️ 퇴사 시 처리 방식 미정 (Canvas Open Questions)
```

---

## 4. 핵심 원칙

### Aggregate 설계 규칙
- **작게** 시작. 의심스러우면 분리.
- **Root만 외부 노출**. 내부 Entity·VO는 Root 경유.
- **Aggregate 간 참조는 ID로만**. 직접 객체 참조 금지.
- **한 트랜잭션 = 한 Aggregate**. 여러 Aggregate 걸친 일관성은 이벤트·Saga로.

### 불변식 vs 정책 구분 ⭐

| 구분 | 불변식 (Invariant) | 정책 (Policy) |
|---|---|---|
| 성격 | 항상 참 | 상황·시점에 따라 가변 |
| 위치 | Aggregate 내부 | **외부 분리**, 교체 가능 |
| 변경 주기 | 도메인 본질 바뀔 때만 | 마케팅·운영이 자주 바꿈 |
| 예시 | "잔액 ≥ 0", "주문 최소 1개 항목" | "VIP 10% 할인", "배송비 계산" |

**판정 질문**:
- 이 규칙이 바뀌면 도메인 정의가 바뀌는가? → Yes = 불변식
- 마케팅·운영 변화로 바뀔 수 있는가? → Yes = 정책
- 테스트에서 가짜로 대체하고 싶은가? → Yes = 정책

**애매하면 Policy로** (올리기 쉽고 내리기 어렵다).

### 정책 구현 위치

| 위치 | 언제 |
|---|---|
| **Specification** | 조건 판별 재사용·조합·부정 |
| **Strategy (Policy 객체)** | 교체 가능한 의사결정 규칙 |
| **Domain Service** | 여러 엔티티 얽혀 단일 Aggregate에 애매 |
| **Event Handler** | 이벤트에 반응하는 비동기 정책 |

### Read-side 원칙 ⭐

- Aggregate는 **쓰기 권위**. Read는 별도 Query Service를 통해 처리.
- **금지 패턴**: `Repository.findX().toDTO()` 로 Aggregate를 재조립해 조회용으로 쓰기. → Aggregate가 비대해지고 트랜잭션 경계 혼란.
- **권장**: Aggregate 상태 변경 시 Projection·View를 갱신 (이벤트 구독). 읽기는 Projection·Raw Read 직독.
- Exposed Queries는 **이 BC가 외부에 제공하는 읽기 계약**을 나열 — 구현 세부가 아닌 시그니처 수준.

---

## 5. 진행 단계

### Step 1 — Aggregate 대상 확정

```
[Q 1/15]

이번 회차에 설계할 Aggregate를 정합니다.
BC `Leave Balance`의 Aggregate 후보 (Canvas UL 기반):

1) AnnualLeaveBalance (잔고 권위, Root) — 권장
2) LeaveLedgerEntry (원장 엔트리, append-only)
3) 기타 (자유 입력)
4) 모름

DDD 권장: Core Aggregate Root부터 (권장: 1)
```

### Step 2 — Design-Level Event Storming

#### 2-1. 이벤트 시드 추출
대상 Aggregate와 관련된 이벤트를 과거형 시간 순 나열.

#### 2-2. 커맨드·액터 연결

```
[시스템] →(자동 부여)→ [연차_적립됨]
[결재자] →(승인)→ (Leave Request)→ [연차_차감요청됨]→ [연차_차감됨]
[관리자] →(조정)→ [연차_조정됨]
```

#### 2-3. 정책(Policy) 추출

```
[연차_차감됨] ─정책: 만료 경계 체크─→ (만료예정분_소멸시키기)
[사원_퇴사됨] ─정책: 진행 중 정산─→ (미사용분_정산하기)
```

#### 2-4. 분기·핫스팟

```
(consume) → [차감됨] or [잔고부족으로_거부됨]
⚠️ 만료 경계에서 취소된 신청의 복원 처리
```

#### 2-5. 사용자에게 흐름 제시

```
[Flow: 연차 생애]
...

1) 그대로 OK
2) 누락 이벤트 추가
3) 잘못 그려진 부분 수정
4) 핫스팟 해소부터 (권장 — 불변식/정책 판정에 영향)
```

### Step 3 — Role 작성

**책임지는 것 + 책임지지 않는 것** 둘 다.

### Step 4 — Structure 결정

- Root: 1개
- 내부 Entity 후보
- VO 후보
- 외부 참조 (ID만)

### Step 5 — Invariant 도출

각 규칙을 판정 질문으로 검증:

```
[Q · 규칙: "잔고 = 원장 누적 합계"]

판정 질문:
- 이 규칙이 바뀌면 도메인 정의가 바뀌는가? ✅
- 마케팅·운영이 바꿀 수 있는가? ❌
- 테스트에서 가짜 대체 원하는가? ❌

→ Invariant. 동의? (1) 동의 2) 실은 Policy 3) 모름
```

### Step 6 — Policy 분리

```
[Q · AccrualPolicy]

제안 위치: Strategy (근기법 공식 구현체 + 사내특례 구현체 교체 가능)
이유: 법 개정 시 구현체 교체로 대응
교체 시나리오: 2026년 근기법 개정 발효 시

동의? (1) 동의 (2) Specification (3) Domain Service (4) Event Handler (5) 불변식
```

### Step 7 — External Dependencies

다른 Aggregate·BC·외부 시스템 참조 방식 명시. context-map과 일치 검증.

### Step 8 — Domain Events 발행/구독

발행 이벤트 + 구독 이벤트.

### Step 9 — Exposed Queries ⭐ (신규)

**이 Aggregate가 외부에 제공하는 읽기 계약** 나열.

```
[Q · Exposed Queries]

Canvas의 Inbound Queries + screen-inventory 매칭:

| Query | 반환 | 원천 | 주의 |
|---|---|---|---|
| getBalance(employeeId) | {remaining, expiring} | Query Service | Aggregate 재조립 금지 |
| getLedger(employeeId, range) | LedgerEntry[] | 원장 테이블 직독 | append-only 보장 |

screen-inventory (_screens.md) 매칭 결과:
  ✅ listMyLeaveHistory — 없음 (이 BC에선 getLedger로 커버)
  ⚠️ getBalanceForecast — Canvas에 없음, 추가할지?

1) 이대로 OK
2) Query 추가
3) Query 매개변수 조정
4) screen-inventory 업데이트 필요 (별도 `screen-inventory` 재호출)
```

**원천(Source) 옵션:**
- **Query Service** — 읽기 전용 서비스, DTO 반환
- **Projection / View** — 이벤트로 갱신되는 읽기 모델
- **원장/Ledger 직독** — append-only 테이블 직접 쿼리
- **Aggregate 재조립** ← **금지**

### Step 10 — Open Questions

미확정 설계 이슈. 비어 있으면 "없음".

### Step 11 — 산출물 저장 + 다음 행동 메뉴 ⭐

파일 저장 후 숫자 메뉴로 묻는다. **재호출 강요 금지.**

```
✅ AnnualLeaveBalance Aggregate 설계 완료
   docs/shared/contexts/leave-balance/annual-leave-balance.md

다음으로 무엇을 하시겠어요?

1) 같은 BC(Leave Balance)의 다른 Aggregate 연속 진행 (LeaveLedgerEntry)
2) 다른 BC로 이동 (context-map 기준으로 제안)
3) 여기서 종료
4) 방금 만든 문서 수정·보강 (--update)
5) Bulk draft 모드 (Supporting/Generic 한정)
```

---

## 6. 산출물

기본 경로: **`docs/shared/contexts/<bc-slug>/<aggregate-slug>.md`**

**YAML frontmatter 없음.** 필요한 경우 본문 최상단에 뱃지 1줄:

```markdown
> **Status**: draft | active | deprecated
```

### 템플릿

본문 구조는 `references/template.md` 를 참조. 주요 섹션:

1. `# <RootName>` + `> **Status**: active` 뱃지
2. `## Role` — 책임지는 것 / 책임지지 않는 것
3. `## Ubiquitous Language (이 Aggregate 범위)` — 이중 표기 테이블
4. `## Event Storming (Design-Level)` — 플로우 + 해소/미해소 핫스팟
5. `## Structure` — Root / Entity / VO / 외부 참조
6. `## Invariants` — 항상 참인 규칙만
7. `## Policies` — 위치 + 이유 + 교체 시나리오
8. `## External Dependencies` — 관계 + 참조 방식
9. `## Domain Events (발행)` / `(구독)`
10. `## Exposed Queries` — 읽기 계약 + Read-side 규약 + screen-inventory 매칭
11. `## Open Questions`

**채워진 예시**: `references/example-order.md`, `references/example-organization.md`.

---

## 7. 업데이트 모드 (`--update`)

1. upstream 최신성 확인 (git). 특히 `_canvas.md` 변경 시 경계 영향 큼.
2. "무엇을 업데이트?" 숫자 메뉴:
   ```
   1) 새 Invariant 추가
   2) Policy 추가·위치 변경
   3) 이벤트 스토밍 갱신
   4) Exposed Queries 수정 (screen-inventory 동기화)
   5) Open Questions 해소
   6) 외부 의존 변경
   ```
3. 해당 부분만 delta 질문.

---

## 8. 강제 체크리스트

```
□ Step 0 upstream 스캔 완료
□ Canvas 존재 확인됨 (없으면 context-designer 먼저)
□ Generic BC면 실행 만류 확인함
□ 이벤트 스토밍에 비정상 경로(실패/취소/타임아웃) 포함
□ 핫스팟 최소 1개 기록 (없으면 "없음" 명시)
□ Role에 "책임 + 책임 아님" 둘 다 서술
□ Root 1개 확정
□ 외부 참조가 ID만 (context-map과 일치)
□ Invariant 최소 2개 + 각각 판정 질문 통과
□ Policy 최소 1개 + 위치 제안 + 교체 시나리오
□ Domain Events 발행·구독 둘 다 나열
□ Exposed Queries 섹션 존재 (없으면 "없음" 명시)
□ Exposed Queries에 "Aggregate 재조립 금지" 규약 언급
□ screen-inventory 있으면 Query 이름 매칭 기록
□ UL 이중 표기 (용어 / Code / 의미)
□ 산출물 경로: docs/shared/contexts/<bc>/<aggregate>.md
□ YAML frontmatter 없음 (status 뱃지만 허용)
□ Open Questions 섹션 존재
```

---

## 9. 맥락별 조정

| BC 분류 | 상세도 |
|---|---|
| **Core** | 최대 — Invariant/Policy 엄격 분리, 이벤트 스토밍 철저, Exposed Queries 명시. Bulk draft **금지**. |
| **Supporting** | 표준 — CRUD 중심, Policy 분리 최소화. Bulk draft 허용. |
| **Generic** | **실행 만류** — 외부 솔루션 인터페이스만 정의 권장. |

---

## 10. Bulk Draft 모드 (옵션, Supporting/Generic 전용)

사용자가 명시 요청했을 때만. Core BC 금지.

### 절차

1. **경고 + 확인**:
   ```
   ⚠️ Bulk draft 모드입니다.

   - Invariant vs Policy 구분 정확도 낮음
   - Exposed Queries는 screen-inventory 없이 Canvas만 참조
   - 생성 문서는 모두 `Status: draft`
   - Core BC 사용 불가

   대상 BC: Organization (Supporting)
   후보 Aggregate: Employee, Department, Position, Title

   진행? (1) 진행 (2) 순차 모드로
   ```
2. **사용자 승인 후**: 각 Aggregate를 짧게 자동 생성.
   - 이벤트 스토밍: Canvas·요구사항 기반 1~2줄 요약
   - Invariant: 명백한 2~3개
   - Policy: 이름만, 위치 제안은 `⚠️ 사용자 확정 필요`
   - Exposed Queries: Canvas Inbound Queries 복붙, 모두 `⚠️ 검토 필요`
   - Open Questions에 미확정 전부
3. **일괄 저장** → 각 파일 상단에 `> **Status**: draft` 뱃지.
4. **마무리 안내**: 각 문서의 Open Questions + ⚠️ 항목이 검토 대상.

### Bulk draft 금지 조건

- 대상 BC가 Core
- 사용자가 명시 요청하지 않음
- Aggregate가 1개뿐 (Bulk 의미 없음)

---

## 11. 절대 하지 말 것

- Generic BC에 Aggregate 설계 강행 (외부 솔루션 먼저 검토).
- Aggregate를 **크게** 잡기.
- 외부 Aggregate를 객체 참조로 가져오기 (ID만).
- 운영·정책 규칙을 Invariant로 박기.
- Policy를 전부 Event Handler로 밀기 (Specification/Strategy도 검토).
- 핫스팟을 "없음"으로 넘기기.
- 이벤트 스토밍을 **해피 패스만** 그리기.
- ORM 모델을 Aggregate와 동일시.
- **Exposed Queries 누락** (없으면 "없음" 명시).
- **Aggregate.load().toDTO() 패턴으로 읽기 구현** (Query Service 분리).
- Open Questions 없이 "완벽" 선언.
- 1문 1답 원칙 무시 (Core BC).
- 한 Aggregate 끝나고 **스킬을 재호출하게 만들기** (다음 행동 메뉴로 이어가기).
- Core BC에 Bulk draft 적용.
- Bulk draft 결과를 `Status: active`로 저장 (반드시 `draft`).
- **YAML frontmatter 추가** (status 뱃지만 허용).
- 산출물을 `docs/domains/` 에 저장 (반드시 `docs/shared/contexts/`).
