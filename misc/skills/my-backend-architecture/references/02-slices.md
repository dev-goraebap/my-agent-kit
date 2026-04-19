---
section: slices
---

# Slices

레이어 내 **기능 단위 폴더**를 슬라이스라 부른다 (FSD 용어 차용). 예: `app/users`, `app/posts`, `domain/users`, `shared/logger`.

## 같은 레이어 슬라이스 간 참조: 금지

```
app/users       ─X─▶ app/posts        (금지)
domain/orders   ─X─▶ domain/payments  (금지)
use-cases/a     ─X─▶ use-cases/b      (금지)
```

### 왜?

- 같은 레이어 슬라이스끼리 의존이 생기면 **의존 그래프가 평면 네트워크**가 되어, 어떤 슬라이스를 수정할 때 영향 범위 예측이 어려워진다.
- 테스트·이해·리팩터링의 단위가 "슬라이스 하나"로 유지되어야 레이어 구조가 실질적 가치를 만든다. 네트워크로 엮이기 시작하면 레이어가 시각적 폴더에 불과해진다.
- 이 규칙이 깨지기 시작하면 다른 규칙(레이어 단방향 등)도 **연쇄적으로 뚫린다**. 첫 위반이 가장 비싸다.

### 어떻게 피하나

같은 레이어 슬라이스끼리 기능이 엮이면 **DIP로 푼다**. 두 방향:

- **A안**: 공통 기능을 상위 레이어(`use-cases` / `features`)로 승격.
- **B안**: 계약 인터페이스를 만들고 원래 자리는 구현체로. 다른 슬라이스는 계약만 참조.

선택 기준은 [`07-dip-patterns.md`](07-dip-patterns.md).

### 단순 중복은 허용

코드 중복이 약간 있다고 해서 매번 A/B 기법을 꺼낼 필요는 없다. "중복 제거 vs 슬라이스 독립성" 사이 **트레이드오프**. 재사용 빈도가 낮고 기능이 단순하면 복사가 더 저렴하다. DIP는 재사용 횟수가 정말 늘어난 뒤 적용.

## shared 예외

`shared`는 위 규칙의 예외. **슬라이스 간 상호 참조 허용**. 단, 양방향 순환은 피한다.

```
shared/logger  ──▶ shared/config   (OK)
shared/db      ──▶ shared/logger   (OK)
shared/config  ──▶ shared/logger   (OK)

shared/a ──▶ shared/b ──▶ shared/a  (순환 금지)
```

### 왜 예외인가

`shared`의 세그먼트는 **도메인 요구사항과 무관한 기술 인프라**다. `logger`가 `config`를 참조하고 `db`가 `logger`를 참조하는 것은 자연스럽다. 이를 막으면 "shared의 shared" 같은 무한 승격이 발생해 구조가 오히려 복잡해진다.

순환(`a → b → a`)만 피하면 된다. 순환이 나타났다는 건 한쪽을 더 작게 쪼개거나 한쪽을 다른 쪽으로 흡수해야 한다는 신호.

## 정적 감지 수단

규칙을 개발자 규율에만 맡기면 결국 깨진다. 정적으로 강제하는 도구들:

- **`eslint-plugin-boundaries`** — ESLint 규칙으로 "app 슬라이스가 다른 app 슬라이스를 import하면 에러" 선언. IDE·PR 모두에서 즉시 피드백.
- **`dependency-cruiser`** — 독립 CLI. 의존 그래프 시각화 + 위반 리포트. CI에서 주기적 감사.
- **Nx monorepo의 `enforce-module-boundaries`** — 태그 기반 강제 (Nx 생태계 전제). 가장 강력하지만 러닝 커브 있음.
- **TypeScript path alias (`tsconfig paths`)** — `@app/*`, `@domain/*` 등으로 import 경로 제한. 우회가 쉬워 얕은 방어선이지만 같이 쓰면 유용.

구체 설정·선택 기준은 [`09-framework-notes.md`](09-framework-notes.md).

## 슬라이스 내부 구조

슬라이스 안은 자유롭게 구성. NestJS 기준 예:

```
app/users/
├── users.module.ts
├── users.controller.ts
├── users.service.ts
├── dto/
│   ├── create-user.dto.ts
│   └── update-user.dto.ts
└── guards/
    └── owner.guard.ts         ← 이 슬라이스 전용
```

```
domain/users/
├── user.entity.ts
├── user-status.vo.ts
├── user.repository.ts         (interface)
├── user.policies.ts           (도메인 정책 순수 함수)
└── events/
    └── user-registered.event.ts
```

원칙:

- **슬라이스 내부에서만 쓰는** guards/pipes/helpers는 슬라이스 안에 유지.
- **공통 guards/pipes**는 `shared`로 승격.
- 내부 파일 개수가 많아져 거슬리기 시작하면 **슬라이스 쪼개기** 신호 (`domain/users` → `domain/users` + `domain/user-authentication`).

## 체크리스트

슬라이스 경계를 판단할 때 쓰는 빠른 질문들:

- [ ] 이 import가 같은 레이어의 다른 슬라이스를 가리키나? → 금지, DIP 필요.
- [ ] 이 import가 아래 레이어를 건너뛰고 더 아래를 가리키나? (예: `app` → `shared`를 건너뛰고...) → 가능하지만 중간 레이어가 정말 필요 없는지 검토.
- [ ] 이 import가 위 레이어를 가리키나? → 역방향. 금지.
- [ ] 이 파일이 슬라이스 내부 전용인데 밖에서 쓰이고 있나? → 슬라이스의 public API를 명확히(barrel `index.ts`) 설계.
