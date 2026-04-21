---
name: strategic-classifier
description: >
  ddd-workshop 파이프라인의 전략적 분류 스킬. 각 Bounded Context를
  Core / Supporting / Generic으로 분류하고, 분류별 투자·구현 전략을 제안한다.
  Core 판정은 project-kickoff의 Why와 domain-research의 경쟁 차별점을 교차
  참조한다. 산출물은 분류표 + 구현 권고(외부 솔루션/표준 패턴/DDD 전술 패턴).
  "core supporting generic", "전략적 분류", "Core Domain 식별",
  "어디에 집중할까", "외부 솔루션 쓸지", "ddd-workshop 6단계" 같은 요청에
  트리거한다.
metadata:
  author: dev-goraebap
  version: "0.1.0"
---

# strategic-classifier 스킬

모든 BC에 똑같이 힘을 쏟으면 프로젝트는 망한다. DDD의 **전략적 설계**는 "어디에 최고의 개발자와 시간을 투입하고, 어디는 외부 솔루션으로 갈음할지"를 정한다.

이 스킬은 `bounded-context-identifier`가 그린 Context Map의 각 BC를 **Core / Supporting / Generic**으로 분류하고, 분류별 실행 권고(투자·패턴·대체 솔루션)를 제안한다.

---

## 1. 언제 트리거하는가

- `/ddd-workshop:strategic-classifier`
- `bounded-context-identifier` 출력 직후
- "Core Domain 뭐야?", "어디에 집중해야 해?", "외부 솔루션 쓸지 말지"

---

## 2. 입력

필수:
- `bounded-context-identifier` 출력 — 분류 대상 BC 목록

강력 권장:
- `project-kickoff` 출력의 **Why · 핵심 가치** — Core 판정의 1순위 근거
- `domain-research` 출력의 **경쟁 서비스 관행** — 차별화 포인트 판단

---

## 3. 분류 기준

### Core Domain
- **정의**: 이 프로젝트의 경쟁력 원천. Why와 직결.
- **판정 기준**:
  - `project-kickoff`의 Why에 직접 언급되었는가?
  - `domain-research`의 경쟁 차별점이 이 BC에 집약되는가?
  - 이 BC가 없으면 프로젝트의 의미가 사라지는가?
- **권고**: 최고 역량 투입, DDD 전술 패턴(Aggregate, Specification, Domain Service) 꼼꼼히 적용, 절대 외주·외부 솔루션 금지.

### Supporting Domain
- **정의**: 필요하지만 차별화 포인트는 아님. 자체 개발하되 표준 패턴으로.
- **판정 기준**:
  - 경쟁사도 다 가진 기능이지만, 도메인 지식이 필요해 외부 솔루션으로 완전 대체는 어려움.
  - 이 영역의 버그는 Core를 직접 위협하진 않지만 가치 전달에 영향.
- **권고**: 표준 CRUD·패턴, 사내 개발, 코드 리뷰는 Core보다 느슨하게.

### Generic Domain
- **정의**: 어디나 있는 일반 기능. 외부 솔루션 대체가 합리적.
- **판정 기준**:
  - 이 기능을 직접 구현하는 것이 경쟁력에 기여하지 않음.
  - 시장에 검증된 솔루션이 존재.
- **권고**: **외부 솔루션 우선 검토** (결제→PG, 인증→Auth0/Keycloak, 이메일→SendGrid, 검색→Algolia 등). 자체 개발은 비용·규제상 필요한 경우에만.

---

## 4. 진행 단계

### Step 1 — 근거 수집

각 BC에 대해 아래를 수집.

| 근거 | 출처 | 어떻게 쓰는가 |
|------|------|---------------|
| Why·가치 | project-kickoff | Core 직결 여부 |
| 경쟁 차별점 | domain-research | 차별화 집약 위치 |
| 업계 관행 | domain-research | Generic 여부(업계가 외부 솔루션 쓰는지) |
| 규제·보안 | domain-research, project-kickoff | Generic이어도 자체 개발해야 하는지 |

