# 서울시 건설알림이 MCP

서울시 열린데이터광장의 건설공사(공사장) 정보를 자연어로 검색·조회할 수 있게
해주는 MCP(Model Context Protocol) 서버. Claude Desktop, Claude Code, Claude
웹(커넥터)에 연결하면 "서초구에 지금 진행 중인 공사 뭐 있어?" 같은 질문에
서울시 공식 데이터로 답할 수 있다.

## 상태

✅ **로컬 전용(stdio) 서버로 운영 중.** 별도 클라우드 서버나 URL 접속이 없고,
인증키·사용량 제한도 없다. 각자 자기 컴퓨터에 이 저장소를 내려받아 Claude가
직접 실행하는 방식이다. (과거 fly.io로 웹 배포했던 버전은 2026-09에 완전히
폐쇄했다 — 더 이상 어떤 URL로도 서비스되지 않는다.)

## 제공 도구 (4개)

| 도구 | 하는 일 |
|---|---|
| `search_construction_projects` | 자치구명·키워드로 공사장 검색 (사업명, 위치, 발주처/시공사, 착공일, 준공예정일, 도급액, 연락처 등) |
| `search_construction_work_by_district` | 자치구별 공사 정보 검색 (프로젝트 코드, 착수일, 사업기간, 진행상태, 주소 등) |
| `get_construction_progress` | 공정률·기성률·D-Day 등 진행 현황 조회 |
| `get_construction_project_photos` | 사업코드로 현장 사진 목록 조회 |

> 모든 도구는 결과에 `출처` 필드(서울 열린데이터광장 데이터셋명)를 포함하며,
> 이 MCP로 얻은 정보를 답변에 쓸 때는 반드시 출처를 함께 밝혀야 한다.

## 사용하는 원본 데이터

