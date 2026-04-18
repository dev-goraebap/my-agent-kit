# values

shared-docs 스킬은 동작에 필요한 값(path, remote_url, branch, pr_tool 등)을 **별도 config 파일에 저장하지 않는다**. 이미 있는 곳(`.gitmodules`, `remote_url` 자체)에서 읽거나 추론하고, 필요한 최소한의 힌트만 `AGENTS.md`/`CLAUDE.md`의 `## References` 섹션에 남긴다.

## 해결 순서 (위가 강함)

1. **CLI 인자** — 1회성 override (예: `--branch=develop`)
2. **환경 변수** — CI 환경용 (예: `SHARED_DOCS_PATH`)
3. **`.gitmodules`** — submodule path/URL/branch의 source of truth
4. **`remote_url` 기반 추론** — pr_tool, host_url
5. **기본값** — small_edit_threshold=20 등
6. **사용자 인터뷰** — 여전히 누락된 키만 한 번씩 물음

---

## 값별 해결 방법

### path / remote_url / branch

submodule 관련 3개 값은 **이미 `.gitmodules`에 저장**되어 있으므로 파일에서 직접 읽는다.

```bash
git config --file .gitmodules --get-regexp "submodule\..*\.path"
git config --file .gitmodules --get-regexp "submodule\..*\.url"
git config --file .gitmodules --get-regexp "submodule\..*\.branch"
```

별도 "shared-docs config 파일"은 **만들지도, 읽지도 않는다** — 같은 정보를 두 곳에 두면 drift가 생긴다.

#### 복수 submodule 환경

프로젝트에 submodule이 여러 개면 어느 게 shared-docs인지 식별이 필요하다. 순서:

1. **단일 submodule** → 그 submodule을 자동으로 shared-docs로 간주 (묻지 않음).
2. **복수 submodule** → 루트 `AGENTS.md`(우선) 또는 `CLAUDE.md`의 `## References` 섹션에서 **`Shared docs submodule: <path>`** 형태의 힌트를 찾는다.
3. **힌트 없음** → 사용자에게 한 번 물음:

    ```
    submodule이 여러 개 있습니다:
      - docs/shared
      - docs/design-system

    shared-docs로 사용할 submodule은 어느 것인가요?
    ```

4. 답 받으면 `## References` 섹션에 한 줄 추가를 제안 (섹션이 없으면 문서 끝에 신설):

    ```markdown
    ## References
    ...
    - Shared docs submodule: `docs/shared`
    ```

    사용자 동의 후 기록. 이후 호출부터는 재질문 없이 같은 값을 사용한다.

### init 시점 (submodule이 아직 없음)

`init` 동작은 `.gitmodules`가 아직 없을 때도 호출되므로 값을 CLI 인자 또는 인터뷰로 받는다.

```
/agent-cowork:shared-docs init \
  --path=docs/shared \
  --remote-url=git@github.com:org/hr-docs.git \
  --branch=main
```

CLI 인자가 없으면 순서대로 인터뷰:

1. "공유 문서 submodule 경로는? (예: `docs/shared`)"
2. "원격 URL은?"
3. "추적 브랜치는? (기본값: `main`)"

`init`이 완료되면 `.gitmodules`가 생성되므로 이후 동작은 그 파일에서 읽는다.

### pr_tool / host_url

`remote_url`에서 추론한다 — 별도 저장 불필요.

| `remote_url` 패턴 | `pr_tool` | `host_url` |
|---|---|---|
| `git@github.com:...` / `https://github.com/...` | `gh` | (불필요) |
| `git@gitlab.com:...` / `https://gitlab.com/...` | `glab` | (glab 기본값) |
| `git@gitlab.<domain>:...` (self-hosted) | `glab` | `https://gitlab.<domain>` |
| 그 외 호스트 | 사용자에게 물음 | 사용자에게 물음 |

CLI 인자로 1회 override 가능: `--pr-tool=glab`, `--host-url=https://...`.

### small_edit_threshold

기본값 **20**. 변경 줄 수가 이 값 이하 + 단일 파일이면 "작은 수정"으로 분류되어 `main` 직접 push 허용. 이보다 크면 브랜치+PR로 자동 전환.

조정은 CLI 인자로만: `--small-edit-threshold=30`. 프로젝트별로 이 값을 영속 저장해야 할 실제 수요가 생기면 향후 `AGENTS.md` 섹션 옵션을 검토.

### 환경 변수 (CI 용)

CI 환경에서는 `.gitmodules`가 아직 없거나 인터뷰가 불가능하므로 환경 변수로 주입한다.

| 변수 | 대응 값 |
|---|---|
| `SHARED_DOCS_PATH` | path |
| `SHARED_DOCS_REMOTE_URL` | remote_url |
| `SHARED_DOCS_BRANCH` | branch |
| `SHARED_DOCS_PR_TOOL` | pr_tool |
| `SHARED_DOCS_HOST_URL` | host_url |

CI 스크립트 예:

```bash
export SHARED_DOCS_PATH=docs/shared
export SHARED_DOCS_REMOTE_URL=git@github.com:org/hr-docs.git
/agent-cowork:shared-docs sync
```

---

## 값 해결 실패 시

각 동작의 가드레일에서 다음을 확인한다.

| 동작 | 필요한 값 | 없을 때 |
|---|---|---|
| `init` | path, remote_url (+ 선택 branch) | CLI 인자 또는 인터뷰로 받음 |
| `sync` | 대상 submodule 식별 | "shared-docs submodule을 찾지 못했습니다. `init`을 먼저 호출하세요." |
| `write` | 대상 submodule + pr_tool | submodule 누락 시 위와 동일. pr_tool 추론 실패 시 사용자에게 물음 |
| `doctor` | 대상 submodule | "shared-docs submodule을 찾지 못했습니다." 후 종료 |

---

## 마이그레이션 (구 config 파일이 있는 사용자)

이 스킬은 0.5.0까지 `.claude/shared-docs.json` 또는 `package.json`의 `"shared-docs"` 필드에서 값을 읽었다. 0.6.0부터는 **무시한다** — 파일을 두어도 에러는 나지 않지만 더 이상 읽히지 않는다.

`.claude/shared-docs.json`이 있는 프로젝트는 다음 단계로 전환:

1. 기존 config의 `path` / `remote_url` / `branch`는 이미 `.gitmodules`에 있으므로 그대로 사용된다. 별도 작업 불필요.
2. 복수 submodule 환경이었다면 `AGENTS.md`의 `## References`에 `Shared docs submodule: <path>` 한 줄 추가.
3. `.claude/shared-docs.json` 파일은 삭제 (또는 `.gitignore`에 추가).
