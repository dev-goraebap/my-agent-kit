---
section: domain-layer
---

# domain Layer

비즈니스 규칙·불변 조건이 모이는 **시스템의 심장**. 외부 의존성 최소화 + 단위 테스트 용이성이 최고 원칙.

## 책임

- Entity · VO · Aggregate 구조 정의
- 비즈니스 규칙·불변 조건 구현
- 도메인 이벤트 선언
- Repository **인터페이스** 선언 (구현은 `infrastructure/`)
- Domain Service — 한 엔티티에 속하지 않는 규칙 판정
- Policy · Factory · Domain Error 등 (아래 파일 카탈로그)

## NestJS `@Module()` 없음 (중요)

`domain`은 **NestJS `@Module()`을 가지지 않는다**. 파일들은 POJO · 클래스 · 함수 · 인터페이스만.

- DI 컨테이너 배선은 `app`과 `infrastructure`에서 수행 (provider 바인딩)
- 도메인 코드는 **프레임워크 의존 없이** 테스트 가능해야 함 — `@Injectable`을 달아도 동작은 하지만 그로 인해 테스트할 때 NestJS Testing Module을 끌고 와야 한다면 설계 의도에 어긋남
- 같은 원칙이 Spring Boot·.NET에서도 적용 — 도메인은 `@Service`·`@Component` 등 프레임워크 어노테이션 최소화

이 점이 `app` · `infrastructure` module과 구별되는 가장 큰 차이. 상세한 module vs NestJS Module 대조는 [`02-modules.md`](02-modules.md).

## 외부 의존성: 최소화하되 0은 아님

Clean/Hexagonal은 "도메인은 외부 의존 0"을 이상으로 삼지만 이 스킬은 **`shared` 참조까지 허용**한다 ([`00-overview.md`](00-overview.md) "Not Clean Architecture").

허용:

- `shared/logger` 인터페이스 (구체 구현 아닌 추상)
- `shared/errors` 기반 에러 (`NotFoundError` 등)
- `shared/util` 순수 유틸 (date/string/math)
- `shared/types` 횡단 타입 (`Result`, `Paginated`)

금지:

- `app` / `use-cases` / `contracts` / `infrastructure` — 역방향
- DB 드라이버 직접 — Repository 인터페이스로 감싼다
- HTTP 클라이언트 직접 — 외부 API 어댑터는 `infrastructure`에 두고 인터페이스로

### 왜 shared는 허용하나

도메인 코드가 `dayjs(user.createdAt).isAfter(...)` 하나 쓴다고 테스트가 어려워지지 않는다. 반면 모든 유틸을 "도메인 서비스"로 감싸면 보일러플레이트 폭증. 실용 타협.

---

## module 구성: Aggregate 단위 vs BC 단위

`domain` module을 어떻게 나눌지는 시스템 규모에 따라 달라진다.

### 단일 Bounded Context — module이 Aggregate 단위

도메인 개념이 한 용어 체계로 충분. 같은 단어가 맥락별로 다른 의미로 쓰이지 않는 경우.

```
domain/
├── index.ts
├── employee/                       ← Aggregate 단위 module
│   ├── employee.entity.ts
│   ├── employee-status.vo.ts
│   └── employee.repository.ts
├── organization/
│   ├── organization.entity.ts
│   └── organization.repository.ts
├── position/
│   ├── position.entity.ts
│   └── position.repository.ts
├── rank/
│   └── rank.vo.ts                  ← VO 전용 module
├── policies/                       ← 여러 Aggregate 걸친 정책
│   └── employee-placement.policy.ts
└── events/
    └── employee-hired.event.ts
```

- module 간 참조 자유 (같은 BC 안)
- ORM 관계를 맺어도 같은 BC 안이라 큰 문제 없음 (단 Aggregate 경계는 여전히 신경)

### 다중 Bounded Context — module이 BC 단위

같은 단어가 BC마다 다른 모델이 될 때. 각 BC는 자체 Aggregate들을 내부에 보관.

