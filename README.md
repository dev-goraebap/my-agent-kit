# my-agent-kit

개인용 **AI 에이전트 맞춤 환경**. 프로젝트마다 원하는 방식으로 에이전트(Claude Code, Cursor, Copilot, Codex, Windsurf, Gemini CLI 등)를 제어하기 위한 스킬을 모아두는 작업 공간입니다. 오픈소스 하네스를 필요에 따라 가져다 쓰면서 확장하고, 개인적으로 사용하는 유틸리티도 함께 보관합니다.

## 철학

- **도구 중립이 기본.** 특정 에이전트에 묶이지 않는 [AGENTS.md 표준](https://agents.md)을 중심으로, 네이티브 지원이 없는 에이전트(Claude Code, Gemini CLI 등)에는 얇은 브릿지만 덧붙인다.
- **Curate, don't enumerate.** 각 프로젝트에는 실제로 필요한 도구만 선언한다. 전역에 설치된 모든 스킬/MCP를 쏟아붓지 않는다.
- **Draft-first.** 한 번에 완벽한 결과물을 만들지 않는다. 초안을 만들고 이터레이션으로 다듬는 흐름을 선호한다.
- **이중 배포, 자체 updater 없음.** Claude 사용자는 마켓플레이스 플러그인으로, 그 외 에이전트는 `skills.sh`(`npx skills`)로 설치한다. 업데이트는 각 배포 채널의 메커니즘에 위임 — 자체 updater 스킬은 만들지 않는다.

## 플러그인

이 저장소는 마켓플레이스 **`dev-goraebap`** 역할을 하며, 다음 플러그인을 포함합니다:

| 플러그인 | 설명 | 문서 |
|---|---|---|
| `agents-md-kit` | AGENTS.md 작성·큐레이션·유지보수 도구 모음 (공개 트랙) | [README](agents-md-kit/README.md) |
| `misc` | 개인 잡동사니 스킬 모음 (개인 트랙) | [README](misc/README.md) |

각 플러그인의 설치 방법과 포함된 스킬은 해당 README를 참고하세요.

## 설치

사용하는 에이전트에 따라 두 가지 방식으로 설치할 수 있습니다.

### Claude Code — 마켓플레이스 등록

```
/plugin marketplace add dev-goraebap/my-agent-kit
```

등록 후에는 원하는 플러그인만 개별 설치합니다 (예: `/plugin install agents-md-kit@dev-goraebap`). 플러그인 단위로 업데이트를 Claude Code가 자동 관리합니다.

### 그 외 에이전트 — `skills.sh`

스킬 단위로 설치합니다. 플러그인 구조와 무관하게 필요한 스킬만 가져올 수 있습니다.

```bash
# 특정 스킬만 설치
npx skills add dev-goraebap/my-agent-kit --skill <skill-name>

# 예시
npx skills add dev-goraebap/my-agent-kit --skill draft-agents-md
npx skills add dev-goraebap/my-agent-kit --skill pdf-parser

# 전역 설치(-g)
npx skills add dev-goraebap/my-agent-kit --skill <skill-name> -g
```

업데이트가 필요하면 같은 명령을 다시 실행합니다.

각 플러그인이 번들하는 스킬 목록과 상세한 사용법은 플러그인 README(위 표)를 참고하세요.

## 라이선스

[MIT](LICENSE).