### Step 2 — 1차 분류

각 BC를 Core/Supporting/Generic 중 하나로 분류. 논쟁적인 BC는 **두 후보와 근거 모두** 기록.

### Step 3 — Core 검증 (중요)

Core는 많아봐야 1~2개여야 한다. 3개 이상이면 재검토 필요. 과다 Core는 "사실 Supporting인데 애착으로 Core라 부르는" 경우가 많다.

재검토 질문:
- "이 BC가 없다면 프로젝트의 본질이 바뀌는가?"
- "이 BC를 경쟁사가 더 잘 만들면 우리 사업이 위협받는가?"

### Step 4 — Generic 검증 (중요)

Generic으로 분류한 BC에 대해 외부 솔루션 후보를 제시. `domain-research`의 "주요 플레이어 및 관행"을 참고.

규제·보안·비용 이유로 외부 솔루션을 못 쓰는 경우 이유를 명시.

### Step 5 — 분류표와 권고 작성

각 BC에 대해 분류 + 구현 권고를 표로 정리. 사용자 확인 후 출력.

출력: `docs/strategic-classification.md`.

---

## 5. 강제 질문 체크리스트

```
□ Core 후보에 project-kickoff Why 인용됨
□ Core 후보에 domain-research 경쟁 차별점 인용됨
□ Core 후보 수가 1~2개 범위
□ Supporting 각 항목에 "왜 Core가 아닌가" 이유 있음
□ Generic 각 항목에 외부 솔루션 후보 최소 1개 제시됨
□ 모든 BC가 분류됨 (누락 없음)
□ 분류별 투자·구현 권고 포함됨
```

---

## 6. 출력 포맷

