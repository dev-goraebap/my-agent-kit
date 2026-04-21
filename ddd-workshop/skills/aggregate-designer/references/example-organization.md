# 예제: Organization Aggregate (사내 업무 시스템)

파일 경로: `docs/domains/identity/organization.md`

조직 구성원·부서·역할을 다루는 사내 솔루션의 핵심 Aggregate.

---

```markdown
---
aggregate: Organization
bc: Identity
classification: Supporting
status: active
version: 0.1.0
last_reviewed: 2026-04-21
---

# Organization

## Role

사내 구성원의 소속 부서·직책·역할(Role)을 관리한다.
**인증(로그인·토큰 발급)은 SSO(Keycloak)의 책임**이고, 이 Aggregate는
**권한·구성원 메타데이터**만 책임진다. 급여·평가 등 HR 영역은 범위 밖.

## Ubiquitous Language

| 용어 | 의미 |
|------|------|
| 조직 (Organization) | 회사 전체의 구성원·부서 트리 루트 |
| 부서 (Department) | 조직 내 계층 단위. 트리 구조. |
| 구성원 (Member) | 조직에 속한 사람 (퇴사자는 상태 변경만, 삭제 금지) |
| 역할 (Role) | 구성원이 가지는 권한 묶음 (예: 영업담당, 승인자, 감사자) |

## Structure

- **Root**: Organization
- **내부 Entity**: Department, Member, MemberRole
- **VO**: EmployeeNumber, Email, MemberStatus (`ACTIVE | ON_LEAVE | RESIGNED`)
- **외부 참조 (ID only)**:
  - `externalIdpUserId: string` (Keycloak subject)

## Invariants

- 모든 Member는 정확히 하나의 Department에 소속된다.
- 구성원의 EmployeeNumber는 전사적으로 유일하다.
- Member는 상태가 `RESIGNED`여도 삭제되지 않는다(감사 추적).
- Department는 자기 자신의 조상이 될 수 없다(순환 금지).
- `RESIGNED` Member는 Role을 가질 수 없다.
- 조직은 최소 1개의 Department(루트)를 가진다.

## Policies

### RoleAssignmentPolicy
- **위치**: Strategy
- **이유**: 조직 개편·신규 Role 신설이 잦다. "이 부서의 이 직책에는 이 Role 기본 할당" 같은 룰이 운영 설정으로 바뀌어야 함.
- **설명**: Member의 Department + 직책을 받아 기본 Role 세트를 산출.
- **교체 시나리오**: 조직 개편 시 운영자가 구성 변경.

### ApprovalDelegationPolicy
- **위치**: Domain Service
- **이유**: 승인 위임(휴가 중 대리 승인자)은 Member, Department, 날짜 기간, Role을 모두 고려해야 해 단일 Entity에 속하기 어색.
- **설명**: 현재 시점의 "실제 승인자"를 계산.

### ResignationPropagationPolicy
- **위치**: Event Handler
- **이유**: 구성원 퇴사 시 여러 BC(견적 시스템, 프로젝트, 결재 대기)에 전파해야 함. 비동기 반응이 자연스럽다.
- **설명**: `구성원퇴사됨` 이벤트를 발행하면 각 BC가 구독해 자기 영역 정리.

## External Dependencies

| Aggregate / BC / 외부 시스템 | 참조 방식 | 비고 |
|------------------------------|-----------|------|
| Keycloak (외부 IdP) | ACL + `externalIdpUserId` | 인증은 Keycloak이, 우리는 권한·메타 관리. Keycloak User Attribute로 EmployeeNumber 동기화. |
| SAP HR (외부 시스템) | ACL, 야간 배치 동기화 | 인사 변동(부서 이동·직책 변경)의 원천. 우리는 읽기만. |
| 견적 BC, 프로젝트 BC | 이벤트 발행 (`구성원퇴사됨`, `역할부여됨`) | 각 BC가 구독. |

## Domain Events (발행)

- `구성원등록됨` — 신규 Member 생성. payload: memberId, employeeNumber, departmentId.
- `구성원부서이동됨` — Department 변경.
- `구성원퇴사됨` — `RESIGNED` 전이.
- `역할부여됨` / `역할해제됨`.
- `부서신설됨` / `부서폐지됨`.

## Domain Events (구독)

- `SAP.인사변동` (외부 이벤트, ACL 변환) — 부서·직책 변경 반영.
- `SAP.입사` — 신규 Member 자동 등록 트리거.

## Open Questions

- ⚠️ **겸직 허용 여부**: 현재 Invariant는 "정확히 1개 Department". 겸직 요구 발생 시 Member:Department를 N:M으로 변경할지.
- ⚠️ **외부 파트너사 계정**: 협력업체 직원에게 제한적 접근을 주는 케이스를 Member로 볼지, 별도 Aggregate로 분리할지.
- ⚠️ **SAP와의 일관성 윈도우**: 야간 배치 주기 동안 두 시스템 불일치 허용 범위.
- ⚠️ **Role 상속 여부**: 부서 Role이 하위 부서로 자동 상속되는지 — 현재는 명시적 할당만.
```
