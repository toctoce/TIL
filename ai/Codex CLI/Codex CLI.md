# Codex CLI

AI Agent는 CLI를 지원하는 경우가 많다. CLI의 사용법에 대해 배우고 싶다면 공식 문서를 찾아보면 된다.

Codex를 CLI 환경에서 사용할 수 있다.

## 설치
> [참고 페이지](https://developers.openai.com/codex/quickstart?setup=cli)

아래 명령어를 실행시키자.

1. `curl -fsSL https://chatgpt.com/codex/install.sh | sh` 명령어로 설치
2. 터미널에서 `codex` 명령어를 실행한 뒤 로그인한다.

![Codex CLI 실행](<Codex CLI 1.png>)

## Command line options
> [참고 페이지](https://developers.openai.com/codex/cli/reference)

### 자주 쓰는 옵션

| 옵션 | 설명 |
| --- | --- |
| `-C`, `--cd` | Codex를 실행할 작업 디렉터리를 지정한다. |
| `-m`, `--model` | 사용할 모델을 지정한다. |
| `-i`, `--image` | 이미지 파일을 프롬프트에 첨부한다. |
| `-s`, `--sandbox` | 명령 실행 권한 범위를 지정한다. |
| `-a`, `--ask-for-approval` | 명령 실행 전에 승인 요청을 언제 받을지 지정한다. |
| `--add-dir` | 현재 작업 디렉터리 외에 추가로 접근할 디렉터리를 지정한다. |
| `--search` | 웹 검색을 활성화한다. |
| `-c`, `--config` | 이번 실행에서만 특정 설정 값을 덮어쓴다. |

```bash
codex -C ~/project
codex -m <model>
codex -i ./image.png "이 이미지 설명해줘"
codex --search
codex -s workspace-write -a on-request
codex --add-dir ../another-project
```

### 자주 쓰는 명령어

| 명령어 | 설명 |
| --- | --- |
| `codex` | 현재 디렉터리에서 Codex TUI를 실행한다. |
| `codex "요청 내용"` | 시작하자마자 요청을 전달한다. |
| `codex app` | Codex 데스크톱 앱을 실행한다. |
| `codex resume` | 이전 대화를 이어서 진행한다. |
| `codex exec "요청 내용"` | 대화형 UI 없이 한 번 실행한다. |
| `codex apply` | Codex Cloud 작업에서 생성된 최신 diff를 로컬에 적용한다. |
| `codex login` | Codex에 로그인한다. |
| `codex logout` | 저장된 인증 정보를 제거한다. |
| `codex doctor` | 설치, 인증, 설정, Git 상태 등을 진단한다. |
| `codex update` | Codex CLI를 업데이트한다. |
| `codex mcp` | MCP 서버를 관리한다. |

```bash
codex
codex "README 정리해줘"
codex exec "테스트 실패 원인 찾아줘"
codex resume
codex doctor
```

## Slash commands
> [참고 페이지](https://developers.openai.com/codex/cli/slash-commands)

| 명령어 | 설명 |
| --- | --- |
| `/init` | 현재 디렉터리에 `AGENTS.md` 초안을 만든다. |
| `/model` | 현재 세션에서 사용할 모델을 변경한다. |
| `/status` | 모델, 권한, 토큰, 작업 디렉터리 등 현재 상태를 확인한다. |
| `/diff` | 현재 작업 트리의 변경 내용을 확인한다. |
| `/review` | 현재 변경 사항을 코드 리뷰 관점으로 검사한다. |
| `/mention` | 특정 파일이나 폴더를 대화에 첨부한다. |
| `/plan` | 바로 수정하지 않고 먼저 계획을 세우게 한다. |
| `/permissions` | 명령 실행 권한과 승인 방식을 조정한다. |
| `/mcp` | 연결된 MCP 도구 목록을 확인한다. |
| `/compact` | 긴 대화를 요약해서 컨텍스트를 줄인다. |
| `/resume` | 저장된 대화를 이어서 진행한다. |
| `/new` | 같은 CLI 안에서 새 대화를 시작한다. |
| `/quit`, `/exit` | Codex CLI를 종료한다. |

## REFERENCE
- https://developers.openai.com/codex
- https://developers.openai.com/codex/cli