```
domain/
├── workforce/                      ← BC 단위 module
│   ├── index.ts
│   ├── employee/                      (내부 Aggregate 폴더)
│   ├── organization/
│   ├── position/
│   ├── rank/
│   ├── policies/
│   └── events/
├── attendance/                     ← BC
│   ├── attendance/
│   └── shift/
└── payroll/                        ← BC
    ├── payroll/
    └── salary-policy/
```

- 같은 BC 안 Aggregate 간 참조는 자유
- **BC 간 참조는 ID 타입 외 강한 의존 지양** (아래 §module 간 참조 규칙)

### 선택 기준

"같은 단어가 맥락에 따라 다른 모델로 쓰이는가?"

- No → Aggregate 단위 (단일 BC)
- Yes → BC 단위 (다중 BC)

프로젝트 시작 시 **명시적으로 결정**하고 팀·에이전트와 공유 (README·AGENTS.md에 한 줄 기록).

### 세그먼트 분류는 안티패턴

```
# 지양
domain/
├── models/
├── value-objects/
├── repositories/
└── services/
```

기술적 유형별 분류는 Employee 개념을 이해하려 네 폴더를 오가게 만든다. 응집도 파괴. **의미 단위**(Aggregate 또는 BC)로 묶는다.

---

## Aggregate — BC 내부의 트랜잭션 단위

Bounded Context 안에는 **여러 Aggregate**가 있는 것이 기본이다.

- **트랜잭션·불변 조건 경계** — 한 Aggregate 안의 변경은 한 트랜잭션으로 완결
- **Root Entity**가 외부 창구 — 외부는 Root만 안다. 내부 VO는 몰라도 됨
- **Aggregate 간 참조는 ID로** — 객체 그래프 대신 ID를 소유 (`order.buyerId: string`)
- **VO는 자유 소유** — 값 자체가 의미라 Aggregate 경계 밖에도 자연스러움

### Aggregate 식별 기준

1. **독립된 라이프사이클** — 따로 생성·삭제되면 Aggregate Root 후보
2. **자체 불변 조건** — "주문 총금액 = 항목 금액 합" 같은 규칙이 있으면 Root
3. **다른 Aggregate가 ID로 참조하는 대상** — 별도 Aggregate
4. **값 자체가 의미** — `Rank`, `Money`, `Email` → VO

HR "Workforce" BC 예: Organization · Position · Employee는 각자 Aggregate Root, Rank · EmployeeStatus는 VO.

### 전체 계층

```
Project / Service
  └─ Bounded Context (module 하나 또는 domain 전체)
       └─ Aggregate (Root Entity + 내부 Entity · VO)
            └─ Entity / VO
```

---

## module 간 참조 규칙 (domain 예외)

`domain`은 `shared`와 함께 **유일하게 module 간 참조가 허용되는** 레이어다. 다만 기술적 주의가 있다.

### 규칙

- **Aggregate 단위 module** (단일 BC): 자유 참조.
- **BC 단위 module** (다중 BC): **BC 간 참조가 정말 필요한지 검토**. ID 타입 참조로 충분하지 않은지.

### ORM 관계는 BC 경계 넘지 말 것

같은 BC 안 Aggregate 간 ORM 관계도 Aggregate 경계를 고려해야 하고, **BC 경계를 넘는 ORM 관계는 사실상 피한다**.

```ts
// 지양 — TypeORM에서 BC 경계 넘는 관계
@Entity()
class Payroll {
  @ManyToOne(() => Employee)    // ← Workforce BC의 Employee와 관계
  employee: Employee;
}

// 권장 — ID 타입만
@Entity()
class Payroll {
  @Column('uuid')
  employeeId: string;
}
```

이유:

- Cascade·eager loading이 BC 경계를 넘어 엮임
- 마이그레이션·인덱스가 두 BC에 걸쳐 복잡해짐
- 나중에 마이크로서비스로 쪼갤 때 ORM 리팩터 비용 폭증

ORM별 구체 주의사항은 [`06-orm-strategies.md`](06-orm-strategies.md).

