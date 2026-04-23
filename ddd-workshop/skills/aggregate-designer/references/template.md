# Aggregate 문서 템플릿 (v0.3)

`docs/shared/contexts/<bc-slug>/<aggregate-slug>.md` 로 복제해 채운다.

**YAML frontmatter 없음.** 필요 시 본문 최상단 뱃지 1줄만.

---

```markdown
# <RootName>

> **Status**: {draft | active | deprecated}

## Role

### 책임지는 것
- ...

### 책임지지 않는 것
- ... (→ 다른 BC 명시. 예: "환불은 Payment BC 소관")

## Ubiquitous Language (이 Aggregate 범위)

BC Canvas의 per-BC UL을 이어받아 Aggregate 세부 용어 추가.
이중 표기 강제: 용어(사용자 언어) | Code Identifier | 의미

| 용어 | Code Identifier | 의미 |
|---|---|---|
| <개념1> | <CodeId1> | ... |
| <개념2> | <CodeId2> | ... |

### 동음이의어·범위 주의 (필요 시)
- "<단어>"는 이 BC에서 <의미>. 다른 BC의 동음이의어와 구분.

## Event Storming (Design-Level)

### 관련 플로우

```
[액터] →(커맨드)→ [이벤트됨]
  ↓ 정책: ...
  (다음커맨드) → [다음이벤트됨] or [실패됨]
```

### 해소된 핫스팟
- <핫스팟>: <결정>

### 미해소 핫스팟
- ⚠️ <핫스팟>

## Structure

- **Root**: <RootName>
- **내부 Entity**: <Entity1>, <Entity2> — Root 없이 존재 불가
- **VO**: <VO1>, <VO2> — 값으로만 의미
- **외부 참조 (ID only)**: `<otherAggId>: <Type>` (해당 BC 명시)

## Invariants

"항상 참"인 규칙만.

1. <규칙1>
2. <규칙2>
3. 상태 전이: `<S1> → <S2>`, `<S2> → <S3>` 만 허용

## Policies

### <PolicyName>
- **위치**: {Specification | Strategy | Domain Service | Event Handler}
- **이유**: 왜 이 위치인가
- **설명**: 규칙 요약
- **교체 시나리오**: 어떤 상황에서 바뀔 수 있는가

## External Dependencies

| 대상 | 관계 | 참조 방식 | 비고 |
|---|---|---|---|
| <Other Aggregate> | 동일 BC | ID 참조 | ... |
| <Other BC> | Customer-Supplier | 이벤트 구독 | `<이벤트명>` |
| <외부 시스템> | ACL | 어댑터 인터페이스 | ... |

## Domain Events (발행)

- `<이벤트1>` — 발행 시점, payload 요점
- `<이벤트2>` — ...

## Domain Events (구독)

- `<이벤트1>` (from <BC>) — 반응: ...

## Exposed Queries

이 Aggregate가 외부에 제공하는 **읽기 계약**. 구현이 아니라 시그니처.

| Query | 반환 | 원천 | 주의 |
|---|---|---|---|
| `<queryName>(args)` | `<ReturnShape>` | Query Service | ... |
| `<anotherQuery>(args)` | `<ReturnShape>` | Projection | ... |

**Read-side 규약**:
- 위 Query 외의 읽기 경로 사용 금지
- `Repository.findX().toDTO()` 패턴 금지 (Aggregate 재조립 금지)
- Projection 갱신은 이벤트 구독으로

**screen-inventory 매칭** (있는 경우):
- ✅ `<queryName>` → `<screen-id / screen-name>`
- ⚠️ <mismatch-항목 있으면 기재>

## Open Questions

- ⚠️ <질문1>
- (비어 있으면 "없음" 명시)
```
