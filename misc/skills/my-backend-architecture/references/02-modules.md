---
section: modules
---

# Modules

레이어 하위의 폴더 단위를 이 스킬은 **module**이라 부른다. 단, `app`만 예외로 **resource**라 부른다. 이 스킬의 "module"은 **NestJS `@Module()`과 다른 개념**이므로 이 파일 앞부분에서 구분부터 명확히 한다.

## 이 스킬의 "module" vs NestJS `@Module()`

| | 이 스킬의 module | NestJS `@Module()` |
|---|------------------|-------------------|
| 의미 | 레이어 내 **폴더 단위 코드 조직 경계** | **DI 런타임** 구성 단위 |
| 경계 표현 | Public API (`index.ts` barrel 등) | `imports` / `providers` / `exports` |
| 항상 1:1 대응? | 레이어별로 다름 (아래) | — |
| 필수? | 파일 조직 원칙이라 어느 레이어든 존재 | NestJS 쓸 때만, 레이어별로 다름 |

### 레이어별 NestJS Module 존재 여부

| 레이어 | NestJS Module | 설명 |
|--------|---------------|------|
| `app` | ✅ 있음 | 대체로 resource 1:1 대응 (`EmployeesModule`) |
| `use-cases` / `features` | △ 선택 | Module로 만들거나 그냥 주입 가능 클래스 |
| `contracts` | ❌ 없음 | 인터페이스·토큰만 담은 파일 — DI 등록은 바인딩하는 쪽에서 |
| **`domain`** | ❌ **없음** | 순수 코드 (POJO·클래스·함수·인터페이스)만. DI 배선은 상위 레이어가 맡음 |
| `infrastructure` | ✅ 대체로 | adapter별 Module 또는 여러 adapter를 묶은 Module |
| `shared` | △ 일부 | `LoggerModule`·`ConfigModule` 등. 유틸은 Module 없는 함수·클래스 |

**`domain`이 NestJS Module을 가지지 않는 것**이 가장 주의할 점이다. 도메인 코드는 프레임워크 의존 없이 테스트 가능해야 하고, NestJS DI 컨테이너 배선은 infrastructure와 app 레이어에서 수행한다.

다른 DI 프레임워크(Spring Boot, .NET 등)를 써도 이 원칙은 동일 — 도메인은 프레임워크 의존 없이 순수 유지, DI 배선은 상위·하위 레이어가 맡는다.

## 명명 요약

| 레이어 | 폴더 단위 이름 | 의미상 별칭 |
|--------|----------------|------------|
| `app` | **resource** | API 리소스·페이지 리소스·CLI 커맨드 |
| `use-cases` / `features` | **module** | use-case / feature |
| `contracts` | **module** | contract |
| `domain` | **module** | context (BC 단위) 또는 aggregate (Aggregate 단위) |
| `infrastructure` | **module** | adapter |
| `shared` | **module** | segment |

규칙 표현은 **"module"**이라는 공통 용어로 쓰되, 레이어별 의미상 별칭을 필요할 때 괄호로 병기한다.

## 레이어별 module 간 참조 규칙

같은 레이어의 module이 서로 `import`하는 것을 허용하는지.

| 레이어 | module 간 참조 | 해법 |
|--------|----------------|------|
| `app` | ❌ resource끼리 참조 금지 | DIP로 해결 — [`07-dip-patterns.md`](07-dip-patterns.md) |
| `use-cases` / `features` | ❌ | DIP |
| `contracts` | ❌ (참조할 이유도 드묾) | 계약끼리 합성할 일이 생기면 상위 레이어에서 조합 |
| `infrastructure` | ❌ adapter끼리 참조 금지 | DIP |
| **`domain`** | ⚠️ **허용하되 검토** | BC 단위로 분류했다면 BC 간 참조는 ID 타입 외 지양 (아래 §domain 참조 규칙) |
| `shared` | ✅ segment끼리 참조 가능 | 순환만 금지 |