### 조인이 필요하면 Query Service로

화면 조회에서 여러 BC 데이터가 필요하면 `infrastructure/queries/`의 Query Service가 테이블 조인으로 화면 DTO를 직접 반환 (CQRS 경량). domain을 거치지 않는다. 상세: [`03-app-layer.md`](03-app-layer.md).

### 도메인 규칙 판정에 다른 BC 정보가 필요하면

도메인 로직(쓰기)에서 다른 BC 정보가 필요한 경우. 예: `payroll` 도메인 서비스가 급여 계산 시 Employee의 직급·근속년수 필요.

해법:

- **ID + Anti-corruption Contract** — `domain/payroll/employee-lookup.contract.ts`에 인터페이스 정의, `infrastructure/payroll/employee-lookup.adapter.ts`가 workforce BC 참조해 구현
- **Snapshot VO** — 필요한 필드만 복사한 VO(`PayableEmployee`)를 자기 BC에 두고 이벤트로 동기화

Query Service와는 다른 용도 — 자세한 구분은 [`03-app-layer.md`](03-app-layer.md)의 "Query Service ↔ 도메인 스냅샷 VO 혼동 주의" 섹션.

---

## 파일 종류 카탈로그

domain module에 들어갈 수 있는 파일 유형.

| # | 종류 | 예시 파일명 | 역할 |
|---|------|-------------|------|
| 1 | **Entity** | `employee.entity.ts`, `order.entity.ts` | 식별자 가진 도메인 객체. Aggregate Root 또는 내부 Entity |
| 2 | **Value Object (VO)** | `rank.vo.ts`, `money.vo.ts`, `employee-status.vo.ts` | 식별자 없이 값 자체로 의미. 불변 |
| 3 | **Domain Service** | `promotion.policy.ts`, `payment-authorizer.service.ts` | 여러 Entity·VO 걸친 규칙 판정. 한 엔티티 메서드로 쓰기 애매할 때 |
| 4 | **Repository Interface** | `employee.repository.ts` | 영속성 계약. 구현은 `infrastructure`에 |
| 5 | **Domain Event** | `employee-hired.event.ts`, `order-paid.event.ts` | 과거형 이벤트. 외부 BC·인프라가 구독 |
| 6 | **Policy / Specification** | `promotion.policy.ts`, `discount.specification.ts` | 추상화된 판정 함수·객체. 여러 정책 결합 시 유용 |
| 7 | **Factory** | `employee.factory.ts` | Aggregate 생성 로직이 복잡할 때 캡슐화 (간단하면 생성자로 충분) |
| 8 | **Domain Error** | `order-not-found.error.ts`, `cannot-suspend-deleted-user.error.ts` | 도메인 예외. `shared/errors`와 달리 도메인 용어로 이름 |
| 9 | **Anti-corruption Contract** | `employee-lookup.contract.ts` | 다른 BC 참조용 인터페이스. 구현체는 `infrastructure` |
| 10 | **Snapshot VO** | `payable-employee.vo.ts` | 다른 BC 정보를 자기 Context에 복사한 VO |
| 11 | **DI Token** | `tokens.ts` | NestJS provider 토큰 모음 (`EMPLOYEE_REPOSITORY` 등) |
| 12 | **Public API** | `index.ts` | module barrel — 외부에 공개할 것만 export |

### 들어가면 안 되는 것

- ORM 구체 클래스·드라이버 (`TypeORM Repository<User>`, `PrismaClient`) → `infrastructure`
- HTTP·메시지 브로커 구체 클라이언트 → `infrastructure`
- 컨트롤러·DTO·validator → `app`
- 화면용 Query Service·Read DTO → `infrastructure`의 CQRS 읽기 경로
- NestJS `@Module()` → `app` 또는 `infrastructure`

---

## Domain Service vs Application Service

둘 다 "Service" 접미사라 혼동하기 쉬운데 **책임과 위치가 완전히 다르다**.

### 핵심 차이

