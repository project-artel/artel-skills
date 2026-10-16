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
npx skills add project-artel/artel-skills --skill artel-game-functions
npx skills add project-artel/artel-skills --list
```

## skill 목록

| skill | 하는 일 |
| --- | --- |
| [`artel-unity-sdk`](skills/artel-unity-sdk/SKILL.md) | Unity 게임에 ARTEL SDK 를 설치하고, 개발 빌드를 만들어 서버에 등록합니다 |
| [`artel-cli`](skills/artel-cli/SKILL.md) | `artel` CLI 를 설치하고 로그인한 뒤, test case·시나리오를 만들고 터미널에서 QA run 을 돌립니다 |
| [`artel-mcp`](skills/artel-mcp/SKILL.md) | 로컬 MCP 서버를 띄워 Claude Code 나 Codex 가 게임을 직접 관측하고 조작하게 합니다 (prototype) |
| [`artel-game-functions`](skills/artel-game-functions/SKILL.md) | 게임에 치트(`[ArtelCheat]`)와 behavior(`[ArtelBehavior]`)를 구현해 QA agent 가 씬 이동, 적 처치, 엔딩 같은 상태를 바로 만들게 합니다 |

처음이라면 `artel-unity-sdk` → `artel-cli` → `artel-game-functions` 순서로 쓰면 됩니다.

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

- `artel-game-functions` 가 쓰는 SDK 기능(치트와 behavior)은 project-artel/artel-sdk#142 에 있습니다.
  그 PR 이 merge 되기 전에는 skill 에 적힌 branch 로 SDK 를 설치해야 합니다.
- `artel-mcp` 는 prototype 인 project-artel/artel-mcp-poc 를 씁니다. 운영 QA 에는 `artel-cli` 를 씁니다.
