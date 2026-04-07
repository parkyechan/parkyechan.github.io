---
title: MCP & Agents Skills
date: 2026-04-04 15:25 +0800
tags:
  - AI
  - AgenticAI
author: Park, Ye Chan
categories:
  - AI
  - AgenticAI
---
## Tool Calling

기존에 `LLM` 이 존재했는데, 여기다가 1+1이 뭐인지 아나요? 라고 묻는다면 2입니다 라고 답변이 나올 것이다.

`LLM` 한테 답변 뿐만 아니라 실질적인 일을 시키고 싶은데 외부와 소통할 수 있는 방법을 찾고 싶었고, 이런 방식을 `Tool Calling` 이라고 한다. 

여기서 `Tool Calling` 의 문제점은 `Tool Call` 이라는 행위를 하기 위해서는 코드를 작성해야 한다. 하지만, 일반 유저가 사용하기에 어려운 점이 있다.

또한 `A` 라는 툴을 사용하고 싶으면 `Too Call` 을 위한 코드를 작성하고, `B` 라는 툴을 부르기 위해서도.. 이걸 개발하는 개개인의 N명이 M개의 툴을 부를 때 `N*M` 개의 구현제가 생겨서 복잡해진다. 

그렇기에 이 문제를 해결하기 위해 다른 사람이 작성한 코드를 재 사용이 가능하게 만들어야 할 것이다.

그래서 `MCP` 의 개념을 생각했다.

## Model Context Protocoal - MCP 란?

일종의 서버를 만들어서 함수/데이터 소스 등을 연결하는 표준이다. 자세한 내용은 아래의 링크에 존재한다. 

[https://modelcontextprotocol.io/docs/getting-started/intro](https://modelcontextprotocol.io/docs/getting-started/intro)

- MCP Client : `Claude` `ChatGPT` 등의 `MCP` 서버를 사용하는 App 이다. 
- MCP Server : 특정 기능을 추가해주는 서버이다(일단 이렇게 이해하고 넘어간다).

#### 왜 쓰는가? 예시 : 계산기 서버

LLM 이 계산을 자꾸 틀려서 계산을 수행할 수 있는 `MCP` 서버를 붙인다. 그러면 `Claude.app` 에서 `MCP` 서버에 요청해 `Tool` 목록을 받아 온다. 그러면 `Tool` 목록이 나와서 유저가 대화에서 어떤 `Tool` 을 사용할 수 있을 지 선택한다. 

## MCP 서버 추가 실습

### 일정 파일 관리하기

먼저, 아래와 같이 `readme.md` 와 `roadmap.md` 파일 두 개가 `/tmp/docs` 경로에 있다고 하자. 

```
park@bba-2-50-184-66 docs % ls
readme.md	roadmap.md
park@bba-2-50-184-66 docs % cat readme.md
# 수업 자료

수업 자료입니다.
park@bba-2-50-184-66 docs % cat roadmap.md
# Roadmap

## 4월

- AI MCP 구현하기
- Tool call 구현하기

## 5월

- Codex 써보기
- Claude Code 써보기
```

### MCP 서버 설정

위에서 나온 내용은 뒤로 하고 먼저아래의 경로에서 `claude_desktop_config.json` 를 찾아 수정을 해야 한다. 

```
macOS: $HOME/Library/Application Support/Claude/claude_desktop_config.json

Windows: %APPDATA%\Claude\claude_desktop_config.json
```

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills.png)

아래와 같은 데이터를 넣어준다. 이 때 앞서 언급했듯 `/tmp/docs` 경로임을 주의해야 한다. 

여기서 우리가 넣을 이름은 `filesystem` 이다. 

```json
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/park/tmp/docs"
      ],
      "env": {
        "PATH": "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin"
      }
    },
```

최종적으로 아래와 같이 `claude_desktop_config.json` 를 만들 수 있다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-2.png)

### Claude 로컬 MCP 서버 추가

정상적으로 위의 사항이 반영이 됐으면 아래와 같이`filesystem` 에 대해서 `running` 이 나올 것이다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-4.png)

그러면 대화창을 열면 커넥터에서 `filesystem` 을 켜고 끌 수 있다. 만약 나오지 않는다면 `Claude` 를 껐다가 켜면 된다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-5.png)

먼저 `filesystem` 을 꺼놓고 질문을 했다. 그랬더니 `roadmap.md` 파일을 찾을 수 없다고 응답이 나온다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-6.png)

그 다음에 `filesystem` 을 켜고 그 내용을 읽어달라고 하더니 디렉토리를 사용하겠다는 내용이 나온다. 이것이 총 3번 나오는데, 모두 허용으로 눌러주면 된다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-7.png)

그러면 아래와 같이 내용을 읽어준다. 우리가 작성한 내용과 동일하다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-8.png)

그러면 기존 내용에 기반해서 6월 로드맵을 해당 파일에 저장해달라고 요청을 했다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-9.png)

