# ddd-workshop

막연한 아이디어·PRD·러프한 프롬프트에서 시작해 **도메인 설계(Aggregate)** 까지 이어지는 DDD 파이프라인 스킬 묶음. 도메인 전문가 없이 혼자 또는 소규모 팀에서 체계적으로 도메인을 추출하고 설계할 때 쓰도록 만들어졌습니다.

이 플러그인의 차별점은 **인식론적 정직성(Epistemic Honesty)** 을 구조적으로 강제한다는 점입니다. 에이전트가 낯선 도메인에서도 "그럴듯하게" 답하는 함정을 `domain-research` 스킬이 자가 진단 + 리서치로 차단합니다.

## 파이프라인 한눈에 보기

```
[0] 막연한 아이디어 / PRD
     │
[1]  project-kickoff               ← 맥락·의도·범위 확정
     ↓
[2]  domain-research  ⭐             ← 에이전트 자가 진단 + 도메인 리서치
     ↓
[3]  requirement-clarifier          ← 빈틈·모순 제거
     ↓
[4]  ubiquitous-language-extractor  ← 용어집 + 동의어/동음이의어 감지
     ↓
[5]  event-flow-mapper              ← 혼자 하는 Event Storming
     ↓
[6]  bounded-context-identifier     ← BC 경계 식별 + Context Map
     ↓
[7]  strategic-classifier           ← Core / Supporting / Generic 분류
     ↓
[8]  aggregate-designer             ← Aggregate·불변식·정책 설계 (최종 산출물)
```

각 스킬은 **독립 실행** 가능하며, 필요한 단계부터 **부분 진입**도 허용됩니다. 앞 단계 출력이 있으면 뒤 단계 질문이 날카로워지지만, 없어도 돌아갑니다.

## 어떤 스킬을 언제 쓰나

| 스킬 | 쓰는 때 | 산출물 |
|---|---|---|
| [`project-kickoff`](skills/project-kickoff/SKILL.md) | "~~ 만들고 싶어" 수준의 아이디어를 구조화할 때 | `docs/project-identity.md` |
| [`domain-research`](skills/domain-research/SKILL.md) | 에이전트·사용자 모두 도메인 전문가가 아닐 때 | `docs/domain-brief.md` |
| [`requirement-clarifier`](skills/requirement-clarifier/SKILL.md) | PRD가 있지만 빈틈·모순이 의심될 때 | `docs/requirements-clarified.md` |
| [`ubiquitous-language-extractor`](skills/ubiquitous-language-extractor/SKILL.md) | 요구사항에서 용어집·BC 분리 힌트를 뽑을 때 | `docs/ubiquitous-language.md` |
| [`event-flow-mapper`](skills/event-flow-mapper/SKILL.md) | 시간 순 이벤트·커맨드·정책 매핑이 필요할 때 | `docs/event-flow.md` |
| [`bounded-context-identifier`](skills/bounded-context-identifier/SKILL.md) | BC 경계와 Context Map을 그릴 때 | `docs/context-map.md` |
| [`strategic-classifier`](skills/strategic-classifier/SKILL.md) | 어디에 집중할지(Core) / 외부 솔루션 후보(Generic) 판단 | `docs/strategic-classification.md` |
| [`aggregate-designer`](skills/aggregate-designer/SKILL.md) | BC 내부 Aggregate·불변식·정책을 설계할 때 | `docs/domains/[bc]/[aggregate].md` × N |

## 핵심 설계 원칙

이 플러그인의 모든 스킬이 공통으로 따르는 원칙입니다.