| 기준 | Application Service | Domain Service |
|------|---------------------|----------------|
| 위치 | `app` / `use-cases` 레이어 | `domain` 레이어 (자기 BC 안) |
| 주 역할 | **오케스트레이션** (여러 조각 조합) | **규칙 판정** (도메인 지식 자체) |
| 트랜잭션 관리 | ✅ 여기서 | ❌ |
| Repository 주입 | ✅ 여기서 | ❌ (규칙 적용 대상을 **인자로** 받음) |
| 외부 API 호출 | ✅ | ❌ |
| 이메일·알림·이벤트 발행 | ✅ | ❌ (순수 지향) |
| 비즈니스 규칙 판정 | ❌ domain으로 위임 | ✅ |
| 의존 | Repository·infrastructure·다른 서비스 | `shared`만 |
| 이름 예 | `EmployeesService`, `OnboardEmployeeUseCase` | `PromotionPolicy`, `PaymentAuthorizer` |

### 같은 시나리오로 비교 — "직원 승진"

**Application Service** — 오케스트레이터

```ts
// use-cases/promote-employee/promote-employee.use-case.ts
@Injectable()
export class PromoteEmployeeUseCase {
  constructor(
    @Inject(EMPLOYEE_REPOSITORY) private employeeRepo: EmployeeRepository,
    @Inject(POSITION_REPOSITORY) private positionRepo: PositionRepository,
    private promotionPolicy: PromotionPolicy,     // ← Domain Service 주입
    private eventBus: EventBus,
    private tx: TransactionManager,
  ) {}

  async execute(dto: PromoteDto) {
    return this.tx.run(async () => {
      const employee = await this.employeeRepo.findById(dto.employeeId);
      const position = await this.positionRepo.findById(dto.newPositionId);

      this.promotionPolicy.apply(employee, position);   // ← 순수 규칙 호출

      await this.employeeRepo.save(employee);
      await this.eventBus.publish(new EmployeePromoted(employee.id));
    });
  }
}
```

**Domain Service** — 순수 규칙

```ts
// domain/workforce/policies/promotion.policy.ts
export class PromotionPolicy {
  // Repository 주입 없음. 의존성 0.
  apply(employee: Employee, newPosition: Position): void {
    if (employee.tenureMonths < 12) {
      throw new InsufficientTenureError(employee.id);
    }
    if (employee.currentRank.level >= newPosition.minRank.level) {
      throw new PositionDowngradeError(employee.id);
    }
    employee.assignPosition(newPosition.id);
    employee.promoteTo(newPosition.minRank);
  }
}
```

역할 분리 명확:

- Application Service가 I/O · 트랜잭션 · 이벤트
- Domain Service가 순수 규칙 판정 + Entity 상태 변경

### 판단 기준

엔티티 메서드 > Domain Service > Application Service 순으로 우선순위:

1. **엔티티 하나에 담을 수 있나** → 엔티티 메서드 (`employee.suspend(reason)`)
2. **여러 엔티티·VO 걸친 규칙** → Domain Service
3. **사이드 이펙트·I/O·이벤트** → Application Service

### Domain Service의 위치 — BC 레벨

Domain Service는 **Bounded Context 레벨**에 위치한다 — Aggregate 폴더 안이 아니라 BC 내 공통 폴더(`policies/` 또는 `services/`)에.

```
domain/workforce/
├── employee/                    ← Aggregate
├── organization/                ← Aggregate
├── position/                    ← Aggregate
├── policies/                    ← BC 레벨 (Aggregate와 나란히)
│   ├── promotion.policy.ts          (Employee + Position 조합)
│   └── employee-placement.policy.ts (Employee + Organization + Position 조합)
└── events/
```

이유:

- 여러 Aggregate에 걸친 규칙은 **어느 한 Aggregate 소유가 아님**
- Aggregate 폴더 안에 넣으면 "이 Aggregate의 일부"로 오인 유도
- BC 레벨이 중립 자리

### 흔한 함정