그랬더니 아래와 같이 내용이 실제로 수정이 됐다.

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-10.png)

## Hashcat MCP 서버 구현

### add mcp

아래와 같은 작업을 진행한다. 

```
mkdir hashcat-mcp-server
cd hashcat-mcp-server
uv init --python 3.13
uv add mcp
```

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-3.png)

### main.py 수정

그리고 `main.py` 를 아래와 같이 수정한다. 

```python
import asyncio
import json
import subprocess
import tempfile
import os
from pathlib import Path
from typing import Any
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

# hashcat 해시 모드
HASH_MODES = {
    "md5": 0,
    "sha1": 100,
    "sha256": 1400,
}

server = Server("find-hash")

def detect_hash_type(hash_value: str) -> str | None:
    """해시 길이로 타입 추정"""
    hash_len = len(hash_value)
    if hash_len == 32:
        return "md5"
    elif hash_len == 40:
        return "sha1"
    elif hash_len == 64:
        return "sha256"
    return None

def run_hashcat(
    hash_value: str,
    hash_type: str,
    timeout: int = 300,
) -> dict[str, Any]:
    if hash_type not in HASH_MODES:
        return {"success": False, "error": f"Unsupported hash type: {hash_type}"}

    mode = HASH_MODES[hash_type]

    # 임시 입력 파일 생성
    with tempfile.NamedTemporaryFile(mode="w", suffix=".hash", delete=False) as f:
        f.write(hash_value)
        hash_file = f.name

    # 결과 파일 이름만
    output_file = tempfile.mktemp(suffix=".out")

    try:
        cmd = [
            "hashcat",
            "-m",
            str(mode),
            "-a",
            "3",
            hash_file,
            "-o",
            output_file,
            "--quiet",  
            "--show",  # potfile 있는 것도 출력
            "?a?a?a?a",  # 4자리 문자
        ]

        _result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            timeout=timeout,
        )

        # 결과 확인
        if Path(output_file).exists():
            with open(output_file, "r") as f:
                content = f.read().strip()
                if content:
                    # 형식: hash:plaintext
                    parts = content.split(":", maxsplit=1)
                    if len(parts) >= 2:
                        return {
                            "success": True,
                            "hash": parts[0],
                            "plaintext": parts[1],
                            "hash_type": hash_type,
                        }

        # 못 찾음
        return {
            "success": False,
            "error": "sorry but cannot find.",
            "hash": hash_value,
            "hash_type": hash_type,
        }

    except subprocess.TimeoutExpired:
        return {"success": False, "error": f"timeout: ({timeout}s)"}
    except FileNotFoundError:
        return {"success": False, "error": "no hashcat avail."}
    except Exception as e:
        return {"success": False, "error": str(e)}
    finally:
        for f in [hash_file, output_file]:
            try:
                os.unlink(f)
            except:
                pass  

@server.list_tools()

async def list_tools() -> list[Tool]:
    """사용 가능한 도구 목록"""
    return [
        Tool(
            name="crack_hash",
            description="Find original text of MD5, SHA1, SHA256 hash.",
            inputSchema={
                "type": "object",
                "properties": {
                    "hash_value": {
                        "type": "string",
                        "description": "hash value",
                    },
                    "hash_type": {
                        "type": "string",
                        "enum": ["md5", "sha1", "sha256", "auto"],
                        "description": "hash type",
                        "default": "auto",
                    },
                    "timeout": {
                        "type": "integer",
                        "description": "maximum time to find hash (s)",
                        "default": 300,
                    },
                },
                "required": ["hash_value"],
            },
        ),
        Tool(
            name="identify_hash",
            description="Find type of hash value.",
            inputSchema={
                "type": "object",
                "properties": {
                    "hash_value": {
                        "type": "string",
                        "description": "hash value",
                    },
                },
                "required": ["hash_value"],
            },
        ),
    ]
    
@server.call_tool()

async def call_tool(name: str, arguments: dict[str, Any]) -> list[TextContent]:

    """도구 실행"""
    if name == "identify_hash":
        hash_value = arguments["hash_value"].strip()
        detected = detect_hash_type(hash_value)
        if detected:
            result = {
                "hash": hash_value,
                "detected_type": detected,
                "length": len(hash_value),
            }
        else:
            result = {
                "hash": hash_value,
                "detected_type": None,
                "length": len(hash_value),
                "error": "Unknown hash type",
            }

        return [
            TextContent(
                type="text", text=json.dumps(result, ensure_ascii=False, indent=2)
            )
        ]

      elif name == "crack_hash":
        hash_value = arguments["hash_value"].strip()
        hash_type = arguments.get("hash_type", "auto")
        timeout = arguments.get("timeout", 300)

        if hash_type == "auto":
            hash_type = detect_hash_type(hash_value)
            if not hash_type:
                return [
                    TextContent(
                        type="text",
                        text=json.dumps(
                            {"success": False, "error": "cannot detect hash type."},
                            ensure_ascii=False,
                        ),
                    )
                ]

        result = await asyncio.to_thread(run_hashcat, hash_value, hash_type, timeout)

        return [
            TextContent(
                type="text", text=json.dumps(result, ensure_ascii=False, indent=2)
            )
        ]
        
    else:
        return [TextContent(type="text", text=f"Unknown tool: {name}")]

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream, write_stream, server.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

### claude_desktop_config.json 수정 (2)

앞서 `claude_desktop_config.json` 를 수정했는데, 이번에 `hashcat` 을 위한 내용도 넣어야 한다. 

```json
"find-hash": {
    "command": "uv",
    "args": [
        "run",
        "--project",
        "/Users/mixnuts/src/x/hashcat-mcp-server",
        "/Users/mixnuts/src/x/hashcat-mcp-server/main.py"
    ]
}
```

아래와 같은 위치에 넣는다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-11.png)

### hashcat MCP 추가

다시 들어가서 확인해 보면 `find-hash` 라는 게 정상적으로 `running` 인 거를 확인할 수 있다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-12.png)

그러면 임의의 해쉬를 던졌을 때 `hashcat` 을 통해서 해쉬를 복호화 해볼 수 있다. 

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-13.png)

![](/assets/images/2026-04-04-MCP%20&%20Agents%20Skills-14.png)

## Skills

### Skills 는 왜 존재하는가?

`MCP` 를 사용하려면 일반 사용자들이 만들기에 어려운 점도 있고, 프로그램 하나 당 서버 1개를 생성해야 하기 때문에 가성비에 맞지 않는다. 

요즘에는 `Agent` 가 커맨드라인과 `Python` 을 사용할 수 있기 때문에 가이드만 적절히 해주는 문서를 만들면 되는데, 이 방식이 `Agent Skills` 이다. 이와 관련한 컨텐츠들은 아래의 링크에 있다. 

[https://skillsmp.com/](https://skillsmp.com/)

### Claude 로 Skills 사용하기

위의 링크 중에서 우리는 다시 `hashcat` 을 통해 실습을 진행한다. 검색창에 `hashcat` 을 검색하면 여러 개가 나오지만 아래의 링크를 주도적으로 사용해볼 것이다. 

[https://skillsmp.com/skills/aeondave-malskill-offensive-tools-cracking-hashcat-skill-md](https://skillsmp.com/skills/aeondave-malskill-offensive-tools-cracking-hashcat-skill-md)

![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills.png)

위의 링크 중에서 `description` 과 `Quick Start` 정도만 따와서 조금 수정해서 사용할 것이다. 

![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-1.png)

![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-2.png)


![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-6.png)



![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-5.png)

### Codex - ChatGPT 로 Skills 사용하기

이번에는 `ChatGPT` 를 통해서 `Skills` 를 이용한다. 이번에는 `Codex` 를 이용하는 것이다. 

```bash
park@bba-2-50-184-66 hashcat % pwd
/Users/park/aistudy/test-codex/.agents/skills/hashcat
park@bba-2-50-184-66 hashcat % ls -al
total 8
drwxr-xr-x@ 3 park  staff   96  4  4 11:45 .
drwxr-xr-x@ 3 park  staff   96  4  4 11:45 ..
-rw-r--r--@ 1 park  staff  313  4  4 11:46 SKILL.md
park@bba-2-50-184-66 hashcat %
```

`test-codex` 디렉토리에 `.agents/skills/hashcat` 디렉토리를 만들고 `SKILL.md` 파일을 위에서 적은 것처럼 지침을 똑같이 작성해서 저장하면 된다. 

![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-8.png)

그러면 아래와 같이 해당 경로를 신뢰하냐고 묻는 질문에 그렇다고 대답해주면 된다. 

![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-9.png)

이후 `test-codex` 에 들어와서 `codex` 라고 타이핑해서 들어오면 된다. 그 다음 `$hashcat` 이라고 쳤을 때 스킬에 대한 해설이 아래와 같이 나오면 정상적으로 스킬이 적용되고 있는 거를 확인할 수 있다. 

![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-7.png)

그렇다면 정말 제대로 작동하는지를 확인해보기 위해 임의의 해쉬를 던져본다. 

```bash
park@bba-2-50-184-66 ~ % echo -n 'abcd' | md5sum
e2fc714c4727ee9395f324cd2e7f331f  -
```

![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-10.png)

```bash
park@bba-2-50-184-66 ~ % echo -n 'ycpp' | md5sum
9fdcf79c14894159948c25ba8b4f1ddc  -
```

![](/assets/images/2026-04-04-MCP%20&%20Agent%20Skills-11.png)

스킬 지침에 4자리 해쉬라고 적어놨고, 4자리 해쉬를 던졌을 때 정상적으로 해독이 되는 것을 보아 스킬이 제대로 적용되었음을 확인할 수 있다. 

---

Thanks to `0xrgb`