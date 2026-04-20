---
name: my-backend-architecture
description: >-
  사용자 개인의 백엔드 아키텍처(레이어: app / use-cases?·features?·contracts? /
  domain / infrastructure / shared)를 Progressive Disclosure로 정리한 지식 스킬.
  FSD의 의존 방향을 백엔드용으로 재구성. app은 resource(API 리소스) 단위, 나머지는
  module 단위이고 이 module은 NestJS @Module()과 다른 개념(특히 domain은 NestJS
  Module 없이 POJO만). 참조 규칙: app·use-cases·contracts·infrastructure의 module
  간 참조 금지(DIP로 해결), domain은 허용하되 BC 기준 분류 시 BC 간 참조는 ID 타입
  외 강한 의존(ORM 관계) 지양, shared는 순환만 피하면 자유. Aggregate 층위,
  Repository 인터페이스-구현 DIP(import 방향은 infrastructure→domain), Domain
  Service vs Application Service 구분, CQRS 경량 적용(읽기는 infra Query Service),
  ORM(TypeORM·Drizzle·Prisma)과 BC 경계에서 ORM 관계 주의, NestJS 경계 강제 수단
  (eslint-plugin-boundaries·dependency-cruiser·Nx boundaries·@nestjs/cqrs·forRoot
  bootstrapping·TS path alias)을 다룬다.
  Triggers — "백엔드 아키텍처", "레이어드 아키텍처", "도메인 레이어 구조",
  "NestJS 모듈 경계", "Domain Service", "Application Service", "Repository 주입",
  "Drizzle 스키마 위치", "contracts 레이어", "Aggregate", "BC 간 참조", "CQRS 읽기",
  "dependency-cruiser".
license: MIT
---

# my-backend-architecture

사용자 개인의 **백엔드 코드 구조 선호**를 정리한 지식 스킬. SKILL.md는 라우팅 테이블 역할만 하고, 실제 내용은 references에 Progressive Disclosure로 분할되어 있다. 사용자 질문 유형에 따라 필요한 파일만 로드해 답한다.

## 핵심 원칙 (요약 5줄)

1. 레이어: `app` → `(use-cases? / features? / contracts?)` → `domain` → `infrastructure` → `shared`. `use-cases`·`features`·`contracts`는 모두 **선택**.
2. 의존은 **단방향**. 위에서 아래로만 참조. 단 `infrastructure`는 `domain`이 정의한 인터페이스를 구현하므로 **import 방향이 역전**되는 경우가 있음(DIP).
3. 폴더 단위 명명: **`app`은 resource**(API 리소스·페이지 리소스), **그 외 레이어는 module**. 이 module은 NestJS `@Module()`과 다른 개념(특히 domain은 NestJS Module 없음).
4. **참조 규칙**: `app`·`use-cases`·`contracts`·`infrastructure`의 module 간 참조 ❌ (DIP로 해결). `domain`은 허용되지만 BC 간 참조는 ID 타입 외 강한 의존(ORM 관계 등) 지양. `shared`는 순환만 피하면 자유.
5. 규칙 판정은 `domain`에, 오케스트레이션·I/O·트랜잭션·이벤트 발행은 `app` / `use-cases`에. Repository는 `use-case` / `app` service에 주입하고 도메인 서비스에는 주입하지 않는다(인자로 전달).

## 의존 다이어그램

```
app (resource)
  ↓
(use-cases / features)?   (module)
  ↓
(contracts)?              (module — 공용 계약 중앙 집중, 선택)
  ↓
domain  ◄───────── infrastructure   ← DIP: infrastructure가 domain 인터페이스 구현
(module)           (module — adapter)
  ↓                    ↓
shared (module — segment)           ← 모든 레이어가 shared 참조 가능

위 → 아래 참조.  domain과 infrastructure 사이만 DIP로 import 방향이 역전.
domain의 module 간 참조는 허용(ORM 관계는 BC 경계 지키기).  shared segment는 순환 금지.
```

## 라우팅 테이블

| 사용자가 묻는 것 | 파일 |
|---|---|
| "이 스킬 뭐야", "왜 존재", 스코프, 클린 아키텍처와의 차이 | [`references/00-overview.md`](references/00-overview.md) |
| "어떤 레이어가 있지", "레이어 책임", "infrastructure 언제 필요", "contracts 언제 쓰지" | [`references/01-layers.md`](references/01-layers.md) |
| "module이 NestJS Module과 뭐가 달라", "resource·module 명명 규칙", "module 간 참조 규칙", "Public API(barrel)" | [`references/02-modules.md`](references/02-modules.md) |
| "컨트롤러 어디에", "app 구조", "오케스트레이션", "CQRS 읽기 경로", "Query Service" | [`references/03-app-layer.md`](references/03-app-layer.md) |
| "도메인 레이어 구조", "Aggregate", "Domain Service vs Application Service", "Repository 어디서 주입", "BC 간 참조", "domain에 뭐가 들어가" | [`references/04-domain-layer.md`](references/04-domain-layer.md) |
| "shared에 뭐가 들어가", "Drizzle 스키마 어디 둬", "마이그레이션 위치" | [`references/05-shared-layer.md`](references/05-shared-layer.md) |
| "TypeORM 도메인 모델", "Drizzle 도메인 모델", "Prisma 리포지토리", "BC 경계에서 ORM 관계 주의" | [`references/06-orm-strategies.md`](references/06-orm-strategies.md) |
| "같은 레이어 기능 재사용", "DIP", "A안 vs B안", "contracts 레이어", "shared에서 도메인 개념 필요" | [`references/07-dip-patterns.md`](references/07-dip-patterns.md) |
| "뭘 테스트해야", "도메인 테스트 범위", "통합 테스트 필요?" | [`references/08-testing.md`](references/08-testing.md) |
| "NestJS 모듈 경계 강제", "eslint-plugin-boundaries", "Spring Boot와 차이", "forRoot bootstrapping", "domain은 NestJS Module 없다" | [`references/09-framework-notes.md`](references/09-framework-notes.md) |

## 자주 묻는 조합

| 상황 | 조합 |
|---|---|
| "NestJS에서 백엔드 폴더 나누는 이유?" | `00` + `01` |
| "같은 레이어 module 참조가 필요해" | `02` + `07` |
| "Drizzle 쓰는데 도메인 모델·스키마 어디" | `06` + `04` + `05` |
| "NestJS 모듈이 경계 뚫음" | `09` + `02` |
| "도메인 레이어 테스트 범위" | `08` + `04` |
| "Domain Service가 Repository를 직접 호출해도 돼?" | `04` (Domain Service 섹션) |
| "BC 간 참조가 필요한데 ORM 관계로 묶어도 되나?" | `04` + `06` |
| "contracts 레이어는 언제 도입?" | `01` + `07` |

## 원칙

- **완벽한 클린 아키텍처 지향 아님**. 도메인이 `shared`를 참조하는 건 허용. 이유는 `00-overview.md`의 "Not Clean Architecture".
- **프레임워크 강제 없음**. NestJS가 주 예시지만 원리는 일반적. Spring Boot 등 설정 응집형 DI 환경과의 차이는 `09-framework-notes.md`.
- **테스트 1순위는 도메인 정책·비즈니스 규칙**. 나머지는 선택.
- **"module"이라는 단어를 NestJS `@Module()`과 구분**한다. 이 스킬의 module은 폴더 단위 코드 조직 경계이고, NestJS Module은 DI 런타임 구성 단위다. 특히 **`domain`은 NestJS Module이 없음** — 순수 코드만 있는 module이다.
