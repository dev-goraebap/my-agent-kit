---
name: aggregate-designer
description: >
  ddd-workshop 파이프라인의 최종 산출물 스킬. 한 Bounded Context 내부의
  Aggregate 구조(Root/Entity/VO)·불변식(Invariant)·정책(Policy)을 설계하고,
  Aggregate 1개당 1개 .md 파일로 출력한다. 불변식(항상 참)과 정책(상황/시점
  따라 가변) 구분을 강제하고, 정책은 Strategy/Specification/Domain Service/
  Event Handler 중 어디에 둘지 제안한다. BC당 여러 번 호출 가능.
  "aggregate 설계", "애그리거트 설계", "invariant", "불변식",
  "domain policy", "정책 분리", "aggregate-designer", "ddd-workshop 7단계"
  같은 요청에 트리거한다.
metadata:
  author: dev-goraebap
  version: "0.1.0"
---

# aggregate-designer 스킬

ddd-workshop 파이프라인의 **최종 산출물**. 한 Bounded Context 안의 Aggregate 하나를 `docs/domains/{bc?}/{aggregate}.md` 파일로 설계한다.

이 스킬은 크게 두 가지 결정을 도와준다:

1. **일관성 경계(Aggregate)** — 어느 엔티티·VO를 한 Aggregate에 묶을 것인가
2. **불변식(Invariant) vs 정책(Policy)** — 항상 참인 것(Aggregate 내부) vs 상황에 따라 가변인 것(외부 분리)

## 레퍼런스

| 파일 | 용도 |
|------|------|
| `references/template.md` | Aggregate 문서 템플릿 (복제해서 채움) |
| `references/example-order.md` | 주문 Aggregate 완성 예제 (일반 커머스) |
| `references/example-organization.md` | 조직 Aggregate 예제 (사내 업무) |

---

## 1. 언제 트리거하는가

- `/ddd-workshop:aggregate-designer`
- `strategic-classifier` 출력 직후
- "Order Aggregate 설계해줘", "애그리거트 그려줘", "불변식 vs 정책 구분"
- BC를 명시하며 "[거래] Aggregate 설계"

**호출 방식**:
- **Aggregate 1개당 1회 호출**. BC에 여러 Aggregate가 있으면 여러 번 호출.
- 사용자가 시작 Aggregate를 지정하지 않으면 `strategic-classifier`의 **Core BC**부터 제안.

---

## 2. 입력

필수:
- `bounded-context-identifier` 출력 — 대상 BC
- `ubiquitous-language-extractor` 출력 — 용어·개념

강력 권장:
- `event-flow-mapper` 출력 — 이벤트·정책 단서
- `strategic-classifier` 출력 — 설계 상세도 결정(Core는 세밀히, Supporting은 표준 패턴)
- `requirement-clarifier` 출력 — 엣지 케이스·상태 전이

---

## 3. 핵심 원칙

### Aggregate 설계 규칙

- **작게 시작한다**. 의심스러우면 분리.
- **Root만 외부에 노출**. 내부 Entity·VO는 Root를 통해서만 접근.
- **Aggregate 간 참조는 ID로만**. 직접 객체 참조 금지.
- **한 트랜잭션 = 한 Aggregate**. 여러 Aggregate에 걸친 일관성은 이벤트·Saga로.

### 불변식 vs 정책 구분 ⭐

이 스킬의 가장 중요한 결정.

| 구분 | 불변식 (Invariant) | 정책 (Policy) |
|------|---------------------|----------------|
| **성격** | 항상 참이어야 함 | 상황·시점에 따라 가변 |
| **위치** | Aggregate 내부 강제 | **외부로 분리**, 교체 가능 |
| **변경 주기** | 도메인 본질 바뀔 때만 | 마케팅·운영이 자주 바꿈 |
| **예시** | "잔액 ≥ 0", "주문은 최소 1개 항목" | "VIP 10% 할인", "금액별 배송비" |

**판정 질문**:
- "이 규칙이 바뀌면 도메인 정의가 바뀌는가?" → Yes = 불변식
- "이 규칙은 마케팅·정책 변화에 따라 바뀔 수 있는가?" → Yes = 정책
- "운영 설정만으로 바꿀 수 있으면 좋겠는가?" → Yes = 정책
- "테스트에서 가짜로 대체하고 싶은가?" → Yes = 정책(교체 가능성)

### 정책의 구현 위치 선택

정책이 식별되면 어디에 두어야 할지 결정한다.

| 위치 | 언제 쓰나 | 특징 |
|------|-----------|------|
| **Specification** | 조건 판별이 여럿 모이거나 재사용 | 조건 객체, 조합·부정 가능 |
| **Strategy (Policy 객체)** | 교체 가능한 의사결정 규칙 | interface + 다형 구현 |
| **Domain Service** | 여러 엔티티가 얽혀 단일 Aggregate에 속하기 어색 | 상태 없음, 순수 로직 |
| **Event Handler** | 이벤트에 반응하는 비동기 정책 | BC 경계 넘는 반응에 적합 |

---

## 4. 진행 단계

### Step 1 — 대상 Aggregate 확정

BC 안의 "명사" 중 **생명주기를 가지는 것**이 Aggregate 후보. `ubiquitous-language-extractor`의 용어집에서:

- 생명주기 O (생성·변경·완료되는 것) → Aggregate / Entity 후보
- 불변 (값으로만 의미 있음) → VO 후보
- 상태·식별자 없음 → Domain Service 후보

Aggregate Root를 먼저 정한다.

### Step 2 — Role 작성