| 잘못된 패턴 | 교정 |
|-------------|------|
| Domain Service가 Repository를 주입 | → Application Service로 이동. Domain Service는 데이터를 **인자로** 받는다 |
| Application Service가 `if (employee.status === ACTIVE) { ... }` 같은 규칙 판정 | → 규칙을 Entity 메서드·Domain Service로 빼라 (Anemic 방지) |
| 둘 다 Service 접미사 | → Domain Service는 **동사·역할 중심 이름** (`PromotionPolicy`), Application Service는 **리소스 + Service** (`EmployeesService`) |
| Domain Service를 Aggregate 폴더 안에 둠 | → BC 레벨 공통 폴더(`policies/`)로 |

---

## Repository 주입 원칙

Repository는 **Application Service(또는 Use-Case Service)에 주입**한다. **Domain Service에는 주입하지 않는다**(원칙). 이유는 네 가지.

### 1. 테스트 용이성

Domain Service가 Repository 주입 → 단위 테스트마다 Mock 작성 필수.
인자로 받으면 → POJO 만들어 넣고 결과 검증. 프레임워크·모킹 도구 없이 끝.

이 스킬의 "도메인 테스트는 모킹 없이" 원칙 ([`08-testing.md`](08-testing.md))과 직결.

### 2. 트랜잭션 경계 명확성

트랜잭션은 **유스케이스 수준**에서 열고 닫는다. Domain Service가 내부에서 `save()`를 호출하면:

- 언제 트랜잭션이 시작되는지 호출자가 모름
- 여러 Domain Service 연쇄 호출 시 각각이 자기 커밋을 해버림
- 중간 실패 시 부분 상태

Application Service가 트랜잭션을 쥐고, Domain Service는 Entity 상태만 바꾸면 경계가 한 곳에 집중.

### 3. 쿼리 횟수 예측성

Domain Service가 내부에서 `findById()`를 호출하면 "이 메서드가 DB 몇 번 치나"가 호출자에게 보이지 않음. N+1 위험.

Application Service가 명시적으로 로드하면 쿼리 횟수가 드러남.

### 4. 도메인 순수성

Repository 호출은 I/O 경계를 넘는 행위. Domain Service가 I/O 하면:

- 순수 함수 아님 — DB 상태에 따라 결과 달라짐
- Clean 아키텍처식 의존 역전이 의미 축소

### 쓰기는 절대 도메인 서비스 금지

읽기는 예외 여지가 있지만 **`save`·`update`·`delete` 같은 쓰기는 Domain Service가 하지 않는다**. 쓰기는 트랜잭션 결정 + 이벤트 발행 + 보상 로직이 걸린 유스케이스 수준 관심사.

### 읽기 — A안 / B안

규칙 판정에 다른 데이터 조회가 **필수**인 경우 (예: 이메일 중복 검사 규칙).

**A안 (엄격, 기본 권장)** — Application Service가 먼저 조회해서 결과를 인자로 전달.

```ts
const isTaken = await this.userRepo.existsByEmail(dto.email);
this.registrationPolicy.apply(dto, isTaken);  // boolean 전달
```

**B안 (실용, 읽기 전용 타협)** — Domain Service가 **쓰기 메서드 없는 Read-Only 인터페이스**를 주입.

```ts
export class RegistrationPolicy {
  constructor(
    @Inject(USER_REPOSITORY) private userRepo: UserReadOnlyRepository,
  ) {}

  async apply(dto: RegisterDto): Promise<void> {
    if (await this.userRepo.existsByEmail(dto.email)) {
      throw new EmailAlreadyTakenError(dto.email);
    }
  }
}
```

기본은 A안, B안은 A안이 부담스럽고 규칙이 확실히 도메인에 속할 때만.

---

## DIP — 코드 import 방향은 `infrastructure → domain`

Repository 배치는 단순 "코드 나누기"가 아니라 **DIP의 실천**이다.

### 일반적 기대 — "상위가 하위를 쓴다"

```
app → use-cases → domain → infrastructure → shared
```

이 관점에서 domain이 infrastructure를 쓰는 것처럼 읽히지만, 실제 import는 반대.

### 실제 코드 import 방향

