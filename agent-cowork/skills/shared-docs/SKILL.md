---
name: shared-docs
description: >
  멀티 레포 환경의 공유 문서(공통 PRD, 도메인 규칙, ADR 등)를
  **git submodule**로 관리. 한 스킬 안에서 다음 4동작을 처리:
  init(공유 문서 레포 신규 생성 + submodule 연결), sync(원격 갱신 받기),
  write(문서 작성 → push → PR, GitHub Flow 기반), doctor(흔한 함정 검증).
  경로/브랜치/원격 URL은 **`.gitmodules`를 source of truth로** 읽고, 복수
  submodule 환경의 식별 힌트만 `AGENTS.md`/`CLAUDE.md`의 `## External Tools`
  섹션에 자기 엔트리로 남긴다 — 별도 config 파일은 만들지 않는다. GitHub Flow 기본:
  main + short-lived 브랜치 + PR. 작은 수정은 main 직접 push 허용.
  Triggers: "shared docs", "공유 문서 셋업", "submodule 추가",
  "submodule 갱신", "공유 문서 수정", "공유 문서 PR", "공유 문서 진단",
  "공유 PRD 업데이트", "shared-docs init", "shared-docs sync",
  "shared-docs write", "shared-docs doctor",
  "/agent-cowork:shared-docs". 모노레포 대신 멀티 레포로 쪼개되 PRD나
  도메인 규칙 같이 여러 레포에서 동시에 참조해야 하는 문서를 한 곳에서
  관리하면서 각 레포에 연결할 때 사용.
metadata:
  author: dev-goraebap
  version: "0.1.0"
---

# shared-docs

여러 레포로 쪼개진 프로젝트(프론트/백/모바일/...)에서 **공유되어야 하는 문서**(공통 PRD, 도메인 규칙, 공통 ADR 등)를 하나의 git 레포로 분리하고, 각 프로젝트 레포에 **submodule로 연결**해서 운영한다. 모노레포로 통합하지 않으면서 source-of-truth를 한 곳에 두는 패턴 — 새 협업자는 `git clone --recurse-submodules` 한 줄로 모든 컨텍스트를 받는다.

이 스킬은 다른 트랙 분기 없이 **동작 분기**만 있다 — `init` / `sync` / `write` / `doctor`.

## 사용 시점

| 사용자 발화·상황 | 호출할 동작 |
|---|---|
| "공유 문서 레포 새로 만들자", "submodule로 hr-docs 연결해줘", 새 프로젝트 셋업 중 | `init` (또는 `init --create`) |
| "공유 문서 최신으로 받아줘", "shared docs pull", 작업 시작 전 | `sync` |
| "공유 PRD 수정해서 올려줘", "도메인 규칙에 X 추가하고 PR 만들어줘", 공유 문서 편집 요청 | `write` |
| "공유 문서 상태 점검", "submodule 뭔가 이상해", 협업자가 clone 후 비어있다고 함 | `doctor` |

**호출 방식**:
- **슬래시**: `/agent-cowork:shared-docs <동작> [옵션]` (예: `/agent-cowork:shared-docs sync`)
- **Auto-trigger**: 위 발화 패턴을 만나면 동작을 추론해 자동 발화. 추론이 애매하면 사용자에게 한 번 물어 확정한다 ("init / sync / write / doctor 중 어떤 동작인가요?")

각 동작의 상세 워크플로우는 references/ 분리 — 사용자가 요청한 동작에 해당하는 references만 로드해 토큰 효율 유지.

## 값 해결 (path·URL·branch 등)

이 스킬은 경로·브랜치·원격 URL을 **하드코딩하지 않지만, 별도 config 파일도 만들지 않는다**. 이미 있는 곳(`.gitmodules`, `remote_url` 자체)에서 읽거나 추론하고, 복수 submodule 환경에서 어느 게 shared-docs인지 구분이 필요할 때만 `AGENTS.md`/`CLAUDE.md`의 `## External Tools` 섹션에 자기 엔트리를 한 줄 추가한다 — 이 섹션은 `agent-cowork:draft-public-rules`가 외부 도구·스킬·MCP 인벤토리로 쓰는 자리와 같으며, shared-docs도 submodule 기반 외부 도구이므로 톤이 맞다. shared-docs는 자기 엔트리만 건드리고 다른 도구의 엔트리는 수정하지 않아 역할 겹침이 없다.

자세한 해결 순서·환경변수·CLI 인자는 [references/values.md](references/values.md) 참조.

## 동작

각 동작의 워크플로우·명령·트러블슈팅은 references에 분리되어 있다. 사용자가 어떤 동작을 요청했는지 식별 후 해당 references만 로드한다.