- 서울시 건설알림이 사업개요 ([OA-15585](https://data.seoul.go.kr))
- 서울시 건설알림이 공사사진 (OA-15586)
- 서울시 건설 알림이 정보, ListConstructionWorkService (OA-1222)
- 서울시 건설공사 추진 현황, ListOnePMISBizInfo (OA-2540)

---

## 설치 방법 (비개발자용, 처음이어도 괜찮습니다)

준비물은 딱 두 가지입니다: **Node.js**와 **서울 열린데이터광장 인증키**(무료,
누구나 즉시 발급).

### 0단계 — 사전 준비

**Node.js 설치 확인**

이미 Claude Code나 다른 MCP를 써보셨다면 대부분 설치되어 있습니다. PowerShell
(윈도우) 또는 터미널(맥)에서 아래 명령을 입력해보세요.

```
node --version
```

`v18` 이상 버전이 뜨면 통과입니다. "명령을 찾을 수 없다"는 에러가 뜨면
[nodejs.org](https://nodejs.org)에서 LTS 버전을 내려받아 설치한 뒤 다시
시도하세요.

**서울 열린데이터광장 인증키 발급**

1. [data.seoul.go.kr](https://data.seoul.go.kr)에서 회원가입/로그인
2. [인증키 신청 페이지](https://data.seoul.go.kr/together/mypage/actkeyMain.do)에서 인증키 신청 (승인까지 보통 즉시~수 분)
3. 발급된 인증키(영문+숫자 조합 문자열)를 복사해둔다 — 아래 2단계에서 씀

### 1단계 — 이 저장소 내려받기

PowerShell(윈도우) 또는 터미널(맥)에서, 원하는 위치로 이동한 뒤:

```
git clone https://github.com/hlucent/seoul-construction-mcp.git
cd seoul-construction-mcp
npm install
```

git이 없다면 [git-scm.com](https://git-scm.com)에서 설치하거나, GitHub 페이지
우측의 "Code → Download ZIP"으로 내려받아 압축을 풀어도 됩니다.

`npm install` 실행 후, 지금 폴더의 전체 경로를 메모해두세요. 다음 단계에서
그대로 씁니다. 확인 방법:

```
# 윈도우
echo %cd%

# 맥
pwd
```

### 2단계 — Claude에 등록하기

아래 세 가지 중 본인 환경에 맞는 방법 하나만 따라 하면 됩니다.

#### 방법 A — Claude Desktop (가장 많이 씀)

1. Claude Desktop 실행 → 설정(Settings) → Developer → "Edit Config" 클릭
   (또는 아래 파일을 직접 텍스트 편집기로 연다)
   - 윈도우 (일반 설치): `%APPDATA%\Claude\claude_desktop_config.json`
   - 윈도우 (Microsoft Store 설치): `%LOCALAPPDATA%\Packages\Claude_<임의문자열>\LocalCache\Roaming\Claude\claude_desktop_config.json`
     (탐색기 주소창에 `%LOCALAPPDATA%\Packages`를 붙여넣으면 `Claude_`로
     시작하는 폴더를 찾을 수 있다. **일반 경로를 수정했는데 Claude가 계속 옛날
     설정을 쓰는 것처럼 보인다면, Microsoft Store 버전이라 이 경로가 진짜일
     가능성이 높다** — 실제로 이 프로젝트에서 겪었던 문제다.)
   - 맥: `~/Library/Application Support/Claude/claude_desktop_config.json`
2. 파일 안 `"mcpServers"` 객체 안에 아래 항목을 추가한다. 이미 다른 MCP가
   등록되어 있다면 그 옆에 콤마(`,`)로 이어붙인다. **경로(`args`)는 반드시
   1단계에서 메모해둔 본인 폴더의 실제 경로로 바꿔야 한다.**

   ```json
   "seoul-construction-mcp": {
     "command": "node",
     "args": [
       "여기에_본인_경로/seoul-construction-mcp/src/index.js"
     ],
     "env": {
       "SEOUL_OPENAPI_KEY": "본인의_서울시_인증키"
     }
   }
   ```

   윈도우 경로는 `\`를 두 번(`\\`) 써야 합니다. 예:
   `"C:\\Users\\내계정\\seoul-construction-mcp\\src\\index.js"`

   > **연결이 자꾸 실패한다면?** `"command": "node"` 대신 node.exe의 전체
   > 경로를 직접 써보세요. PowerShell에서 `(Get-Command node).Source`로
   > 확인할 수 있습니다 (보통 `C:\\Program Files\\nodejs\\node.exe`).

3. 파일 저장 → Claude Desktop을 **완전히 종료했다가** 다시 실행한다.
   (창을 닫는 것만으로는 부족할 수 있다. 작업표시줄 트레이 아이콘에서 종료하거나,
   윈도우는 PowerShell에서 `Get-Process | Where-Object { $_.ProcessName -like "*Claude*" } | Stop-Process -Force`로 완전히 끈 뒤 재실행하면 확실하다.)
4. 새 대화창을 열어 "서초구 진행 중인 공사 알려줘"처럼 물어보고 확인한다.

#### 방법 B — Claude Code (CLI를 이미 쓰고 있다면)

터미널에서 프로젝트 폴더 어디서든 아래 한 줄로 등록할 수 있습니다.

```
claude mcp add seoul-construction-mcp -s user -- node 본인_경로/seoul-construction-mcp/src/index.js
```

인증키는 실행할 때 환경변수로 넘기거나, 위 방법 A의 `env` 방식처럼
`.mcp.json`에 직접 넣어도 됩니다. 자세한 옵션은 `claude mcp add --help` 참고.

#### 방법 C — Claude 웹(claude.ai) 커넥터로 쓰고 싶다면

claude.ai의 커넥터는 기본적으로 URL(원격 서버) 연결만 지원하고, 이 프로젝트처럼
내 컴퓨터에서 직접 실행하는 stdio 방식은 등록할 수 없습니다. 웹에서 쓰고
싶다면 별도로 웹 서버로 배포해야 하는데(예: Fly.io 등), 이 저장소는 현재
로컬 전용 구조라 별도 작업이 필요합니다. 개인/사무실 용도로는 방법 A(Desktop)나
B(Code)를 권장합니다.

### 3단계 — Claude Code로 설치 자체를 대신 시키기 (제일 쉬운 방법)

위 단계가 복잡하게 느껴진다면, 이 저장소를 내려받은 뒤(1단계까지만) Claude
Code를 열고 이렇게 요청하세요.

```
CLAUDE.md 읽고, 이 프로젝트를 Claude Desktop에 등록하는 것까지 도와줘.
내 서울시 인증키는 [여기에 붙여넣기]야.
```

Claude Code가 본인 컴퓨터의 실제 경로를 확인하고, 설정 파일 위치를 찾아서
(Microsoft Store 버전인지도 자동으로 확인), 등록까지 대신 진행해준다.

---

## 문제 해결

| 증상 | 확인할 것 |
|---|---|
| 연결이 "Connection closed"로 계속 실패 | ① 경로에 오타/상대경로는 없는지 ② `command`를 node 전체경로로 바꿔봤는지 ③ Claude Desktop을 완전 종료 후 재시작했는지 |
| 설정을 분명히 고쳤는데 반영이 안 됨 | Microsoft Store 버전이면 `%APPDATA%\Claude`가 아니라 `%LOCALAPPDATA%\Packages\Claude_...\LocalCache\Roaming\Claude` 경로가 진짜 설정 파일일 수 있음 |
| "서울시 인증키가 없다"는 응답 | `env`의 `SEOUL_OPENAPI_KEY` 값이 비어있거나 오타 — 발급받은 키를 다시 확인 |
| 도구는 뜨는데 결과가 빈 목록 | 자치구명 표기(예: "서초구" vs "서초"), 키워드 철자를 다시 확인 |

## 개발/기여용 참고

이 저장소를 직접 수정하고 싶다면 `CLAUDE.md`(작업 원칙)와 `DEVLOG.md`(개발
일지)를 참고하세요. 서버 코드는 `src/index.js` 한 파일이며, MCP SDK의
`StdioServerTransport`를 사용하는 순수 stdio 서버입니다(웹 서버, 인증, 사용량
제한 로직 없음).

로컬에서 코드를 고친 뒤 직접 동작을 확인하려면:

```
cp .env.example .env
# .env를 열어 SEOUL_OPENAPI_KEY 값을 채운다
npm start
```
