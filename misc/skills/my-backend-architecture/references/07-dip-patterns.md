---
section: dip-patterns
---

# DIP Patterns

같은 레이어 module 간 참조가 필요해질 때의 해결 기법. 두 방향(A안·B안)을 **동등하게** 소개하고 상황별 선택 기준을 제시한다.

## 시나리오

### 시나리오 A — app resource 간 (API 리소스끼리)

`app/employees`의 퇴사 처리 기능이 `app/payroll`의 급여 정지 기능을 필요로 한다.

```
app/employees/employees.service.ts
  └─ 퇴사 처리 시 → app/payroll/payroll.service.ts 의 stopPayroll() 호출하고 싶음
```

[`02-modules.md`](02-modules.md) 규칙상 같은 레이어 참조는 금지. 해결책 두 가지 (아래 A안/B안).

### 시나리오 B — domain Bounded Context 간

BC 단위 domain(다중 Bounded Context)에서 `domain/payroll`이 `domain/organization`의 Employee 정보를 필요로 한다.

```
domain/payroll/payroll.service.ts
  └─ 급여 계산 시 → domain/organization 의 Employee 를 참조하고 싶음 (금지)
```

이 경우 아래 A/B에 더해 **Context 특유의 네 가지 기법**이 기본 옵션으로 제공된다:

- **ID만 소유** — `employeeId: string`만 저장.
- **스냅샷 VO** — `PayableEmployee` 같은 VO로 필요한 필드만 자기 Context에 복제. 갱신은 이벤트 구독.
- **Anti-corruption 계약** — `EmployeeLookup` 인터페이스를 payroll 안에 정의, 구현체는 infrastructure에서 organization 참조.
- **도메인 이벤트 구독** — `EmployeeTerminated` 발행 → payroll이 구독.

자세한 Context 협력 패턴은 [`04-domain-layer.md`](04-domain-layer.md)의 "Context 간 협력". 아래 A/B는 **직접 호출이 꼭 필요한 상황**에 적용.

## A안: 상위 레이어로 승격

공통 기능을 `use-cases` (또는 `features`) 레이어로 승격.

### 구조

```
use-cases/
└── offboard-employee/
    └── offboard-employee.use-case.ts   ← orchestrator

app/employees/employees.service.ts ──▶ use-cases/offboard-employee
app/payroll/payroll.service.ts는 자기 도메인 처리만 담당
```

### 장단

장점:

- 구조가 선명하다. "이 기능은 여러 도메인을 엮는다"는 의도가 이름에 담김.
- 재사용성 명확 (다른 app module도 같은 use-case 호출 가능).

단점:

- `use-cases` 레이어 자체가 도입돼 레이어 수 증가.
- 단발성 조합에 쓰면 오버엔지니어링.

### 적합한 경우

- 같은 조합이 **2곳 이상에서 쓰이거나 쓰일 것** 같을 때.
- 조합의 로직이 **여러 단계**(트랜잭션·이벤트 발행·보상 로직 등)일 때.
- 이미 프로젝트에 `use-cases` 또는 `features` 레이어가 존재할 때.

## B안: 계약(인터페이스) 분리

원 module가 계약을 export하고, 타 module는 계약만 참조. 구현체는 자리 유지.

### 구조

```
app/payroll/
├── index.ts                       (barrel — PAYROLL_STOPPER 토큰, PayrollStopper 인터페이스 export)
├── payroll.service.ts             (PayrollService — 원래 구현체)
└── payroll-stopper.contract.ts    (export interface PayrollStopper)

app/employees/employees.service.ts ──▶ PayrollStopper (인터페이스)
                                         │
                                         └─ 런타임에 PayrollService를 주입 (DI)
```

### 런타임 구성 (NestJS 기준)

- `app/payroll/payroll.module.ts`: `{ provide: PAYROLL_STOPPER, useClass: PayrollService }` + exports에 `PAYROLL_STOPPER` 포함.
- `app/employees/employees.module.ts`: `imports: [PayrollModule]`, 서비스에서 `@Inject(PAYROLL_STOPPER)` 주입.

**주의**: 모듈 파일에 `PostsModule` import가 남는다. 엄격한 import 경계로 보면 "같은 레이어 참조"가 **여전히 존재**. 이 문제와 해법은 [`09-framework-notes.md`](09-framework-notes.md).

### 장단

장점:

- 새 레이어 도입 최소 (아래 B-2 변형은 레이어 1개 추가).
- 의존 방향이 **추상에 대한 의존**으로 역전 (DIP의 전형).

단점:

- 계약을 어디 두느냐로 어색함이 생길 수 있음 → 아래 B-1 / B-2 선택.
- NestJS 모듈 시스템에서 module 파일에 구체 import가 여전히 남는 경우가 있음 — 상세·해결은 [`09-framework-notes.md`](09-framework-notes.md).

### B-1 — 계약을 원 module에 둠

위 구조 예시가 B-1. 계약 파일(`payroll-stopper.contract.ts`)을 `app/payroll/` 안에 두고 barrel로 export. 소비자(`app/employees`)가 `app/payroll` module을 import한 뒤 계약만 바라봄.

**적합**: 계약이 한두 개, 소비자가 소수(1~2개 module).
**한계**: 소비자 module이 제공자 module을 실제로 import하므로 정적 경계 분석기에서는 "같은 레이어 참조"로 감지됨. 소비자가 늘거나 계약 수가 늘면 의존 그래프가 복잡.

### B-2 — 계약을 `contracts/` 레이어로 중앙 집중

계약을 **별도 레이어**(`contracts/`)로 올려 중앙 집중. `app` 양쪽 module은 `contracts/`만 바라봄.