Repository 인터페이스가 domain에 있기 때문에:

```
infrastructure/workforce/employee.typeorm.repository.ts
   │
   │  implements
   ▼
domain/workforce/employee/employee.repository.ts  (interface)
```

**`infrastructure → domain`** — 하위 레이어가 상위 레이어 인터페이스를 구현. 이것이 의존 **역전**.

### 두 관점 분리

| 관점 | 방향 | 의미 |
|---|---|---|
| 컴파일 타임 (코드 `import`) | `infrastructure → domain` | 구현체가 인터페이스 참조 |
| 런타임 (호출 흐름) | `app → ... → domain → Repository interface → (DI) → infrastructure 구현체` | 도메인 서비스가 인터페이스 호출하면 DI 컨테이너가 실제 구현체 실행 |

두 방향이 반대임을 혼동하지 말 것.

### 왜 중요한가

- domain이 ORM·인프라를 모른 채 **순수 유지** → 단위 테스트 용이
- ORM 교체(TypeORM → Drizzle) 시 domain 코드 무영향
- "도메인이 규칙, 인프라가 기계"라는 관계가 코드 구조에 그대로 드러남

---

## 비즈니스 규칙의 위치 (ORM 의존)

규칙의 물리적 위치는 **ORM 전략에 종속**된다.

- **TypeORM (Rich Entity)**: 엔티티 클래스 = ORM 엔티티 = 도메인 모델. 규칙 메서드를 엔티티에 탑재.
- **Drizzle / Prisma**: 스키마 별도(`shared/db/schema/`), 도메인 모델은 POJO. 규칙은 모델 또는 Domain Service에. Repository 구현체(`infrastructure`)가 스키마 ↔ 도메인 모델 매핑.

어느 쪽이든 **규칙은 데이터와 가까이** 둔다. Application Service에 규칙이 흘러가면 **Anemic Domain Model 안티패턴** — 피한다.

자세한 비교: [`06-orm-strategies.md`](06-orm-strategies.md).

## 도메인 이벤트

- Context 안 `events/` 폴더
- **과거형** 이름: `EmployeeHired`, `OrderPaid` (미래형은 커맨드)
- 발행/구독:
  - 작은 프로젝트: 서비스에서 직접 콜백 호출 또는 간단한 EventEmitter
  - 커지면 `@nestjs/cqrs` EventBus 또는 메시지 브로커(Kafka 등)

CQRS 관점은 [`09-framework-notes.md`](09-framework-notes.md).

## 테스트 우선순위

도메인 레이어는 이 스킬의 **테스트 1순위**. 자세히: [`08-testing.md`](08-testing.md).

- Entity 메서드·Domain Service·Policy → 단위 테스트 필수
- 모킹 없이 순수 입력·출력으로
- Repository 필요 시 in-memory 가짜 구현체

---

## 체크리스트

- [ ] domain 코드에서 `import typeorm` / `import { PrismaClient }` 등장? → Repository로 감싸라
- [ ] domain에 `@Module()` 선언 있는가? → 제거. DI는 상위 레이어에서
- [ ] 비즈니스 규칙이 Application Service에 흘렀는가? → Anemic Model. domain으로 올려라
- [ ] Domain Service가 Repository를 주입받는가? → Application Service로 이동. 인자 전달
- [ ] Domain Service가 `save()`·쓰기 호출하는가? → 제거. 쓰기는 Application Service
- [ ] 다른 BC의 Entity를 ORM 관계로 맺었나? → ID 타입으로 교체
- [ ] module이 기술 유형별 분류(`models/`·`repositories/`)로 빠졌나? → Aggregate 또는 BC 단위로 재구성
- [ ] Aggregate 간 객체 참조(`employee.position: Position`)가 있는가? → ID 참조(`employee.positionId: string`)로
- [ ] Domain Service가 Aggregate 폴더 안에 묻혀있는가? → BC 레벨 `policies/`로 이동
- [ ] module마다 Public API(`index.ts`)가 있고 외부는 barrel만 참조하는가?
