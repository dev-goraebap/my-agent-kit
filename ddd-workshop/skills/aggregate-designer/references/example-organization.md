# 예제: Organization Aggregate (사내 업무 시스템)

파일 경로: `docs/shared/contexts/identity/organization.md`

조직 구성원·부서·역할을 다루는 사내 솔루션의 핵심 Aggregate.

---

```markdown
# Organization

> **Status**: active

## Role

### 책임지는 것
- 사내 구성원의 소속 부서·직책·역할(Role) 관리
- 권한·구성원 메타데이터 관리
- 조직 개편 이벤트 발행

### 책임지지 않는 것
- 인증(로그인·토큰 발급) — SSO(Keycloak) BC
- 급여·평가 — HR BC
- 프로젝트 배치 — Project BC

## Ubiquitous Language (이 Aggregate 범위)

| 용어 | Code Identifier | 의미 |
|---|---|---|
| 조직 | Organization | 회사 전체의 구성원·부서 트리 루트 |
| 부서 | Department | 조직 내 계층 단위. 트리 구조 |
| 구성원 | Member | 조직에 속한 사람 (퇴사자는 상태 변경만, 삭제 금지) |
| 역할 | Role | 구성원이 가지는 권한 묶음 (예: 영업담당, 승인자) |
| 사번 | EmployeeNumber | 전사 유일 식별자 |
| 구성원 상태 | MemberStatus | `ACTIVE / ON_LEAVE / RESIGNED` |

## Event Storming (Design-Level)

### 관련 플로우

```
[HR] →(구성원 등록)→ [구성원등록됨]
  ↓ 정책: 기본 Role 할당
  [역할부여됨]
[HR] →(부서 이동)→ [구성원부서이동됨]
[HR] →(퇴사 처리)→ [구성원퇴사됨]
  ↓ 정책: 타 BC 전파 (Event Handler)
```

### 해소된 핫스팟
- 퇴사 전파: 각 BC가 `구성원퇴사됨` 구독, 자기 영역 정리

### 미해소 핫스팟
- ⚠️ 겸직 허용 여부

## Structure

- **Root**: Organization
- **내부 Entity**: Department, Member, MemberRole
- **VO**: EmployeeNumber, Email, MemberStatus
- **외부 참조 (ID only)**:
  - `externalIdpUserId: string` (Keycloak subject)

## Invariants

1. 모든 Member는 정확히 하나의 Department에 소속된다.
2. 구성원의 EmployeeNumber는 전사적으로 유일하다.
3. Member는 상태가 `RESIGNED`여도 삭제되지 않는다(감사 추적).
4. Department는 자기 자신의 조상이 될 수 없다(순환 금지).
5. `RESIGNED` Member는 Role을 가질 수 없다.
6. 조직은 최소 1개의 Department(루트)를 가진다.

## Policies

### RoleAssignmentPolicy
- **위치**: Strategy
- **이유**: 조직 개편·신규 Role 신설이 잦음
- **설명**: Member의 Department + 직책을 받아 기본 Role 세트 산출
- **교체 시나리오**: 조직 개편 시 운영자가 구성 변경

### ApprovalDelegationPolicy
- **위치**: Domain Service
- **이유**: Member·Department·기간·Role을 모두 고려해 단일 Entity에 속하기 어색
- **설명**: 현재 시점의 "실제 승인자" 계산

### ResignationPropagationPolicy
- **위치**: Event Handler
- **이유**: 여러 BC에 전파되는 비동기 반응이 자연
- **설명**: `구성원퇴사됨` 발행, 각 BC 구독

## External Dependencies

| 대상 | 관계 | 참조 방식 | 비고 |
|---|---|---|---|
| Keycloak (외부 IdP) | ACL | `externalIdpUserId` | 인증 소관, 우리는 권한·메타 |
| SAP HR (외부) | ACL, 야간 배치 | 단방향 읽기 | 인사 변동 원천 |
| 견적 BC, 프로젝트 BC | Customer-Supplier | 이벤트 발행 | 각 BC가 구독 |

## Domain Events (발행)

- `구성원등록됨` (MemberRegistered) — payload: memberId, employeeNumber, departmentId
- `구성원부서이동됨` (MemberDepartmentChanged)
- `구성원퇴사됨` (MemberResigned) — `RESIGNED` 전이
- `역할부여됨` / `역할해제됨` (RoleAssigned / RoleRevoked)
- `부서신설됨` / `부서폐지됨` (DepartmentCreated / DepartmentArchived)

## Domain Events (구독)

- `SAP.인사변동` (외부 이벤트, ACL 변환) — 부서·직책 변경 반영
- `SAP.입사` — 신규 Member 자동 등록 트리거

## Exposed Queries

| Query | 반환 | 원천 | 주의 |
|---|---|---|---|
| `getMember(memberId)` | `MemberDetail` | Query Service | 권한 체크 포함 |
| `listDepartmentTree()` | `DepartmentTree` | Projection | 트리 캐시 |
| `findMembersByDepartment(deptId)` | `MemberSummary[]` | Projection | RESIGNED 기본 제외 |
| `getMemberRoles(memberId)` | `Role[]` | Query Service | 상속 여부 정책 반영 |

**Read-side 규약**: Aggregate 재조립 금지. 조직도는 변경 이벤트 구독으로 Projection 갱신.

## Open Questions

- ⚠️ **겸직 허용 여부**: 현재 Invariant는 "정확히 1개 Department". 겸직 요구 시 N:M 변경?
- ⚠️ **외부 파트너사 계정**: 협력업체 직원을 Member로 볼지, 별도 Aggregate 분리?
- ⚠️ **SAP와의 일관성 윈도우**: 야간 배치 주기 동안 두 시스템 불일치 허용 범위
- ⚠️ **Role 상속 여부**: 부서 Role이 하위 부서로 자동 상속? (현재는 명시 할당만)
```
