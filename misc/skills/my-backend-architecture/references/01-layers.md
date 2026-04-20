---
section: layers
---

# Layers

레이어는 **5개 기본** + **3개 선택**.

## 계층 구조

```
app (resource)
  ↓
(use-cases / features)?     (module)        선택
  ↓
(contracts)?                (module)        선택
  ↓
domain                      (module)
  ↓                          ▲
shared                       │ implements (DIP)
  ↑                          │
  └─────  infrastructure  ◄──┘
          (module — adapter)
```

- **위에서 아래로** 참조. 역방향 금지.
- **`infrastructure → domain`**만 import 방향이 역전됨(DIP). `domain`이 정의한 인터페이스를 `infrastructure`가 구현한다. 런타임 호출 흐름은 `domain → infrastructure`지만 코드 import는 반대.
- **모든 레이어가 `shared` 참조 가능**.

### 폴더 단위 명명

- **`app`은 resource** (API 리소스·페이지 리소스)
- **그 외 레이어는 module**

단, 이 스킬의 **"module"은 NestJS `@Module()`과 다른 개념**이다. 자세한 정의·참조 규칙·Public API(barrel)는 [`02-modules.md`](02-modules.md).

---

## app (resource)

### 책임

- 외부 세계와의 **경계**. HTTP 라우팅·GraphQL 리졸버·CLI 커맨드·스케줄 잡 엔트리.
- 여러 도메인을 **오케스트레이션**해서 유스케이스를 완성.
- 입출력 경계: 인증·validation·serialization·에러 매핑.

상세: [`03-app-layer.md`](03-app-layer.md).

### 구성

- 컨트롤러 / 리졸버 / CLI 핸들러
- 서비스 (여러 도메인 조합 오케스트레이터)
- DTO, Guards, Pipes, Interceptors (NestJS 기준)
- NestJS Module (resource별 1:1 대응이 일반적)

### API 경로 대응

REST라면 리소스 경로와 `app/` 하위 resource를 1:1 대응: `GET /employees` → `app/employees/employees.controller.ts`. 상세는 03.

---

## use-cases / features (선택 — module)

### 언제 추가

- 같은 조합이 **2곳 이상에서 재사용**될 때
- 조합이 **다단계**(트랜잭션·이벤트·보상 로직)로 복잡해 app 서비스에 담기 버거울 때
- **여러 Bounded Context를 엮는** 유스케이스일 때

필요 없으면 추가하지 않는다. **빈 레이어는 세금**.

### module 단위

`use-cases`의 module은 **비즈니스 유스케이스 단위** — 동사+목적어. `use-cases/onboard-employee/`, `use-cases/offboard-employee/`.

Bounded Context별로 묶지 않는다. BC 단위는 domain의 관심사.

### 이름 선택

`use-cases`(DDD·클린 아키텍처) vs `features`(FSD 영향) — **프로젝트당 하나만** 고정.

---

## contracts (선택 — module)

### 언제 추가

여러 쪽에서 공유하는 **계약 인터페이스**를 중앙 집중하고 싶을 때. DIP의 B안(계약 분리) 확장.

- 예: `app/users`·`app/orders` 등 여러 resource가 `Notifier` 인터페이스를 참조하고, 구현체는 `infrastructure/notification-notifier.adapter.ts`가 제공.
- 계약을 원 resource 안에 두면 그 resource에 import가 생겨 경계가 뚫림. 중립 레이어(contracts)로 올리면 깨끗.

세부: [`07-dip-patterns.md`](07-dip-patterns.md).

### module 단위

계약 단위 또는 주제 단위 — `contracts/notifier/`, `contracts/employee-lookup/`.

**계약끼리 참조할 일은 거의 없다**. 각 계약은 독립.

---

## domain (module)

### 책임

- 도메인 엔티티·VO·Aggregate·정책·이벤트 정의
- 비즈니스 규칙·불변 조건 구현
- Repository **인터페이스** 선언 (구현은 infrastructure)
- 자체는 NestJS `@Module()`을 **가지지 않는다** — POJO·순수 클래스·함수만

상세: [`04-domain-layer.md`](04-domain-layer.md).

### 외부 의존성

- 허용: `shared`만 (실용 타협, `00-overview` 참조)
- 금지: `app`·`use-cases`·`contracts`·`infrastructure` — 역방향. DB 드라이버·HTTP 클라이언트 직접 사용 금지.

### module 단위

도메인 module을 어떻게 나눌지 — 두 가지 패턴:

- **단일 Bounded Context** (작은 시스템) — module이 **Aggregate 단위**: `domain/employee/`, `domain/organization/`, `domain/rank/`
- **다중 Bounded Context** (중·대 시스템) — module이 **BC 단위**: `domain/workforce/`, `domain/payroll/` (각 BC 안에 여러 Aggregate)

module 간 참조는 `app`·`use-cases`·`contracts`·`infrastructure`와 달리 **허용**된다. 다만 BC 단위로 나눴다면 **BC 간 참조는 ID 타입 외 강한 의존(ORM 관계·객체 그래프) 지양**. 상세: 02 · 04.

### 포함 요소 (요약)

Entity · VO · Domain Service · Repository Interface · Domain Event · Policy · Factory · Domain Error · Anti-corruption Contract · Snapshot VO · DI Token · Public API(barrel).

파일 종류 카탈로그는 [`04-domain-layer.md`](04-domain-layer.md).

---

## infrastructure (module — adapter)

### 책임

`domain`이 정의한 인터페이스를 **실제 기술 스택으로 구현**하거나, 외부 시스템과의 접점을 캡슐화.