- **맥락 적응성** — 학습/개인/사내/B2B/B2C 맥락별로 질문의 종류와 깊이를 조정.
- **인식론적 정직성** — 아는 것/얕게 아는 것/모르는 것을 명확히 구분. 얼버무림 금지.
- **점진적 심화** — 한 번에 많이 묻지 않고 답변에 따라 다음 층위로.
- **구조화된 출력** — 다음 스킬이 입력으로 받을 수 있는 표준 포맷(프런트매터 + 마크다운).
- **불확실성 명시** — 추론한 내용은 `⚠️` 마커로 표시해 사용자 확인 유도.
- **반복 개선 여지** — 한 번에 완성하지 않고 "1차 초안 → 피드백 → 2차 정제" 루프 내장.
- **강제 질문 프로토콜** — 각 스킬마다 "반드시 물어야 할 체크리스트" 보유. "알아서 해주세요" 금지.

## 스킬 간 연결 규약

모든 스킬 출력 상단에 공통 메타데이터가 박힙니다.

```yaml
---
skill: <스킬명>
version: 0.1.0
context: {학습|개인|사내|B2B|B2C}
confidence: {high|medium|low}
next_suggested: <다음 스킬명>
---
```

`domain-research`의 **도메인 브리프**는 이후 모든 스킬이 참조하는 공유 자원입니다. 각 스킬은 이 브리프에서 자신에게 필요한 힌트("업계 특화 질문", "누락 가능 이벤트" 등)를 꺼내 씁니다.

## 범위 — 무엇을 하고 무엇을 하지 않는가

**포함**: 프로젝트 정체성 → 도메인 리서치 → 요구사항 정제 → Ubiquitous Language → 이벤트 흐름 → Bounded Context → 전략적 분류 → Aggregate 설계.

**제외**: ERD / DB 스키마 / API 설계 / 인프라 / 코드 생성.

> **왜 ERD를 제외하는가**: 도메인 모델과 ERD는 층위가 다릅니다. 저장 전략(일반 ORM / 이벤트 소싱 / CQRS)은 도메인 설계 이후의 별도 결정이며, ERD를 이 파이프라인에 포함하면 "CRUD 마인드셋"으로 도메인 사고가 오염될 위험이 있습니다.

## 설치

### Claude Code (플러그인)

```
/plugin marketplace add dev-goraebap/grimoire
/plugin install ddd-workshop@grimoire
```

### 그 외 에이전트 (`skills.sh`)

```bash
npx skills add dev-goraebap/grimoire --skill project-kickoff --skill domain-research \
  --skill requirement-clarifier --skill ubiquitous-language-extractor \
  --skill event-flow-mapper --skill bounded-context-identifier \
  --skill strategic-classifier --skill aggregate-designer
```

## 사용 예시

### 케이스 A: 새 B2C 아이디어 → Aggregate까지

```
/ddd-workshop:project-kickoff                → 프로젝트 정체성 확정
/ddd-workshop:domain-research                → 자가 진단 + 규제·관행 리서치
/ddd-workshop:requirement-clarifier          → 업계 특화 빈틈 질문
/ddd-workshop:ubiquitous-language-extractor  → 동음이의어로 BC 힌트 확보
/ddd-workshop:event-flow-mapper              → 이벤트 흐름 + 핫스팟
/ddd-workshop:bounded-context-identifier     → BC 후보 + Context Map
/ddd-workshop:strategic-classifier           → Core / Supporting / Generic
/ddd-workshop:aggregate-designer             → Core BC의 Aggregate부터 설계 (반복 호출)
```

### 케이스 B: PRD 보유 + 부분 진입

```
/ddd-workshop:domain-research                → PRD를 입력으로 받음
/ddd-workshop:requirement-clarifier
... 이후 동일 ...
```

## 스킬 상세

각 스킬의 트리거 조건·입력·출력·강제 체크리스트는 링크된 `SKILL.md`를 참조하세요.

## 주의

- **0.x 단계**입니다. 인터페이스·메타데이터 포맷은 1.0 이전까지 바뀔 수 있습니다.
- `domain-research`는 `WebSearch` / `WebFetch` 권한을 필요로 합니다(`allowed-tools`로 명시).
- 의료·법률·세무·금융 도메인에서는 리서치 결과가 **전문가 자문을 대체하지 않습니다**. `domain-research`가 경고 마커로 명시합니다.

## 라이선스

[MIT](../LICENSE).