### 왜 대부분 금지인가

- 같은 레이어 module끼리 의존이 생기면 **의존 그래프가 평면 네트워크**가 되어 영향 범위 예측이 어려워진다.
- module 단위가 테스트·이해·리팩터링의 단위로 유지되어야 레이어 구조의 가치가 살아난다.
- 금지 규칙이 깨지기 시작하면 다른 규칙(레이어 단방향 등)도 **연쇄적으로 뚫린다**. 첫 위반이 가장 비싸다.

### 어떻게 피하나

module 간 기능이 엮이면 **DIP로 푼다**. 두 방향:

- **A안**: 공통 기능을 상위 레이어(`use-cases` / `features`)로 승격.
- **B안**: 계약 인터페이스를 만들고 원래 자리는 구현체로 유지. 다른 module은 계약만 참조 (계약 위치는 같은 레이어 안 또는 `contracts/` 중립 레이어).

선택 기준·코드 예시는 [`07-dip-patterns.md`](07-dip-patterns.md).

## domain module 간 참조 규칙 (예외)

`domain`은 **유일하게 module 간 참조를 허용**하는 레이어다 (`shared` 제외). 다만 다음 두 가지 유의사항이 있다.

### 1. 단일 BC이면 자유, 다중 BC면 검토

- **단일 Bounded Context**로 운영 (module이 Aggregate 단위) → 같은 BC 안이라 module끼리 참조 자유.
- **다중 Bounded Context**로 운영 (module이 BC 단위) → BC 간 참조는 정말 필요한지 검토. 대부분 ID 타입 참조로 충분하다.

### 2. ORM 관계를 BC 경계 넘어 맺지 말 것

같은 BC 안 Aggregate 간에도 Aggregate 경계를 넘는 ORM 관계(`@ManyToOne` 등)는 조심해야 하고, **BC 경계를 넘는 ORM 관계는 사실상 피해야 한다**.

- **TypeORM**: `@ManyToOne(() => Employee)`로 다른 BC의 Employee와 관계를 맺으면 cascade·eager loading이 BC 경계를 넘어 엮임. `@Column('uuid') employeeId: string`만 써라.
- **Drizzle**: `relations()` 선언을 BC 넘어 쓰지 말고, 조인이 필요하면 `infrastructure/queries/`에서.
- **Prisma**: `@relation`을 BC 넘어 쓰면 마이그레이션·인덱스가 꼬임. `String` FK 컬럼만.

상세: [`06-orm-strategies.md`](06-orm-strategies.md).

### 3. 조인이 꼭 필요하면 Query Service로

화면 조회에서 여러 BC 데이터가 필요하면 **CQRS 경량 적용** — `infrastructure/queries/`의 Query Service가 테이블 조인으로 화면 DTO를 바로 반환. domain은 우회한다. 상세: [`03-app-layer.md`](03-app-layer.md).

## shared segment 상호 참조 — 순환 금지

`shared`는 `domain`과 함께 **module 간 참조가 자유로운** 레이어. 단 순환은 피한다.

```
shared/logger   ──▶ shared/config    (OK)
shared/db       ──▶ shared/logger    (OK)

shared/a ──▶ shared/b ──▶ shared/a   (순환 금지)
```

이유: `shared`는 도메인 무관 기술 세그먼트라 의존이 엮이기 자연스럽다. 순환이 나타나면 한쪽을 더 작게 쪼개거나 흡수.

## Public API — barrel export (프레임워크 의존)

각 module은 **외부에 공개할 것만 Public API로 명시**한다. 내부 구조는 자유 구성.

### TypeScript 환경 — barrel(`index.ts`)

```ts
// domain/workforce/index.ts  (Public API)
export { Employee } from './employee/employee.entity';
export { Organization } from './organization/organization.entity';
export type { EmployeeRepository } from './employee/employee.repository';
export { EMPLOYEE_REPOSITORY } from './tokens';
// 정책 내부 함수, 보조 VO 등은 export 안 함 — private
```

