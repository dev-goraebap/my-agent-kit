# Aggregate 문서 템플릿

`docs/domains/{aggregate}.md` 또는 `docs/domains/{bc}/{aggregate}.md` 로 복제해 채운다.

---

```markdown
---
aggregate: <RootName>
bc: <BoundedContextName>
classification: {Core|Supporting|Generic}
status: {draft|active|deprecated}
version: 0.1.0
last_reviewed: YYYY-MM-DD
---

# <RootName>

## Role

이 Aggregate가 책임지는 것을 2~4줄로 서술하고,
**책임지지 않는 것**을 반드시 함께 명시한다.
(예: "환불·배송·정산은 이 Aggregate의 책임이 아니다.")

## Ubiquitous Language

| 용어 | 의미 |
|------|------|
| <개념1> | ... |
| <개념2> | ... |

## Structure

- **Root**: <RootName>
- **내부 Entity**: <Entity1>, <Entity2> — Root 없이 존재 의미 없음
- **VO**: <VO1>, <VO2> — 값으로만 의미 있음
- **외부 참조 (ID only)**: `<otherAggId>: <type>`

## Invariants

"항상 참"이어야 하는 규칙만 여기에.
정책은 아래 Policies 섹션에.

- <규칙1>
- <규칙2>
- 상태 전이: `<S1> → <S2>`, `<S2> → <S3>` 만 허용

## Policies

상황·시점에 따라 바뀔 수 있는 규칙. 각 Policy마다 구현 위치를 명시.

### <PolicyName>
- **위치**: {Specification | Strategy | Domain Service | Event Handler}
- **이유**: 왜 이 위치인가
- **설명**: 규칙 요약
- **교체 시나리오**: 어떤 상황에서 변할 수 있는가

### <AnotherPolicy>
- **위치**: ...
- **이유**: ...
- **설명**: ...

## External Dependencies

| Aggregate / BC / 외부 시스템 | 참조 방식 | 비고 |
|--------------------------------|-----------|------|
| <Other Aggregate> | ID 참조 (`<otherId>`) | ... |
| <Other BC> | 이벤트 구독 | `<이벤트명>` |
| <외부 시스템> | ACL + 계약 인터페이스 | ... |

## Domain Events (발행)

이 Aggregate가 상태 변경 시 발행하는 이벤트들.

- `<이벤트1>` — 발행 시점, payload 요점
- `<이벤트2>`

## Domain Events (구독)

이 Aggregate의 정책이 반응하는 외부 이벤트.

- `<이벤트1>` — 반응: ...

## Open Questions

설계 중 확정되지 않은 이슈. 비어 있으면 "없음" 명시.

- ⚠️ <질문1>
- ⚠️ <질문2>
```
