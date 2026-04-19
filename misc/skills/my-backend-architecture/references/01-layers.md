---
section: layers
---

# Layers

기본 3개 레이어 + 선택적 상위 레이어 하나.

## 계층 구조

```
app
 ↓
(use-cases | features)   ← 선택적 상위 레이어
 ↓
domain
 ↓
shared
```

**위에서 아래로만 참조**. 역방향 금지. 위 레이어는 아래의 타입·인터페이스·함수·클래스를 import할 수 있지만 그 반대는 허용하지 않는다.

## app

### 책임

- 외부 세계와의 **경계**. HTTP 라우팅, GraphQL 리졸버, CLI 커맨드, 스케줄 잡 엔트리 등 "바깥에서 시스템으로 들어오는 문".
- 여러 도메인을 **오케스트레이션**해서 한 유스케이스를 완성.
- 입출력 경계 처리: 인증·validation·serialization·에러 매핑.

세부: [`03-app-layer.md`](03-app-layer.md).

### 구성

- 컨트롤러 / 리졸버 / CLI 핸들러
- 서비스 (오케스트레이터 역할 — 여러 도메인 조합)
- DTO (요청·응답 모양)
- Guards, Pipes, Interceptors (NestJS 기준)
- 슬라이스 모듈 (`users.module.ts` 등)

### API 경로 대응

REST라면 리소스 경로와 폴더를 1:1 대응한다.

```
GET  /users         → app/users/users.controller.ts
POST /posts         → app/posts/posts.controller.ts
DELETE /posts/:id   → app/posts/posts.controller.ts
```

GraphQL이면 타입 이름 단위, CLI면 커맨드 단위로 같은 원리. 경로와 폴더가 **예측 가능한 대응**을 가지는 것이 이 규칙의 핵심 — 기능을 찾을 때 `grep` 이전에 경로만으로 위치를 안다.

## use-cases / features (선택)

### 언제 추가하나

다음 중 하나가 분명할 때:

- **여러 슬라이스에서 재사용**될 복잡한 오케스트레이션이 있다 (`07-dip-patterns.md`의 A안)
- **계약 인터페이스**만 모아두는 중립 자리가 필요하다 (B안에서 `domain/ports`나 이 레이어를 활용)
- 여러 도메인이 엮이는 흐름이 `app` 서비스에 담기에는 너무 크고 재사용성이 있다

필요 없으면 추가하지 않는다. **빈 레이어는 세금**.

### 이름 선택

- **`use-cases`** — 클린 아키텍처/DDD 전통. "여러 도메인 오케스트레이션"이라는 의도가 이름에 담김. "이 유스케이스를 실행한다"는 자연스러운 언어.
- **`features`** — FSD 영향. 프론트 FSD와의 통일성.

둘 중 **프로젝트 당 하나만** 고정해서 쓴다. 한 저장소에 `use-cases`와 `features`가 섞여 있으면 혼란.

### 구성

- 유스케이스 서비스 (여러 도메인 조합 로직)
- 선택: 계약 인터페이스 + DTO (B안 사용 시)

### app과의 경계

`app`의 서비스가 2~3개 도메인만 조합하고 한 곳에서만 쓰이면 `app` 슬라이스 내부 서비스로 충분. 같은 조합이 **두 곳 이상에서 필요해지는 순간**이 이 레이어로 올릴 타이밍.

## domain

### 책임

- **비즈니스 규칙·불변 조건·도메인 엔티티·도메인 이벤트** 정의.
- 외부 인프라(DB 드라이버, HTTP 클라이언트)를 **모르는 것이 이상** — 다만 이 스킬은 `shared` 참조는 허용(순수주의 타협, `00-overview.md` 참조).
- **단위 테스트 용이성**이 최상 원칙.

세부: [`04-domain-layer.md`](04-domain-layer.md).

### ORM 전략에 따른 형태

- **TypeORM (Rich Entity)**: 엔티티 클래스에 비즈니스 규칙 메서드 탑재. 엔티티 자체를 테스트 대상.
- **Drizzle / Prisma**: 스키마는 별도. 도메인 모델은 POJO/POCO. 리포지토리가 ORM 호출을 감싼다.

자세한 선택 기준: [`06-orm-strategies.md`](06-orm-strategies.md).

### 포함 요소

- 엔티티 / VO (Value Object)
- 도메인 서비스 (규칙이 한 엔티티에 속하지 않을 때)
- 리포지토리 **인터페이스** (구현체는 `shared`나 별도 infrastructure)
- 도메인 이벤트 정의

## shared

### 책임

- 도메인 의미 없는 **기술 세그먼트**. 어떤 레이어든 참조 가능한 기반.

세부: [`05-shared-layer.md`](05-shared-layer.md).

### 전형적 세그먼트

- `logger` — 로깅 인터페이스·구현
- `config` — 환경변수·설정 파싱
- `db` — DB 연결 팩토리·트랜잭션 헬퍼
- `util` — date/string/number 헬퍼
- `middleware` — 공통 HTTP 미들웨어
- `filters` — 예외 필터 (NestJS)
- `errors` — 도메인 무관 기반 에러 (NotFound, Unauthorized 등)

### 여기 들어가면 안 되는 것

- 도메인 의미가 묻은 코드 (`user` 개념이 등장하는 것은 `domain/users`로).
- 역참조가 필요해 보이면 `07-dip-patterns.md`의 DIP 기법으로 해결.

## 사용자 정의 레이어 추가

4개(app / use-cases|features / domain / shared) 외에 프로젝트 요구에 따라 추가 가능.

원칙:

- **의존 방향 규칙은 그대로** — 추가 레이어도 단방향.
- **레이어 수가 늘수록 이해·유지보수 비용**이 비례한다. 5개 이상이면 정말 필요한지 재고.
- 흔한 추가 예: `infrastructure` (DB·외부 API 어댑터 구현체) — `domain`과 `shared` 사이에 끼워 DIP를 선명하게. 대부분은 `shared`로 커버 가능해 꼭 필요하진 않다.

## 확장 예시 (디렉토리 트리)

```
src/
├── app/
│   ├── users/
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.module.ts
│   │   └── dto/
│   │       ├── create-user.dto.ts
│   │       └── update-user.dto.ts
│   └── posts/
│       └── ...
├── use-cases/                   ← 선택
│   └── remove-user-with-posts/
│       └── remove-user-with-posts.use-case.ts
├── domain/
│   ├── users/
│   │   ├── user.entity.ts       (TypeORM이면 ORM 엔티티 겸 도메인)
│   │   ├── user-status.vo.ts
│   │   └── user.repository.ts   (interface)
│   └── posts/
│       └── ...
└── shared/
    ├── logger/
    ├── config/
    ├── db/
    │   └── base-typeorm.repository.ts
    └── util/
```

슬라이스 내부 구조 자유도는 `02-slices.md` 참조.
