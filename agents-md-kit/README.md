# agents-md-kit

**AGENTS.md 작성·큐레이션·유지보수**를 위한 도구 모음. 도구 중립 표준인 [AGENTS.md](https://agents.md)를 중심으로, 네이티브 지원이 없는 Claude Code/Gemini CLI용 브릿지도 함께 다룹니다.

## 이 플러그인의 설계 원칙

- **Earn its place.** 모든 섹션은 에이전트가 놓치거나 잘못 추측할 내용만 담는다. 빈 placeholder, 상투어, `package.json`/README/`git log`에서 읽을 수 있는 내용은 쓰지 않는다.
- **Curate, don't enumerate.** `## External Tools`는 설치된 모든 도구의 덤프가 아니다. 이 프로젝트에 실제로 필요한 것만 담는다.
- **Draft-first.** 분석 → 완성 초안 → 사용자 비판. 순차 인터뷰보다 한 번의 리뷰 루프가 낫다.
- **최소로 시작.** 필수 섹션은 2개뿐. 나머지는 실제 내용이 있을 때만 등장한다.

## 설치

### Claude Code (플러그인)

```
/plugin marketplace add dev-goraebap/my-agent-kit
/plugin install agents-md-kit@dev-goraebap
```

### 그 외 에이전트 (`skills.sh`)

```bash
# 전체 설치
npx skills add dev-goraebap/my-agent-kit

# 특정 스킬만
npx skills add dev-goraebap/my-agent-kit --skill draft-agents-md
```

## 포함된 스킬

### `draft-agents-md` (v0.1.0)

현재 프로젝트를 분석해 **AGENTS.md 초안**을 생성하고, 네이티브로 읽지 못하는 에이전트(Claude Code, Gemini CLI)용 브릿지 파일을 함께 만듭니다. 이름 그대로 **초안**이며, 이후 후속 스킬로 계속 다듬어나가는 것을 전제로 설계됐습니다.

**주요 동작:**

- **프로젝트 자동 분석** — `package.json` / `pyproject.toml` / `README.md` / `git log` / `.cursorrules` 등에서 신호 수집
- **스킬·MCP 인벤토리 스캔** — 전역·프로젝트 레벨에서 사용자가 접근 가능한 도구를 수집. Claude Code(skills.sh + 플러그인), Cursor, Windsurf, Gemini CLI, Codex CLI의 MCP 설정 경로를 모두 고려
- **큐레이션** — 인벤토리 중 프로젝트에 실제로 필요한 것만 `## External Tools`에 포함. 전역 도구는 분석 관련성이 확인될 때만 사전 선택
- **Draft-first 리뷰 루프** — 완성 초안을 제시하고 사용자 비판("여기를 이렇게 고쳐")에 따라 재작성. 인터뷰를 최소화
- **모드 A/B 자동 분기** — 신호가 충분하면 바로 초안, 빈약하면 "뭘 만들려고 하세요?" 한 질문만
- **브릿지 자동 생성** — Claude Code용 `CLAUDE.md` 또는 `.claude/CLAUDE.md`(위치는 사용자 선택) + Gemini CLI용 설정 가이드 출력
- **최소 2개 섹션** — `# Project Name` + overview, 빈 `## Boundaries` 3-tier 스켈레톤. 나머지(`External Tools`, `Git Workflow`, `References`, `Project Structure`, `Code Style`, `Testing`)는 실제 내용이 있을 때만 조건부 생성

**트리거 예시:**

```
AGENTS.md 만들어줘
이 프로젝트용 에이전트 규칙 생성해줘
/draft-agents-md
```

자세한 설계와 워크플로우는 [SKILL.md](skills/draft-agents-md/SKILL.md) 참조.

## 로드맵

이 플러그인에 추가 예정인 스킬:

- **Boundaries 자동 누적** — 에이전트가 실수했을 때 교정 내용을 `## Boundaries`의 Always/Ask/Never tier 중 적절한 곳에 누적. Stop hook 연동 실험 포함.
- **스킬/MCP 정리** — `AGENTS.md`에 선언된 External Tools를 source of truth로 삼아 전역에 남아있는 미사용 도구 식별·정리 제안.
- **섹션별 관리 스킬** — `## References`, `## Git Workflow` 등 각 섹션의 점진적 업데이트 지원.

## 라이선스

[MIT](../LICENSE).
