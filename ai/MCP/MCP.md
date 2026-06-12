# MCP (Model Context Protocol)
> AI 애플리케이션을 외부 시스템에 연결하기 위한 오픈 소스 표준

MCP를 사용하면 Claude나 ChatGPT 같은 AI 애플리케이션이 로컬 파일, 데이터베이스, 검색 도구, 업무 도구 같은 외부 시스템에 연결될 수 있다.

즉, MCP는 AI 애플리케이션과 외부 시스템을 연결하는 표준 방식이다.

![MCP 구조](<MCP 1.png>)

## 어디에 사용되나?

아래는 예시일 뿐, 이외에도 다양한 곳에 사용될 수 있다.

- 사용자의 Google 캘린더와 Notion에 접근해 일정, 메모, 업무 내용을 바탕으로 개인화된 AI 비서 역할을 수행할 수 있다.
- Claude Code는 Figma MCP 서버를 통해 디자인 정보를 읽고, 이를 바탕으로 웹 앱을 생성할 수 있다.
- 기업용 챗봇은 조직 내부의 데이터베이스, 문서 저장소, 업무 시스템에 연결되어 데이터를 조회하고 분석할 수 있다.
- 개발자는 AI를 GitHub, Jira, Slack과 연결해 이슈 확인, 코드 리뷰, 작업 내역 정리 등을 자동화할 수 있다.

## MCP가 중요한 이유

- 개발자: 외부 시스템 연동 방식을 매번 새로 만들 필요가 줄어든다.
- AI 애플리케이션 또는 에이전트: 파일, DB, API 같은 외부 도구를 사용할 수 있다.
- 최종 사용자: 자신의 데이터와 도구를 AI와 연결해 더 실용적인 작업을 수행할 수 있다.

## 실습

### filesystem MCP 사용해보기

Claude Desktop에서 filesystem MCP를 연결해 로컬 파일을 읽어보자.

`npx`로 filesystem MCP 서버를 실행하기 때문에 Node.js를 준비한다.

#### 1. 실습용 폴더 생성

filesystem MCP가 접근할 폴더를 만든다.

```bash
mkdir -p ~/mcp-test
echo "MCP 실습용 메모입니다." > ~/mcp-test/memo.txt
```

#### 2. Claude Desktop 설정 열기

Claude Desktop에서 설정 파일을 연다.

```text
Claude Desktop 실행
→ Settings
→ Developer
→ Edit Config
```

![Claude Desktop 개발자 설정](<MCP 2.png>)

설정 파일 위치는 보통 `~/Library/Application Support/Claude/claude_desktop_config.json`이다.

![Claude Desktop 설정 파일](<MCP 3.png>)

#### 3. filesystem MCP 설정 추가

`claude_desktop_config.json`에 아래 설정을 추가한다.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/<username>/mcp-test"
      ]
    }
  }
}
```

filesystem MCP는 설정한 폴더 안에서만 파일을 읽거나 쓸 수 있다.

#### 4. Claude Desktop 재시작

설정 파일을 저장한 뒤 Claude Desktop을 완전히 종료하고 다시 실행한다.

#### 5. Claude에서 확인

Claude를 통해 `mcp-test` 폴더와 저장되어 있는 `memo.txt`에 접근을 시도해본다.

![filesystem MCP 실행 결과](<MCP 4.png>)

접근 제한을 확인하기 위해 설정하지 않은 폴더에도 접근을 시도해본다.

![허용되지 않은 폴더 접근 실패](<MCP 5.png>)

> 파일 읽기뿐 아니라 파일 생성, 수정도 가능하다.

#### 동작 흐름

Claude가 로컬 파일을 원래 알고 있는 것은 아니다.

1. filesystem MCP 서버가 허용된 폴더에 접근한다.
2. Claude는 filesystem MCP 서버가 제공하는 도구를 호출한다.
3. 필요한 경우 사용자가 권한을 승인한다.
4. 승인된 작업만 로컬 파일에 수행된다.

### GitHub MCP 사용해보기

GitHub MCP를 사용하면 저장소, 이슈, PR 등을 조회할 수 있다.

Docker로 GitHub MCP 서버를 실행하기 때문에 Docker Desktop을 준비한다.

#### 1. GitHub Personal Access Token 발급

GitHub에서 `Personal Access Token`을 발급한다.

```text
GitHub
→ Settings
→ Developer settings
→ Personal access tokens
→ Generate new token
```

토큰은 필요한 저장소와 권한만 선택해서 발급한다.

#### 2. GitHub MCP 설정 추가

`claude_desktop_config.json`에 아래 설정을 추가한다.

```json
{
  "mcpServers": {
    "github": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "발급받은_토큰"
      }
    }
  }
}
```

#### 3. Claude Desktop 재시작

설정 파일을 저장한 뒤 Claude Desktop을 완전히 종료하고 다시 실행한다.

#### 4. Claude에서 확인

Claude에게 GitHub 저장소나 이슈를 조회해달라고 요청한다.

```text
내 GitHub 저장소 목록을 확인해줘.
```

![GitHub MCP 도구 탐색 과정](<MCP 6.png>)

![GitHub MCP 저장소 조회 결과](<MCP 7.png>)

## REFERENCE

- https://modelcontextprotocol.io/docs/getting-started/intro