외부는 반드시 barrel을 거친다:

```ts
// OK
import { Employee } from '@domain/workforce';

// 금지 — 내부 파일 직접 접근
import { Employee } from '@domain/workforce/employee/employee.entity';
```

**같은 module 내부에서는** barrel을 거치지 않고 상대 경로 import OK — 내부 자유 구성 원칙.

### 지원하지 않는 프레임워크·언어

barrel 개념을 지원하지 않거나 관용이 다른 환경에서는 **억지로 쓰지 않는다**. 핵심은 "외부 노출 대상을 의도적으로 명시"라는 원칙이고, 각 환경의 관용에 맞추면 된다.

- Java: package-private / `module-info.java` 경계
- Python: `__init__.py`에서 re-export 또는 `__all__`
- Go: 대문자/소문자 네이밍으로 export 경계

이 스킬은 TypeScript·NestJS 환경을 주로 가정하므로 설명은 barrel 기준이지만, 철학은 범용이다.

### 정적 강제 수단

barrel 관례는 개발자 규율에 맡기면 깨지기 쉽다. 정적 강제 도구:

- `eslint-plugin-boundaries` — 외부 module은 barrel만 import하도록 규칙
- `dependency-cruiser` — 경로 패턴 위반 감지
- TypeScript `paths` alias — `@domain/workforce/*` 금지, `@domain/workforce`만 허용

상세: [`09-framework-notes.md`](09-framework-notes.md).

## module 내부 구조 — 자유

Public API만 지키면 **내부는 어떻게 나누든 무방**.

### app resource 예 (NestJS)

```
app/employees/
├── index.ts                  ← public: EmployeesModule
├── employees.module.ts
├── employees.controller.ts
├── employees.service.ts
├── dto/
└── guards/
    └── owner.guard.ts        ← resource 전용
```

### domain module 예 (BC 단위 — 다중 BC)

```
domain/workforce/
├── index.ts                  ← public: Entity·Repository·Token 등
├── employee/                 ← Aggregate 그룹
│   ├── employee.entity.ts
│   ├── employee.repository.ts  (interface)
│   └── employee-status.vo.ts
├── organization/
├── position/
├── rank/
│   └── rank.vo.ts
├── policies/                 ← 여러 Aggregate 걸친 도메인 서비스
└── events/
```

### domain module 예 (Aggregate 단위 — 단일 BC)

```
domain/employee/
├── index.ts                  ← public
├── employee.entity.ts
├── employee.repository.ts    (interface)
├── employee-status.vo.ts
└── events/
```

**원칙**:
- module 내부에서만 쓰는 helper는 내부에 유지 + barrel에서 export 안 함
- 공통 helper는 `shared`로 승격
- 내부 파일이 많아져 거슬리면 **module 쪼개기** 신호

## 체크리스트

- [ ] 같은 레이어 module 간 import 있는가? (`domain`·`shared` 제외) → DIP로 해결
- [ ] 각 module에 Public API(`index.ts`)가 있는가? (TypeScript 환경)
- [ ] 외부 module 경로를 내부 파일까지 직접 import하지 않는가?
- [ ] domain에서 다른 BC의 entity를 ORM 관계로 맺고 있지 않은가? → ID 타입으로 바꿔라
- [ ] shared segment 간 순환 있는가? → 리팩터 필요
- [ ] 레이어 역방향 import 없는가? (`domain → app` 등)
- [ ] "domain에 NestJS `@Module()`을 만들어 두지 않았는가?" → domain은 순수 코드만
- [ ] `domain/models/`, `domain/repositories/` 같은 기술 유형별 분류에 빠졌나? → Aggregate 또는 BC 단위로 재구성 ([`04-domain-layer.md`](04-domain-layer.md))
