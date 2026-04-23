---
name: screen-inventory
description: >
  ddd-workshop 파이프라인의 4-옵션 단계 (선택적). context-designer 완료 후
  UI가 있는 BC에 대해서만 호출. 해당 BC가 기여하는 화면 목록을 표시
  데이터·액션·Query 이름과 함께 정리해 **read-path를 1급 산출물로**
  노출시킨다. 와이어프레임 없이도 작성 가능하며 후속 aggregate-designer의
  "Exposed Queries" 섹션과 이름 매칭으로 크로스체크된다. UI가 없는
  Generic BC(알림 등)에는 호출하지 않는다. "스크린 인벤토리",
  "screen inventory", "UI 인벤토리", "화면 목록", "read path",
  "조회 경로", "screen-inventory", "ddd-workshop 4-옵션" 같은 요청에
  트리거한다.
metadata:
  author: dev-goraebap
  version: "0.3.0"
---

# screen-inventory 스킬

한 BC가 **기여하는** 화면 목록을 정리한다. 한 화면이 여러 BC의 데이터를 섞어 보여주면 각 BC의 Screen Inventory에 동일 화면이 등장하되, **관점이 다르게** 기술된다.

이 산출물의 존재 이유는 **read-path 강제 노출**. 요구사항이 동사 중심으로 정리되면 조회 요구가 빠지기 쉬운데, 화면 단위로 보면 "이 화면엔 이 데이터가 필요" 가 자동으로 드러난다.

**와이어프레임 없이도 작성 가능**. 표시 데이터와 액션만 자연어로 기술하면 충분.

---

## 대화 UX 규약 (필수)

- **카테고리 선택은 숫자 메뉴**. 자유 입력 금지.
- **화면 1개당 1문 1답 묶음**. (개별 필드가 아닌 화면 단위로 묶어 질문)
- **진행 표시**: `[Q 3/7 · 화면: 내 신청 목록]`
- **"나중에" / "모름" / "스킵" 허용** → `⚠️ 미정`.
- **화면이 없는 BC면 즉시 종료** — "이 BC는 UI 노출 없음"이라고 안내하고 파일을 만들지 않음.

---

## 1. 언제 트리거하는가

- `/ddd-workshop:screen-inventory <bc-name>`
- `context-designer` 완료 후 UI 있는 BC에 대해 개별 호출
- "스크린 인벤토리", "화면 목록 정리", "read path 뽑아줘"
- 기존 Screen Inventory 업데이트 → `--update`

**호출 단위**: BC 1개당 1회. 여러 BC에 적용하려면 여러 번 호출.

---

## 2. 입력

- `docs/shared/contexts/<bc>/_canvas.md` — **주 입력**
  - 해당 BC의 Inbound Commands/Queries
  - 관여 액터
- `docs/shared/event-flow.md` — 이벤트 흐름 (액터 활동 추출)
- `docs/shared/context-map.md` — BC 간 관계 (어느 BC가 어느 데이터 권위인지 확인)

---

## 3. Step 0 — 화면 존재 여부 확인

```
[Q · 이 BC에 화면이 있나요?]

이 BC의 외부 노출 형태를 확인합니다.

1) 사용자 UI가 있음 (웹·모바일 화면)
2) 관리자 UI만 있음
3) 외부 API만 제공 (UI 없음, 타 BC/시스템이 호출)
4) 내부 전용 (타 BC만 사용, UI·API 모두 없음)

2번도 Screen Inventory 대상입니다.
3번·4번은 이 스킬을 종료합니다.
```

3·4 선택 시 종료 메시지:

```
✅ 이 BC는 UI 노출이 없어 Screen Inventory가 필요 없습니다.
   aggregate-designer의 "Exposed Queries" 섹션만으로 read-path가 충분히
   기술됩니다.
```

---

## 4. 진행 단계 (화면 있는 BC)

### Step 1 — 화면 목록 초안

Canvas의 Inbound Commands/Queries와 event-flow의 액터 활동을 근거로 화면 후보를 제시:

```
[Q · 화면 후보 검토]

이 BC가 기여하는 화면 후보입니다. 빠진 것 있나요?

액터별 후보:
- [사원]
  1) 내 신청 목록
  2) 내 신청 상세
  3) 신청 작성
  4) 연차 잔고 위젯
- [결재자]
  5) 내 결재 대기함
- [관리자]
  6) 전체 신청 관리
  7) 잔고 조정 화면

1) 이대로 OK
2) 추가할 화면이 있음
3) 위 목록에서 제거할 화면
4) 화면 이름 변경
```

### Step 2 — 화면별 상세 (1화면 1문)

각 화면마다 다음을 묶음으로 질문:

```
[Q 2/7 · 화면: 내 신청 목록]

이 화면에 대해 확인합니다.

- 경로: /me/leave-requests
- 주 액터: 사원
- 표시 데이터 후보: 신청 ID, 기간, 휴가 종류, 상태, 제출일, 잔여 연차
- 액션 후보: 상세 보기, 취소 신청
- 필터/정렬 후보: 상태별, 최근순

1) 이대로 OK
2) 표시 데이터 수정
3) 액션 수정
4) 경로 수정
5) 이 화면은 제거
```

### Step 3 — Query 이름 부여

각 화면의 데이터 공급원이 될 Query 이름을 정한다. 이 이름이 **aggregate-designer의 Exposed Queries와 매칭**된다.

```
[Q · Query 이름 확정]

"내 신청 목록" 화면의 데이터 공급 Query를 다음과 같이 제안:

  listMyLeaveRequests(employeeId, status?, range?)

1) 이대로 OK
2) 이름 수정
3) 매개변수 수정
4) 이 화면은 2개 이상의 Query가 필요함 (분리)
```