| 동작 | 무엇을 하나 | 상세 |
|---|---|---|
| `init` | 새 공유 문서 레포 생성(선택) + 현재 레포에 submodule 추가 | [references/init.md](references/init.md) |
| `sync` | 원격에서 공유 문서 최신 커밋 받아오기 (`git submodule update --remote`) | [references/sync.md](references/sync.md) |
| `write` | 공유 문서 작성·수정 → push (작은 수정) 또는 브랜치+PR (큰 수정), GitHub Flow 기반 | [references/write.md](references/write.md) |
| `doctor` | 빈 submodule, 브랜치 불일치, .gitmodules 동기화 등 흔한 함정 검증 | [references/doctor.md](references/doctor.md) |

## 워크플로우 기본값

**GitHub Flow** — 단일 long-lived 브랜치 `main` + short-lived feature 브랜치 + PR. 문서 레포에 가장 흔한 단순 모델이라 채택.

| 변경 규모 | 권장 흐름 |
|---|---|
| 작은 수정 (오타, 한 단락 수정) | `main` 직접 push 허용 |
| 큰 변경 (여러 페이지 추가, 구조 변경) | feature 브랜치 + PR |

판단 기준과 명령은 [references/write.md](references/write.md)에 정리.

## 가드레일

모든 동작 시작 전 공통:

| # | 확인 | 실패 시 |
|---|---|---|
| 1 | 현재 디렉토리가 git 저장소 (`git rev-parse --git-dir`) | "git 저장소 안에서 호출하세요." 후 중단 |
| 2 | 값 해결 통과 ([values.md](references/values.md)) — `init`은 입력값 검증, 그 외 동작은 대상 submodule 식별 | 누락 시 친절한 메시지 + 해결 방법 안내 후 중단. doctor는 D8로 잡음 |

동작별 추가 가드레일:

| 동작 | # | 확인 | 실패 시 |
|---|---|---|---|
| `init` | 1 | 마운트 경로(입력받은 `path`)가 비어있거나 미존재 | 사용자 확인: 덮어쓰기 / 다른 path / 취소 |
| `init` | 2 | 원격 인증 ping 통과 (`ssh -T <host>` 또는 `git ls-remote <url>`) | [init.md 인증 섹션](references/init.md#인증) 안내 후 중단 |
| `sync` | 1 | submodule 디렉토리가 clean (`git -C <path> status --porcelain` 빈 결과) | dirty면 stash 또는 write 동작 안내, sync 중단 |
| `sync` | 2 | 부모 레포 working tree가 clean | 경고만 — 사용자가 진행 결정 |
| `write` | 1 | submodule이 브랜치 위에 있음 (detached HEAD 아님) | `git checkout <branch>` 자동 실행 (사용자 확인 후) |
| `write` | 2 | 변경 분석 후 작은/큰 수정 분류를 사용자에게 한 번 확인 | 사용자가 분류를 뒤집을 수 있음 |
| `doctor` | — | 가드레일 없음 (read-only, 자동 수정은 별도 동의) | — |

## 금지 행동

- **경로/브랜치/URL을 본문이나 references에 하드코딩하기** — 팀마다 컨벤션이 다르고 한 번 박힌 기본값은 사실상 강제가 된다. 모두 [values.md](references/values.md)의 해결 순서대로 외부에서 받는다
- **별도 shared-docs config 파일(`.claude/shared-docs.json` 등)을 만들거나 읽기** — 같은 정보를 두 곳에 두면 drift가 생긴다. `.gitmodules`가 이미 source of truth다 (0.6.0 이상)
- **submodule 디렉토리 안에서 부모 레포 명령 실행하기** — CWD를 명확히 분리. 작업 시 `cd <path>`로 들어가고 끝나면 `cd -`로 복귀
- **`main` 브랜치 force-push** — 문서 레포여도 협업자 영향이 큼. 일반 push가 거부되면 큰 수정 흐름(브랜치+PR)으로 전환을 제안할 뿐
- **사용자 확인 없이 PR 자동 생성·머지** — write 동작이 PR 생성까지는 도와도 머지 버튼은 사용자 손에 둔다. 머지 후 부모 레포 bump도 사용자 동의 받기
- **인증 누락 상태로 init 진행** — SSH 키/HTTPS credential 검증 없이 시작하면 중간에 실패해 부분 상태가 남는다. 사전 ping 필수
- **사용자가 만든 기존 CLAUDE.md/CLAUDE.local.md/.claude/* 등을 임의 수정** — 이 스킬은 submodule + 공유 문서만 다룬다. 다른 영역은 건드리지 않는다
- **doctor의 자동 수정을 동의 없이 적용** — 자동 수정 가능 항목이라도 사용자가 명시 선택해야만 실행
- **호스트 API(GitLab/GitHub)로 빈 레포 자동 생성** — 인증·권한 복잡도 회피. 사용자가 호스트에서 빈 레포를 직접 만든 뒤 URL을 config에 입력하는 흐름 유지
