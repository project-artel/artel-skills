# 어떤 것을 skill 로 만들면 좋은가

이 문서는 다음에 만들 skill 을 고르는 기준과, 그 기준으로 고른 후보 목록입니다. 각 후보에는 skill 을 쓸 때 근거로 삼을 문서를 링크했습니다.

## 고르는 기준

아래 다섯 가지 중 둘 이상에 해당하면 skill 로 만들 만합니다.

| 기준 | 설명 | 예 |
| --- | --- | --- |
| 단계가 여러 개이고 순서가 정해져 있다 | agent 가 순서를 틀리면 처음부터 다시 해야 합니다 | SDK 설치 → 개발 빌드 → 등록 |
| 조용히 실패한다 | 오류 메시지 없이 결과만 틀립니다. 사람이 겪고 나서야 아는 내용이라 코드에 적혀 있지 않습니다 | 개발 빌드가 아니면 게임이 등록되지 않는데, 로그에는 아무것도 남지 않습니다 |
| 여러 저장소나 도구에 걸쳐 있다 | 한 저장소의 README 만 읽어서는 전체 절차가 나오지 않습니다 | 로컬 stack: orchestration, agent-server, 게임, MinIO |
| 사용자가 반복해서 요청한다 | 같은 설명을 매번 다시 하게 됩니다 | "치트키 만들어 줘", "QA 돌려 줘" |
| 검증 방법이 정해져 있다 | 끝났는지 agent 가 스스로 확인할 수 있는 명령이나 기준이 있습니다 | `GET .../game-functions` 에 치트가 나오는지 |

반대로 이런 것은 skill 로 만들지 않습니다.

- 코드나 `--help` 를 읽으면 바로 알 수 있는 것. 그런 내용은 skill 에 옮기는 순간 원본과 어긋나기 시작합니다.
- 한 번만 하는 일. 문서 하나로 충분합니다.
- 정답이 사람의 판단에 달린 일. 예를 들어 테스트 결과를 결함으로 볼지 판단하는 일은 skill 이 대신할 수 없습니다. 판단에 필요한 정보를 모으는 절차까지만 skill 로 만듭니다.

## 지금 있는 skill

| skill | 근거 문서 |
| --- | --- |
| `artel-unity-sdk` | [SDK README](https://github.com/project-artel/artel-sdk/blob/develop/README.md), [Package README](https://github.com/project-artel/artel-sdk/blob/develop/Packages/kr.artel.sdk/README.md) |
| `artel-cli` | [CLI README](https://github.com/project-artel/artel-cli/blob/main/README.md), [artel.kr/cli.md](https://artel.kr/cli.md) |
| `artel-mcp` | [artel-mcp-poc README](https://github.com/project-artel/artel-mcp-poc/blob/main/README.md) |
| `artel-game-functions` | [SDK protocol](https://github.com/project-artel/artel-sdk/blob/develop/docs/protocol.md), [ADR 0007](https://github.com/project-artel/artel-sdk/blob/develop/docs/adr/0007-game-registered-cheats-and-behaviors.md), WordVenture 구현 |

## 후보

우선순위는 사용자가 겪는 빈도와, skill 이 없을 때 실패가 얼마나 조용한지로 정했습니다.

| 우선순위 | 후보 이름 | 하는 일 | 해당하는 기준 | 근거 문서 |
| --- | --- | --- | --- | --- |
| 1 | `artel-test-cases` | 기획서에서 test case 와 시나리오를 만들고, 일부러 실패해야 하는 step 의 기대값 표시까지 붙여 등록합니다 | 단계 여러 개, 반복 요청, 검증 가능 | [CLI README](https://github.com/project-artel/artel-cli/blob/main/README.md), [benchmarks README](https://github.com/project-artel/artel/blob/main/benchmarks/README.md) |
| 1 | `artel-qa-results` | QA run 결과를 정답지와 비교해 읽습니다. `steps_passed` 를 점수로 읽지 않게 하고, 결함과 오탐을 나눕니다 | 조용한 실패, 반복 요청 | [benchmarks README](https://github.com/project-artel/artel/blob/main/benchmarks/README.md), [WordVenture answer key](https://github.com/project-artel/artel/blob/main/benchmarks/wordventure/answer-key.md) |
| 2 | `artel-local-stack` | orchestration, agent-server, 게임을 이 컴퓨터에서 띄워 실제 QA run 을 돌립니다 | 여러 저장소, 조용한 실패, 검증 가능 | [qa-run-local skill](https://github.com/project-artel/artel/blob/main/.agents/skills/qa-run-local/SKILL.md), [local-stack](https://github.com/project-artel/artel-orchestration-server/blob/develop/.agents/docs/local-stack.md) |
| 2 | `artel-ci` | CI 에서 `ARTEL_TOKEN` 으로 빌드를 띄우고 QA 를 돌려, 결과로 job 을 통과시키거나 실패시킵니다 | 단계 여러 개, 검증 가능 | [CLI README](https://github.com/project-artel/artel-cli/blob/main/README.md) |
| 2 | `artel-content-map` | 게임을 scan 해서 content map 을 채우고, agent 가 무엇을 읽을 수 있는지 확인합니다 | 조용한 실패(map 이 비면 run 은 돌지만 아무것도 읽지 못함), 검증 가능 | [CLI README](https://github.com/project-artel/artel-cli/blob/main/README.md) 의 `doc scan`·`map show`, [Package README](https://github.com/project-artel/artel-sdk/blob/develop/Packages/kr.artel.sdk/README.md) 의 scan 절 |
| 3 | `artel-knowledge` | 기획서와 규칙 문서를 올려 agent 가 검색할 수 있게 합니다 | 단계 여러 개 | [CLI README](https://github.com/project-artel/artel-cli/blob/main/README.md) 의 `doc upload` |
| 3 | `artel-self-host` | Docker Compose 로 ARTEL 을 직접 호스팅하고, 이메일 로그인과 LLM key 를 설정합니다 | 여러 저장소, 조용한 실패 | [deploy README](https://github.com/project-artel/artel/blob/main/deploy/README.md) |
| 3 | `artel-game-functions` 장르별 예시 | 플랫포머, 카드 게임, 실시간 전투 등 장르별로 어떤 치트와 behavior 가 test case 를 줄이는지 예시를 더합니다 | 반복 요청 | WordVenture 구현과 이후 다른 게임의 구현 |

## 후보를 skill 로 만드는 순서

1. 실제로 한 번 해 봅니다. 해 보지 않은 절차를 적으면, 조용히 실패하는 지점을 놓칩니다. `artel-game-functions` 도 WordVenture 에 치트를 넣고 로컬 stack 에서 호출해 본 뒤에 썼습니다.
2. 겪은 문제를 증상, 원인, 해결로 적습니다. 이 부분이 skill 에서 가장 값진 내용입니다.
3. `skills/<이름>/SKILL.md` 를 영어로 씁니다. frontmatter 의 `description` 에 사용자가 실제로 쓰는 말(한국어 포함)을 넣어야 agent 가 그 skill 을 고릅니다.
4. 근거 문서를 `## References` 절에 링크합니다. skill 이 원본을 대신하지 않고, 원본으로 가는 길을 알려 주게 합니다.
5. `npx skills add <checkout> --list` 로 skill 이 보이는지 확인합니다.