**주의**: Query는 **BC 권위 데이터만** 반환. 타 BC의 데이터(예: 잔여 연차)는 별도 Query로 분리해 합성.

### Step 4 — 타 BC 의존 식별

한 화면이 여러 BC 데이터를 필요로 하면 명시:

```
[Q · 타 BC 의존]

"내 신청 목록" 화면은 다음 BC 데이터도 필요합니다:

- Leave Balance BC의 `getBalance(employeeId)` — 잔여 연차 표시용

이 의존 관계를 기록할까요?
1) 기록
2) 이 화면에서는 잔여 연차 제외 (별 화면에서만)
```

### Step 5 — 범위 밖 화면 명시

"이 BC의 범위가 아닌" 화면을 분명히 마킹:

```
[Q · 범위 밖 화면]

혹시 이 BC에 속한 줄 알았지만 범위 밖인 화면이 있나요?

예:
- 관리자 통계 대시보드 → Later (현재 범위 밖)
- 캘린더 뷰 → 다른 BC(Leave Balance)의 Screen Inventory

1) 없음
2) 있음 (목록 제공)
```

---

## 5. 산출물

기본 경로 `docs/shared/contexts/<bc>/_screens.md`. 프로젝트 관례가 다르면 사용자에게 확인.

**Frontmatter 없음.** 본문만.

```markdown
# Screen Inventory — <BC Name>

## 범위
이 BC가 **기여하는** 화면만 기재. 한 화면이 여러 BC의 데이터를 섞어
보여주면 각 BC의 Screen Inventory에 동일 화면이 등장할 수 있음 (관점이 다름).

## 화면 목록

### S1. 내 신청 목록
- **경로**: `/me/leave-requests`
- **주 액터**: 사원
- **표시 데이터**:
  - (이 BC) 신청 ID, 기간, 휴가 종류, 상태, 제출일
  - (Leave Balance 의존) 잔여 연차
- **액션**: 상세 보기, 취소 신청
- **필터/정렬**: 상태별, 최근순 기본
- **Query**: `listMyLeaveRequests(employeeId, status?, range?)`
- **타 BC 의존**: Leave Balance.`getBalance(employeeId)`

### S2. 내 결재 대기함
- **경로**: `/leave-requests?assigned-to-me`
- **주 액터**: 결재자 (팀장·관리자)
- **표시 데이터**: 신청자, 기간, 휴가 종류, 제출일, 사유 요약
- **액션**: 승인, 반려(사유 필수), 상세 보기
- **Query**: `listPendingApprovals(approverId)`

(이하 화면들)

## 범위 밖
- 관리자 통계 대시보드 — Later
- 캘린더 뷰 — Leave Balance BC의 Screen Inventory 참조

## ⚠️ 미정
- S3 "신청 작성"에서 결재선 미리보기 API 구조

## 다음 단계

→ `aggregate-designer`의 "Exposed Queries" 섹션에서 위 Query 이름들을 구현.
   누락된 Query가 생기면 이 파일과 Aggregate 명세를 동기화.
```

---

## 6. 업데이트 모드 (`--update`)

1. `_canvas.md` `updated_at` 비교 → stale 경고 (Canvas가 바뀌면 Inventory도 바뀔 수 있음).
2. "무엇을 업데이트?" 숫자 메뉴:
   ```
   1) 화면 추가
   2) 기존 화면의 표시 데이터·액션 수정
   3) Query 이름 변경 (aggregate-designer와 동기화 필요)
   4) 화면 제거
   ```
3. 해당 부분만 delta 질문.

**주의**: Query 이름 변경은 aggregate-designer와 반드시 동기화. 사용자에게 "aggregate 파일도 업데이트할까요?" 안내.

---

## 7. 강제 체크리스트

```
□ Step 0에서 UI 존재 확인됨
□ UI 없으면 파일 생성 없이 종료
□ 화면마다 경로·주 액터·표시 데이터·액션 모두 기재
□ 화면마다 Query 이름 부여됨
□ 타 BC 의존은 명시됨 (해당 BC.QueryName 형식)
□ 범위 밖 화면 명시됨 (없으면 "없음")
□ Frontmatter 없음
```

---

## 8. 맥락별 조정

| 맥락 | 화면 수 (한 BC당) | Query 수 |
|---|---|---|
| 학습 | 1~3 | 2~5 |
| 개인 | 2~5 | 3~8 |
| 사내 | 3~10 | 5~15 |
| B2B | 5~15 | 10~25 |
| B2C | 5~20 | 10~30 |

---

## 9. 절대 하지 말 것

- UI 없는 BC에 Screen Inventory 강제 생성.
- 화면 1개를 여러 BC에서 **같은 관점으로** 중복 기술 (각 BC는 자기 기여분만).
- Query 이름을 aggregate-designer와 동기화 없이 변경.
- 표시 데이터와 DB 스키마를 동일시 (표시는 사용자 관점, 스키마는 저장 관점).
- 와이어프레임 없다고 작성 거부 (자연어만으로 충분).
- Frontmatter 추가.

---

## 10. 왜 이 단계가 필요한가

ddd-workshop 파이프라인은 전통적으로 **command 중심**(event·aggregate·invariant)으로 설계되어 있어, **조회(read) 요구가 누락**되는 실질 문제가 반복 관측됐다.

Screen Inventory는:
- **BC Canvas의 Inbound Queries**를 구체화.
- **Aggregate의 Exposed Queries**에 이름 매칭으로 연결.
- 누락된 read-path를 **와이어프레임 없이도** 강제 노출.

Alberto Brandolini의 Event Storming 공식 산출물 중 "Screen Inventory" 가 본래 존재하며, 이 스킬은 그 관행을 스킬화한 것.