```
contracts/                         ← 선택 레이어 (01-layers 참조)
└── payroll-stopper/
    ├── index.ts                   (barrel — 인터페이스와 토큰만 export)
    └── payroll-stopper.contract.ts

app/payroll/
├── index.ts
├── payroll.module.ts              ← contracts 구현 등록:
│                                    { provide: PAYROLL_STOPPER, useClass: PayrollService }
└── payroll.service.ts             implements PayrollStopper

app/employees/
├── index.ts
├── employees.module.ts            ← contracts 참조만. app/payroll import 없음.
└── employees.service.ts           @Inject(PAYROLL_STOPPER)
```

**적합**:

- 소비자가 여러 module에 걸침 (Notifier·Logger 같이 광범위 공용 계약)
- app 레이어에서 "같은 레이어 module 참조 금지"를 정적 분석기로 엄격히 강제하고 싶을 때
- 여러 BC·여러 use-case가 공유하는 계약

**한계**:

- 레이어 1개 추가 — 프로젝트가 작으면 과함
- 계약 자체에도 관리 비용 (이름·토큰·버전)

### B-1 vs B-2 선택

| 조건 | 선택 |
|------|------|
| 계약 1~2개 + 소비자 1~2개 | **B-1** (원 module에 둠) |
| 계약 여러 개 또는 소비자 다수 | **B-2** (contracts 레이어로 중앙 집중) |
| app module 간 import를 완전히 차단하고 싶음 | **B-2** |
| 레이어 수 최소화가 우선 | **B-1** |

계약끼리 참조하는 일은 거의 없으니 `contracts/` 안에서 module 간 참조 규칙은 사실상 "참조 없음"으로 유지.

### 적합한 경우 (B안 전체)

- 레이어 수 최소화 (B-1) 또는 정적 경계 강제 (B-2)
- 두 module 사이 결합이 **약한 API 한두 개**에 그칠 때 (B-1) / 여러 곳에서 공유 (B-2)
- CQRS·EventBus 같은 런타임 수신자 은닉을 적용하기 전 중간 단계

## A안 vs B안 선택 기준

| 질문 | A안 유력 | B안 유력 |
|---|---|---|
| 조합이 복잡한가 (다단계·트랜잭션)? | ✔ | |
| 조합이 재사용되는가 (2곳 이상)? | ✔ | |
| 레이어 수 증가를 피하고 싶은가? | | ✔ |
| 양쪽이 **대등한 도메인**이라 상하 관계가 어색한가? | | ✔ |
| 이미 `use-cases` 레이어가 있는가? | ✔ | 상관없음 |

**동시에 선택 가능**. `use-cases` 레이어 안에 계약 분리를 써도 된다. 단일 프로젝트 내에서 **일관성 유지**가 핵심.

## 계약 이름 규약

B안에서 인터페이스 이름은 **동사 중심**으로 — "what it does" 관점.

- Good: `PostRemover`, `UserSuspender`, `OrderApprover`
- Bad: `IPostsService`, `PostsServiceInterface` (구현체에 종속)

인터페이스 이름이 도메인의 업무 용어(Ubiquitous Language)를 따르면 팀 전체가 같은 어휘를 쓴다.

## shared → domain 역참조 회피

**`shared`가 도메인 개념을 필요로 하는 듯 보일 때** 같은 DIP 기법을 적용.

### 예시

`shared/audit-logger`가 로그에 사용자 이름을 포함하고 싶다 → `domain/users` 참조 유혹.

### 해결

```
shared/audit-logger/
├── audit-logger.ts
└── user-context.contract.ts    (export interface UserContextProvider)

app/<어느 상위 레이어>/
└── app-user-context.provider.ts   (implements UserContextProvider, domain/users 참조)

bootstrap 시:
  { provide: USER_CONTEXT_PROVIDER, useClass: AppUserContextProvider }
```

- `shared`는 **계약만** 정의.
- 계약을 만족하는 **구현체를 상위 레이어**(app)에서 제공.
- shared는 구현체를 모른 채 계약 메서드만 호출.

### 이 패턴의 의미

DIP의 정확한 적용: **"고수준 모듈이 저수준 모듈에 의존하지 않고, 둘 다 추상에 의존한다."** 여기서는 고·저수준이 레이어 위치와 반대일 수 있다 — 역참조 회피 맥락에서는 **위치가 아닌 추상화 수준**이 기준.

## 단순 중복은 허용

매번 A/B 기법을 꺼내기보다 **중복을 수용하는 게 나은 경우**도 많다.

- 두 module의 로직이 겹쳐 보이지만 **도메인 의미가 다르다** (우연한 중복) → 각자 유지.
- 재사용 횟수가 1회뿐 → 미리 추상화하지 말기 (AHA 원칙 — Avoid Hasty Abstractions).

**3번째 재사용 시점**에 A/B 중 하나 적용이 경험적 균형.

## 체크리스트

- [ ] 같은 레이어 module import를 하고 있나? → A/B 검토.
- [ ] 이미 비슷한 조합이 다른 module에 있는가? → A안 유력.
- [ ] 레이어 수를 늘리고 싶지 않은가? → B안.
- [ ] 계약 이름이 동사형인가? (`PostRemover` vs `IPostsService`)
- [ ] B안 채택 후 모듈 import 경계를 정적으로 강제할 수단은? → [`09-framework-notes.md`](09-framework-notes.md).
- [ ] shared에서 도메인 개념이 필요한가? → 역참조 회피 DIP 적용.
- [ ] 재사용이 2회 미만이면 중복을 수용할지 재고.
