# artel-skills

ARTEL 을 쓰는 데 필요한 작업을 AI coding agent(Claude Code, Codex, Cursor 등)가 대신 할 수 있게
해 주는 skill 모음입니다.

## 설치

```bash
npx skills add project-artel/artel-skills
```

- 기본으로는 지금 프로젝트에 설치됩니다. 모든 프로젝트에서 쓰려면 `-g` 를 붙입니다.
- skill 하나만 설치하려면 `--skill <이름>` 을 붙입니다.
- 설치하지 않고 목록만 보려면 `--list` 를 붙입니다.

```bash
npx skills add project-artel/artel-skills --skill artel-cheats
npx skills add project-artel/artel-skills --list
```

## skill 목록

| skill | 하는 일 |
| --- | --- |
| [`artel-unity-sdk`](skills/artel-unity-sdk/SKILL.md) | Unity 게임에 ARTEL SDK 를 설치하고, 개발 빌드를 만들어 서버에 등록합니다 |
| [`artel-cli`](skills/artel-cli/SKILL.md) | `artel` CLI 를 설치하고 로그인한 뒤, test case·시나리오를 만들고 터미널에서 QA run 을 돌립니다 |
| [`artel-mcp`](skills/artel-mcp/SKILL.md) | 로컬 MCP 서버를 띄워 Claude Code 나 Codex 가 게임을 직접 관측하고 조작하게 합니다 (prototype) |
| [`artel-cheats`](skills/artel-cheats/SKILL.md) | 게임에 치트(`[ArtelCheat]`)를 구현해 QA agent 가 씬 이동, 스테이지 해금, 적 처치, 엔딩 같은 상태를 한 번에 만들게 합니다 |
| [`artel-behaviors`](skills/artel-behaviors/SKILL.md) | 게임에 behavior(`[ArtelBehavior]`)를 구현해, QA agent 가 켜 두면 게임 안에서 매 프레임 계속 공격·시전·턴 넘기기를 하게 합니다 |

처음이라면 `artel-unity-sdk` → `artel-cli` → `artel-cheats` → `artel-behaviors` 순서로 쓰면 됩니다.

각 skill 은 필요한 문서를 자기 폴더 안에 둡니다. `npx skills add` 는 `skills/<이름>/` 폴더만 설치하므로, skill 이 읽어야 하는 문서가 폴더 밖에 있으면 설치한 쪽에서 읽을 수 없기 때문입니다.

| skill | 폴더 안의 문서 |
| --- | --- |
| `artel-cheats` | `references/choosing-cheats.md` (어떤 치트를 만들고 어떤 것은 만들지 않는가), `references/reference.md` (attribute 규칙, wire, 실패 메시지), `references/compile-guard.md` (`#if ARTEL_GAME_FUNCTIONS` 로 감싸는 이유와 SDK 가 define 을 관리하는 방식), `references/wordventure-example.cs` |
| `artel-behaviors` | `references/choosing-behaviors.md` (어떤 behavior 를 만들고 어떤 것은 만들지 않는가), `references/lifecycle.md` (끝나는 이유 9가지, wire, 실패 메시지), `references/wordventure-example.cs` |
| `artel-unity-sdk`, `artel-cli`, `artel-mcp` | 원본 문서가 자주 바뀌어서 복사하지 않고, 끝의 `References` 절에서 원본으로 링크합니다 |

## 다음에 만들 skill

| 후보 | 하는 일 | 근거 문서 |
| --- | --- | --- |
| `artel-test-cases` | 기획서에서 test case 와 시나리오를 만들고, 일부러 실패해야 하는 step 의 기대값까지 붙여 등록 | [CLI README](https://github.com/project-artel/artel-cli/blob/main/README.md), [benchmarks README](https://github.com/project-artel/artel/blob/main/benchmarks/README.md) |
| `artel-qa-results` | QA run 결과를 정답지와 비교해 읽고, `steps_passed` 를 점수로 읽지 않게 함 | [benchmarks README](https://github.com/project-artel/artel/blob/main/benchmarks/README.md) |
| `artel-local-stack` | orchestration, agent-server, 게임을 이 컴퓨터에서 띄워 실제 QA run 을 돌림 | [qa-run-local skill](https://github.com/project-artel/artel/blob/main/.agents/skills/qa-run-local/SKILL.md) |
| `artel-ci` | CI 에서 `ARTEL_TOKEN` 으로 빌드를 띄우고 QA 결과로 job 을 통과·실패시킴 | [CLI README](https://github.com/project-artel/artel-cli/blob/main/README.md) |
| `artel-self-host` | Docker Compose 로 ARTEL 을 직접 호스팅 | [deploy README](https://github.com/project-artel/artel/blob/main/deploy/README.md) |

skill 로 만들 만한 것은 단계가 여러 개이고, 조용히 실패하고, 반복해서 요청되는 일입니다. 실제로 한 번 해 보고, 겪은 문제를 증상·원인·해결로 적은 뒤 skill 로 만듭니다.

## 구조

[`skills`](https://github.com/vercel-labs/skills) CLI 가 찾는 구조를 따릅니다.

```text
skills/
  <skill 이름>/
    SKILL.md          # YAML frontmatter 의 name 과 description, 그리고 agent 가 따를 절차
    references/       # 필요할 때만 읽는 예시 파일
```

skill 을 추가할 때는 `skills/<이름>/SKILL.md` 를 만들고, frontmatter 의 `name` 을 디렉터리 이름과
같게 씁니다. `SKILL.md` 는 agent 가 읽는 문서라 영어로 쓰고, 이 README 는 사람이 읽는 문서라
한국어로 씁니다.

## 참고

- `artel-cheats` 와 `artel-behaviors` 가 쓰는 SDK 기능은 project-artel/artel-sdk#142 에 있습니다.
  그 PR 이 merge 되기 전에는 skill 에 적힌 branch 로 SDK 를 설치해야 합니다.
- `artel-mcp` 는 prototype 인 project-artel/artel-mcp-poc 를 씁니다. 운영 QA 에는 `artel-cli` 를 씁니다.