이 Aggregate가 **무엇을 책임지고 무엇은 책임지지 않는가**를 2~4줄로 기술. "이건 이 Aggregate의 몫이 아니다" 명시가 중요.

### Step 3 — Ubiquitous Language 확정

이 Aggregate 안에서 쓰는 용어만 뽑아 사전 형식으로 정리.

### Step 4 — Structure 결정

- Root: 1개
- 내부 Entity: Root에 종속, Root 없이 존재 의미 없음
- VO: 값으로만 의미 있음 (Money, Address, Email 등)
- 외부 참조: 다른 Aggregate·BC의 **ID만**

### Step 5 — Invariant 도출

"항상 참"이어야 할 규칙을 나열. 각 항목이 **진짜 불변식인지** 3번 원칙의 판정 질문으로 검증.

### Step 6 — Policy 분리

Step 5에서 "불변식은 아니지만 중요한 규칙"들을 Policy로 모은다. 각 정책에 구현 위치(Specification / Strategy / Domain Service / Event Handler)를 제안.

### Step 7 — External Dependencies

다른 Aggregate·BC·외부 시스템과의 관계. 참조 방식(ID 참조 / 계약 인터페이스 / 이벤트 구독) 명시.

### Step 8 — Open Questions

아직 결정되지 않은 설계 이슈 기록. 사용자 확인 필요 사항도 여기에.

### Step 9 — 파일 출력

- 기본 경로: `docs/domains/{aggregate}.md` (flat)
- BC가 명확하면: `docs/domains/{bc}/{aggregate}.md` (grouped)

---

## 5. 강제 질문 체크리스트

```
□ Role이 "책임 + 책임 아님"을 모두 서술
□ Root가 1개로 확정됨
□ 모든 외부 Aggregate 참조가 ID로만 이루어짐
□ Invariant 최소 2개 이상 (없는 Aggregate는 거의 없음)
□ 각 Invariant가 "3번 원칙 판정 질문"으로 검증됨
□ Policy 최소 1개 식별됨 (대부분 Aggregate에 정책이 있다 — 없다고 답하면 의심)
□ 각 Policy에 구현 위치 제안됨
□ 외부 의존이 ID 참조 vs 계약 인터페이스 vs 이벤트 구독으로 명시됨
□ Open Questions 섹션 있음 (없으면 "없음"을 명시)
```

---

## 6. 출력 포맷

`references/template.md`의 구조를 따른다. 핵심 형태:

```markdown
---
aggregate: <RootName>
bc: <BoundedContextName>
classification: {Core|Supporting|Generic}
status: active
version: 0.1.0
---

# <RootName>

## Role
...

## Ubiquitous Language
| 용어 | 의미 |

## Structure
- Root:
- 내부 Entity:
- VO:
- 외부 참조 (ID only):

## Invariants
- ...

## Policies
### [PolicyName]
- 위치: {Specification | Strategy | Domain Service | Event Handler}
- 이유: ...
- 설명: ...

## External Dependencies
| Aggregate / BC | 참조 방식 | 비고 |

## Domain Events (발행)
- ...

## Open Questions
- ⚠️ ...
```

---

## 7. 맥락별 조정

| 맥락 | Aggregate 수 | 상세도 | 비고 |
|------|--------------|--------|------|
| 학습 | 1~3개 | 핵심만 | 정책 1~2개 |
| 개인 | 3~5개 | 중간 | 단순 유지 |
| 사내 | 5~10개 | 상세 | 감사·이력 관련 Policy 주의 |
| B2B | 5~15개 | 상세 | 멀티테넌시 ID 참조(tenant_id) 필수 |
| B2C | 5~15개 | 상세 | 외부 시스템 ACL 경계 분명히 |

`strategic-classifier`의 분류별 상세도:
- **Core**: 엄격히. Invariant/Policy 구분 철저.
- **Supporting**: 표준 CRUD 수준.
- **Generic**: 보통 Aggregate 설계 **생략**하고 외부 솔루션 인터페이스만.

---

## 8. 불확실성 표기

- Invariant 판정이 모호하면 Policy로 내리는 것이 안전 (나중에 올리기 쉬움, 내리기 어려움).
- 외부 Aggregate 경계가 확실치 않으면 Open Questions에 기록.
- 운영 정책(VIP 할인 등)을 Invariant로 오인하면 ⚠️ 경고.

---

## 9. 파이프라인 종료

이 스킬이 실행되면 ddd-workshop 파이프라인의 **핵심 산출물이 완성**된다.

```
이후 단계 (ddd-workshop 범위 밖):
- 저장 전략이 필요하면 → 별도 ERD/DB 설계 단계
- API 설계가 필요하면 → 별도 도구
- 구현 착수는 `docs/domains/*.md` 파일들을 바탕으로

이 스킬은 BC별·Aggregate별로 여러 번 호출할 수 있다.
새 Aggregate가 필요하면 재호출.
```

---

## 10. 절대 하지 말 것

- Aggregate를 **크게** 잡기 (1개 Aggregate에 모든 엔티티 몰아넣기). 확장성·성능·트랜잭션 경계가 무너진다.
- 외부 Aggregate를 **객체 참조**로 가져오기. 반드시 ID만.
- 운영·정책 규칙을 Invariant로 박아넣기 (VIP 할인, 배송비 계산 등).
- Policy를 전부 Event Handler로 밀어넣기 (Specification/Strategy도 선택지).
- Open Questions 없이 "완벽한 설계"라고 선언하기.
- `strategic-classifier`가 Generic으로 분류한 BC에 자체 Aggregate 설계하기 (외부 솔루션 검토 먼저).
- "저장소 ORM 모델 = Aggregate" 착각 (ORM 설계는 이 단계가 아님).