- **Repository 구현체** — domain 인터페이스의 TypeORM/Drizzle/Prisma 구현
- **Query Service** — CQRS 읽기 전용, 화면 DTO 직접 반환 (domain 우회)
- **외부 API 어댑터** — Payment Gateway·Email Provider·OAuth·SMS·검색 엔진 등
- **메시지 브로커 어댑터** — Kafka·Redis Pub/Sub·RabbitMQ
- **파일 스토리지 어댑터** — S3·GCS·Azure Blob
- **캐시 어댑터** — Redis·Memcached (domain이 캐시 인터페이스를 정의했을 때 구현체)
- **이벤트 게시자 구현체** — domain EventBus 인터페이스의 구현
- **Anti-corruption adapter** — 다른 BC 참조용 계약의 구현체

### module 단위

adapter 대상별로 분류. 보통 **도메인 이름 + 기술 이름**이 파일명에 들어감.

```
infrastructure/workforce/employee.typeorm.repository.ts
infrastructure/payment/stripe.gateway.ts
infrastructure/storage/s3-storage.adapter.ts
infrastructure/messaging/kafka-publisher.ts
```

파일명에 도메인 이름이 묻어도 OK — `infrastructure`는 "도메인 × 기술의 결합"이 정상이다. 이 점이 `shared`와 다르다.

### DIP

infrastructure가 domain 인터페이스를 구현하므로 **코드 import는 `infrastructure → domain`** 방향. 런타임 호출 흐름(`domain → infrastructure`)과 반대다. 상세: [`04-domain-layer.md`](04-domain-layer.md)의 DIP 섹션.

---

## shared (module — segment)

### 책임

도메인 의미 없는 **기술 인프라**. 어떤 레이어든 참조 가능한 기반.

상세: [`05-shared-layer.md`](05-shared-layer.md).

### 전형 세그먼트

- `logger`, `config`, `db`, `util`, `middleware`, `filters`, `guards`(기술적), `errors`, `types`

### DB 관련은 여기

**Drizzle/Prisma 스키마·마이그레이션·DB 연결·베이스 리포지토리는 `shared/db/`**. 스키마 자체는 "테이블 정의"라는 기술 자산이지 도메인 모델이 아니다. `infrastructure`는 그 스키마를 바라보고 **도메인 어댑터**(Repository·Query Service)를 작성한다.

구체 배치:

```
shared/db/
├── connection.ts                  (DataSource / Drizzle db 팩토리)
├── transaction-manager.ts
├── base-typeorm.repository.ts     (도메인 무관 베이스)
├── schema/                        (Drizzle·Prisma 스키마)
└── migrations/                    (drizzle-kit · typeorm migration)
```

### segment 간 참조

순환만 피하면 자유 참조 (02 참조).

---

## 확장 예시 (디렉토리 트리)

### 예 A — 도메인을 Aggregate 단위로 (단일 BC, 작은 HR 시스템)

```
src/
├── app/
│   ├── employees/                 (resource)
│   │   ├── index.ts
│   │   ├── employees.module.ts
│   │   ├── employees.controller.ts
│   │   ├── employees.service.ts
│   │   └── dto/
│   └── organizations/
│       └── ...
├── domain/                        (단일 Context, module이 Aggregate 단위)
│   ├── index.ts
│   ├── employee/
│   │   ├── employee.entity.ts
│   │   ├── employee.repository.ts     (interface)
│   │   └── employee-status.vo.ts
│   ├── organization/
│   │   └── organization.entity.ts
│   ├── rank/
│   │   └── rank.vo.ts
│   └── policies/
│       └── employee-placement.policy.ts
├── infrastructure/
│   ├── employee.typeorm.repository.ts
│   └── organization.typeorm.repository.ts
└── shared/
    ├── logger/
    ├── config/
    ├── db/
    │   ├── base-typeorm.repository.ts
    │   ├── transaction-manager.ts
    │   └── schema/              (Drizzle 쓸 경우)
    └── util/
```

### 예 B — 도메인을 BC 단위로 (다중 Context, HR 시스템 분화)

```
src/
├── app/
│   ├── employees/                 (resource)
│   ├── organizations/
│   ├── attendance/
│   └── payroll/
├── use-cases/                     (선택 — module)
│   └── onboard-employee/
│       └── onboard-employee.use-case.ts
├── contracts/                     (선택 — module)
│   └── employee-lookup/
│       └── employee-lookup.contract.ts
├── domain/                        (다중 Context, module이 BC 단위)
│   ├── workforce/                 ← BC
│   │   ├── index.ts
│   │   ├── employee/              (내부 Aggregate)
│   │   ├── organization/
│   │   ├── position/
│   │   ├── rank/
│   │   └── policies/
│   ├── attendance/                ← BC
│   │   ├── attendance/
│   │   └── shift/
│   └── payroll/                   ← BC
│       ├── payroll/
│       └── salary-policy/
├── infrastructure/
│   ├── workforce/
│   │   ├── employee.typeorm.repository.ts
│   │   └── queries/employee-dashboard.query-service.ts
│   ├── payroll/
│   │   └── payroll.drizzle.repository.ts
│   └── employee-lookup.adapter.ts      (contracts 구현체)
└── shared/
    ├── logger/
    ├── config/
    ├── db/
    │   ├── connection.ts
    │   ├── transaction-manager.ts
    │   └── schema/
    └── util/
```

두 예시의 차이는 오직 **`domain` 내부를 Aggregate 단위로 나누느냐 BC 단위로 나누느냐**. 선택 기준은 [`04-domain-layer.md`](04-domain-layer.md).