```markdown
---
skill: strategic-classifier
version: 0.1.0
context: {학습|개인|사내|B2B|B2C}
confidence: {high|medium|low}
next_suggested: aggregate-designer
created_at: YYYY-MM-DD
---

# 전략적 분류

## 분류 요약

| BC | 분류 | 핵심 근거 | 구현 권고 |
|----|------|-----------|-----------|
| 거래(매칭·평가) | **Core** | Why "지역 기반 신뢰" 집약, 경쟁사 대비 신뢰지표 차별화 | 최고 역량, DDD 전술 패턴 |
| 카탈로그 | Supporting | 차별화 아니지만 도메인 특수성 있음 | 표준 CRUD, 검색은 외부 검토 |
| 분쟁 조정 | Supporting | 사용자 경험에 영향 크지만 차별화는 아님 | 상태머신 설계 주의 |
| 결제·에스크로 | Generic | 업계가 모두 PG 연동 | 외부 PG (토스페이먼츠, 포트원) |
| 알림 | Generic | 범용 기능 | 외부 (SendGrid, NHN Cloud) |
| 인증 | Generic | 범용 기능 | 외부 (소셜 로그인 + Auth 서비스) |

---

## Core Domain

### 거래 (Trade Context)
- **Why 근거**: project-kickoff § Why — "지역 기반 신뢰를 재료로 마찰을 낮춤"
- **경쟁 차별점**: domain-research § 경쟁 플레이어 — 당근마켓의 매너 온도 대비 "구체적 거래 이력 신뢰지표" 차별화.
- **권고**:
  - 최고 역량 개발자 투입
  - Aggregate 설계 시 불변식·정책 철저히 분리
  - 코드 리뷰 엄격
  - 절대 외주·외부 솔루션 대체 금지
  - 이 영역의 변경 이력은 ADR로 남김

---

## Supporting Domains

### 카탈로그 (Catalog Context)
- **왜 Core가 아닌가**: Why와 직결되지 않음. 경쟁사 공통 영역.
- **구현 권고**:
  - 표준 CRUD 패턴
  - 검색은 성장 단계에서 Elasticsearch·Meilisearch 도입 검토
  - 이미지 저장은 외부 CDN

### 분쟁 조정 (Dispute Context)
- **왜 Core가 아닌가**: 사용자 신뢰에 영향 크지만, 차별점은 "거래" BC가 담당.
- **구현 권고**:
  - 상태 전이 설계 정교하게 (상태 머신)
  - 운영 업무 흐름이 중요 → 어드민 UX 투자

---

## Generic Domains

### 결제·에스크로 (Payment Context)
- **외부 솔루션 후보**:
  - 토스페이먼츠, 포트원(아임포트), KG이니시스
- **자체 구현이 필요한 경우**: 에스크로 특수 규제 대응 시에만 (v1 범위에서 외부로 위임 권장)
- **권고**: ACL을 통해 외부 결제 모델이 우리 도메인 모델을 오염시키지 않도록 격리.

### 알림 (Notification Context)
- **외부 솔루션 후보**: SendGrid, NHN Cloud Notification, AWS SES/SNS
- **권고**: 이벤트 기반 발신 추상화 레이어만 내부에 두고 전송은 외부.

### 인증 (Identity Context)
- **외부 솔루션 후보**: Auth0, Keycloak, AWS Cognito, 소셜 로그인 + NextAuth
- **규제 고려**: 개인정보 최소 수집 원칙, 위치 정보는 별도 동의 플로우 필요(자체 구현 영역).

---

## ⚠️ 경계 사례

### 재고 (Inventory Context)
- **논쟁점**: Supporting vs Core.
- **논거 A (Supporting)**: 업계 일반 기능.
- **논거 B (Core 가능성)**: 지역 직거래 특성상 "재고 1개 = 물리적 1개"의 동시성 관리가 차별화 근거가 될 수 있음.
- **권고**: 현재는 Supporting으로 두되, 사기·중복 거래 이슈가 Core 수준으로 부상하면 재분류.

---

## 다음 단계
→ `aggregate-designer`:
  - **Core BC 먼저** Aggregate 설계 (투자 우선순위)
  - Supporting은 표준 패턴으로 축약 가능
  - Generic은 Aggregate 설계 생략, 외부 솔루션 연동 인터페이스만 설계
```

---

## 7. 맥락별 조정

| 맥락 | Core 수 | Generic 외부 솔루션 제안 |
|------|---------|---------------------------|
| 학습 | 전체 Core로 봐도 OK | 학습 목적이면 전부 자체 구현 |
| 개인 | 1개 | 외부 서비스 적극 활용 |
| 사내 | 1~2개 | 사내 솔루션 정책 따름 |
| B2B | 1~2개 | 고객 보안 요구 따라 자체 구현 강제될 수도 |
| B2C | 1~2개 | 외부 솔루션 적극 활용 |

---

## 8. 불확실성 표기

- Core 근거에 project-kickoff Why가 **인용되지 않았다면** confidence를 낮춘다.
- 업계 경쟁 정보가 low confidence이면 Core 판정에도 ⚠️ 표시.
- 분류가 정말 50/50인 BC는 "경계 사례" 섹션에 별도 명시.

---

## 9. 다음 스킬로의 연결

```
→ `aggregate-designer`
  - 권장 순서: Core → Supporting → (Generic은 인터페이스만)
  - 각 BC 1개씩 호출 반복
```

---

## 10. 절대 하지 말 것

- 모든 BC를 Core로 분류 (전략적 집중의 반대).
- Why·경쟁 근거 없이 "이게 Core 같아" 감으로 정하기.
- Generic에 외부 솔루션 후보 없이 "알아서 붙이세요" 마치기.
- 사내 맥락에서 "사내 SSO 써라" 같은 일반론으로 넘기기 (사내 정책·SSO 실제 종류 확인).
- 경쟁 차별점 인용 없이 Core 판정.
