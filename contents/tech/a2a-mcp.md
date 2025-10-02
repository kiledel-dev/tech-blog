---
date: "2025-04-29 17:10"
title: 'LLM 에이전트 통신 프로토콜 분석: MCP와 A2A'
tags: ['markdown', 'github']
summary: 'LLM 모델이 외부 도구와 상호작용하는 방식인 MCP와, Agent 간 통신 방식인 A2A 프로토콜에 대해 탐구'
thumbnail: '../assets/a2a-mcp.png'
update: true
---
# AI 에이전트 통신 프로토콜 분석: MCP와 A2A

> 본 문서는 최근 에이전트 기술의 핵심적인 두 가지 개념, \*\*MCP(Model Context Protocol)\*\*와 **A2A(Agent-to-Agent) 프로토콜**을 각각 분석하는 것을 목표로 합니다. 특히, LLM 모델이 외부 도구 및 데이터와 상호작용하는 표준화된 방식인 MCP를 이해하고, 최근 화제가 되고 있는 A2A 프로토콜이 어떻게 구성되어 있는지 탐구하고자 합니다.

-----

## 1\. MCP (Model Context Protocol)

### 1-1. MCP의 의미: LLM이 외부 소통을 위한 통신 규약

**MCP**는 LLM 모델(예: GPT)이 외부의 다양한 데이터나 도구(예: 내 컴퓨터의 파일, 데이터베이스, 웹 API)와 표준화된 방식으로 안전하게 소통하기 위한 \*\*'통신 규약(프로토콜)'\*\*입니다.

흔히 **USB 포트**에 비유됩니다. 전 세계의 모든 전자제품이 USB 포트라는 표준 규격으로 쉽게 연결되는 것처럼, MCP는 어떤 LLM 모델이든, 어떤 코드 에디터든 "MCP"라는 표준화된 포트를 통해 쉽게 연결할 수 있도록 만들어 줍니다.

### 1-2. MCP 이전의 LLM 한계점

#### ✔️ 함수 호출 (pre-Settings)

  - 기존에는 LLM 프롬프트 엔지니어링을 통해 구조화된 출력(Structured Output)을 유도했습니다.
  - **한계**: 텍스트 파싱(Text Parsing) 방식은 원하는 형식으로 정확히 출력되지 않는 경우가 많아 LLM 기반 애플리케이션의 불안정성을 높였습니다.

#### ✔️ 도구 호출 (Function Calling)

2023년 7월, OpenAI가 **Function Calling** 기능을 공개하며 사용자의 자연어 요청을 이해하고 외부 시스템의 특정 기능(함수)을 호출할 수 있게 되었습니다.

A. **함수 정의 및 메시지 전달**: 모델에게 호출 가능한 함수(`Name`, `Description`, `Args`)와 사용자 요청을 함께 전달합니다.
B. **함수 호출**: 모델은 호출할 함수 이름과 입력 인자(`args`)를 결정하여 반환합니다.
C. **외부 함수 실행**: 모델로부터 받은 정보를 바탕으로 실제 함수를 실행합니다.
D. **실행 결과 반환**: 실행 결과를 다시 모델에게 전달합니다 (Add Contextual Message).
E. **최종 응답 생성**: 모델은 함수 실행 결과를 포함하여 사용자에게 최종 응답을 생성합니다.

#### ✔️ Function Calling 방식의 한계점: M \* N 통합 문제

  - **상태 유지 어려움**: 단일 요청-응답 형태로 작동하여 각 호출이 독립적이었습니다.
  - **복잡한 작업 처리**: 여러 단계의 워크플로우나 다양한 데이터 동시 활용에 부적합했습니다.
  - **플랫폼 단편화**: LLM 모델이나 API마다 다른 함수 호출 방식을 사용해 반복적인 구현이 필요했습니다.

이는 각 모델과 도구를 연결할 때마다 개별적인 '맞춤형 어댑터'를 만들어야 하는 번거로움, 즉 \*\*'M x N 통합 문제'\*\*로 이어졌습니다.

### 1-3. MCP의 배경 및 구성

#### ✔️ MCP 등장 배경

  - **Function Calling의 비효율성**: 플랫폼, 도구 별로 제각기 다른 접근 로직이 필요했습니다.
  - **Context 기반 답변 필요**: 유용한 답변을 제공하기 위해 적절한 맥락(Context)에 기반한 LLM의 행동이 필요했습니다.
  - **환각(Hallucination) 발생**: 오류가 난 데이터조차 정상처럼 전달되는 현상을 방지하고, 사용자 개인화된 맥락을 반영할 필요가 있었습니다.

#### ✔️ Function Calling vs MCP

  - **Function Calling**: LLM이 정해진 규격으로 외부와 상호작용을 시작합니다. 특정 작업을 수행하기 위한 \*\*"어떻게(How)"\*\*에 가깝습니다.
  - **Model Context Protocol**: 다양한 도구와 맥락을 일관된 방식으로 활용하는 규약입니다. \*\*"무엇을(What)", "왜(Why)", "언제(When)"\*\*까지 고려합니다.

> 2024년 11월, Claude 모델을 개발한 Anthropic에서 자사 도구로 활용하던 기술을 오픈 소스로 공개했습니다. MCP는 AI 모델(Model)이 외부 자원 간 맥락(Context) 정보를 일관된 형태로 주고받을 수 있도록 정의한 표준 규약입니다.

#### ✔️ MCP 핵심 아키텍처: 호스트-클라이언트-서버 구조

MCP는 \*\*AI 애플리케이션(호스트)\*\*가 여러 **MCP 클라이언트**를 관리하고, 각 클라이언트는 개별 **MCP 서버**들과 1:1로 연결하여 필요한 맥락과 도구를 동적으로 조율하는 모듈식 구조를 가집니다. 이를 통해 AI 에이전트와 도구 간의 **수직적인 연결을 표준화**했습니다.

  - **MCP Host**: LLM 통합의 오케스트레이터. LLM 애플리케이션(예: Claude Desktop, Cursor AI IDE)이 해당됩니다. 여러 서버 정보를 종합하여 LLM에 전달합니다.
  - **MCP Client**: MCP 프로토콜 어댑터. 호스트 내에서 실행되며 개별 MCP 서버와 1:1로 통신 채널을 격리하여 안정성을 확보합니다.
  - **MCP Server**: 특화된 맥락 및 도구 제공자. 파일 시스템, DB, API 등 특정 데이터 소스나 도구에 대한 접근을 제공하며 독립적으로 운영됩니다.

#### ✔️ MCP 표준 전송 메커니즘 / 통신 방식

  - **메시지 형태**: 요청(Request), 응답(Response), 알림(Notification)
  - **통신 프로토콜**: **JSON-RPC 2.0** 기반
  - **인증 (Authorization)**: **OAuth 2.1** 표준 활용
  - **전송 방식**:
      - **STDIO (Standard I/O)**: 주로 로컬에서 사용. 간단하고 빠른 연결에 적합합니다.
      - **Streamable HTTP**: 주로 원격/외부에서 사용. HTTP POST와 Server-Sent Events(SSE)를 결합하여 양방향 스트리밍을 지원합니다.
  - **MCP 도구 바인딩 플랫폼**: [smithery.ai](https://smithery.ai/)

### 1-4. MCP 구현 예제

  - **도움 사이트**:
      - [wikidocs.net/book/17905](https://wikidocs.net/book/17905)
      - [github.com/kwonskdev/nl\_map\_search\_mcp](https://github.com/kwonskdev/nl_map_search_mcp)

#### MCP Server 구현 (Python FastMCP)

```python
# 출처: https://gofastmcp.com/deployment/running-server
from typing import Any
import httpx
from mcp.server.fastmcp import FastMCP

# "name-origin"이라는 이름으로 MCP 서버 객체를 생성합니다.
mcp = FastMCP("name-origin")

# API 요청을 보낼 기본 URL을 정의합니다.
base_url = "https://api.nationalize.io"

@mcp.tool()
async def predict_origin(self, name: str) -> dict:
    """Nationalize API를 사용하여 주어진 이름의 출신 국가를 예측합니다."""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"{base_url}/?name={name}")
        return response.json()

@mcp.tool()
async def batch_predict(self, names: list[str]) -> dict:
    """여러 이름의 출신 국가를 한 번에 예측합니다."""
    results = {}
    for name in names:
        results[name] = await self.predict_origin(name)
    return results

if __name__ == "__main__":
    print("이름 출신 국가 서버 시작 중...")
    mcp.run(transport='stdio')
```

#### MCP Client 구현

```python
from mcp import ClientSession, StdioServerParameters, types
from mcp.client.stdio import stdio_client
import asyncio

# STDIO 방식으로 서버에 연결하기 위한 파라미터를 설정합니다.
server_params = StdioServerParameters(
    command="python",
    args=["example_server.py"],
)

async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            
            # 서버가 제공하는 도구 목록을 가져옵니다.
            tools = await session.list_tools()
            print(f"Available tools: {tools}")

            # 'predict_origin' 도구를 호출하고 결과를 받습니다.
            result = await session.call_tool("predict_origin", arguments={"name": "michael"})
            print(f"Tool call result: {result}")

if __name__ == "__main__":
    asyncio.run(run())
```

-----

## 2\. A2A (Agent-to-Agent)

### 2-1. A2A 이전의 멀티 에이전트 한계점

#### ✔️ A2A 등장 배경

  - **MCP 이후**: MCP는 'Agent-Tool' 간의 **수직적** 통합 문제를 해결했지만, 에이전트 간의 **수평적** 통합 표준이 필요해졌습니다.
  - **프레임워크 비호환성**: LangGraph, AutoGen, CrewAI 등 다양한 프레임워크 간의 상호 운용성 문제가 발생했습니다. 에이전트들이 서로 다른 프로젝트에 흩어져 있으면 협업이 어려워지고, 하나의 프로젝트에서 모든 것을 관리하면 의존성이 과도해지는 문제가 있었습니다 (Monolithic → Microservices 전환 필요).

#### ✔️ A2A 프로토콜

> **A2A(Agent-to-Agent) 프로토콜**은 AI 에이전트 간 구조화된 통신을 가능하게 하는 개방형 프로토콜입니다.

  - **첫 등장**: 2025년 4월, Google Cloud Next 2025 컨퍼런스에서 발표되었습니다.
  - **오픈 소스화**: 2025년 6월, 프로젝트를 리눅스 재단에 기부하여 벤더 종속 없이 산업 표준으로 성장할 기반을 마련했습니다. (구글 외 AWS, Microsoft, SAP 등 참여)
  - **GitHub**: [github.com/a2aproject/A2A](https://github.com/a2aproject/A2A)

#### ✔️ A2A 핵심 가치 및 설계 원칙

  - **핵심 가치**: 서로 다른 기술/플랫폼으로 만들어진 **이기종(Heterogeneous) 에이전트 간의 소통**을 지원합니다.
  - **설계 원칙**:
    1.  **에이전트 능력 수용**: 메모리, 도구를 공유하지 않아도 협업 가능.
    2.  **기존 표준 기반 개발**: HTTP, SSE, JSON-RPC 등 널리 쓰이는 기술 기반.
    3.  **기본적인 보안 보장**: 엔터프라이즈급 인증 및 승인 지원.
    4.  **장기 실행 작업 지원**: 실시간 피드백, 알림, 상태 업데이트 제공.
    5.  **다양한 모달리티 지원**: 텍스트뿐만 아니라 오디오, 비디오 스트리밍 지원.

> **A2A의 핵심**: **기능의 서비스화**. 각 에이전트가 자신의 핵심 기능을 '서비스'로 외부에 제공합니다.

#### ✔️ A2A의 핵심 비유: AI 에이전트들의 SNS

MCP가 에이전트와 도구를 연결한다면, A2A는 에이전트와 에이전트를 연결하여 네트워크를 형성합니다.

  - 프로필(**Agent Card**)을 공개하고
  - 친구를 맺으며(**Agent Discovery**)
  - 메시지를 보내 요청하고(**Task / Message**)
  - 상호작용하며 협력합니다(**Artifact**).

### 2-2. A2A 주요 구성 요소

  - **Core Actors**: `User`, `A2A Client` (Client Agent), `A2A Server` (Remote Agent)
  - **Fundamental Elements**: `Agent Card`, `Task`, `Message`, `Part`, `Artifact`
  - **Interaction Mechanism**: `Request/Response` (Polling), `Streaming` (SSE), `Push Notifications` (Webhook)

#### ✔️ Agent Card: 에이전트의 '명함'

A2A 서버를 설명하는 JSON 메타데이터로, 일반적으로 `/.well-known/agent.json` 경로를 통해 공개적으로 접근할 수 있습니다.

| 필드명 | 타입 | 필수 | 설명 |
| :--- | :--- | :--- | :--- |
| `protocolVersion` | string | Yes | A2A 프로토콜의 버전입니다. |
| `name` | string | Yes | 사람이 읽을 수 있는 에이전트의 이름입니다. |
| `description` | string | Yes | 사람이 읽을 수 있는 설명입니다. |
| `url` | string | Yes | 에이전트의 A2A 서비스를 위한 기본 URL입니다. |
| `version` | string | Yes | 에이전트 또는 A2A 구현 버전의 문자열입니다. |
| `capabilities` | AgentCapabilities | Yes | 지원하는 A2A 프로토콜 기능을 지정합니다. |
| `defaultInputModes` | string[] | Yes | 에이전트가 허용하는 입력 미디어 유형입니다. |
| `defaultOutputModes` | string[] | Yes | 에이전트가 생성하는 출력 미디어 유형입니다. |
| `skills` | AgentSkill[] | Yes | 스킬 배열입니다. 최소 하나 이상의 스킬이 있어야 합니다. |

#### ✔️ Task, Message, Part, Artifact

  - **Task**: 상태를 가지는(Stateful) 공식적인 업무 단위. `submitted`, `working`, `completed` 등의 수명 주기를 가집니다.
  - **Message**: 클라이언트와 에이전트 간의 단일 대화 턴(turn).
  - **Part**: 메시지 내 콘텐츠의 기본 단위 (`TextPart`, `FilePart`, `DataPart` 등).
  - **Artifact**: Task 처리 중 생성된 유형의(Tangible) 결과물 (문서, 이미지, JSON 등).

#### ✔️ 3가지 상호작용 메커니즘

1.  **Request/Response (Polling)**: 클라이언트가 주기적으로 상태를 확인하는 단순한 방식.
2.  **Streaming (SSE)**: 서버가 클라이언트에게 실시간으로 진행 상황을 계속 알려주는 스트리밍 방식.
3.  **Push Notifications (Webhook)**: 작업이 매우 오래 걸릴 때, 서버가 작업 완료 후 클라이언트에게 비동기적으로 알려주는 방식.

### 2-3. A2A 프로토콜 구성요소 요약

  - **Actors (행위자)**: `User`(사용자), `A2A Client`(요청 주체), `A2A Server`(작업 수행 주체).
  - **Agent Cards (명함)**: 에이전트의 능력과 접속 방법을 담은 공개 프로필.
  - **Transports (통신 방식)**: `Request/Response`, `Streaming (SSE)`, `Push Notifications` 3가지 방식.

### 2-4. A2A 프로토콜 예제

  - **공식 사이트**: [a2aprotocol.ai](https://a2aprotocol.ai/#developers)
  - **샘플 코드**: [A2A Hello World 프로젝트](https://github.com/a2aproject/a2a-samples/tree/main/samples/python/agents/helloworld)

#### `agent_executor.py` (비즈니스 로직)

```python
from typing_extensions import override
from a2a.server.agent_execution import AgentExecutor, RequestContext
from a2a.server.events import EventQueue
from a2a.utils import new_agent_text_message

class HelloWorldAgent:
    """'Hello World' 기능을 수행하는 비즈니스 로직을 담은 클래스"""
    async def invoke(self) -> str:
        return 'Hello World'

class HelloWorldAgentExecutor(AgentExecutor):
    """A2A 프로토콜 규칙에 따라 HelloWorldAgent를 실행하고 통신하는 실행기"""
    def __init__(self):
        self.agent = HelloWorldAgent()

    @override
    async def execute(self, context: RequestContext, event_queue: EventQueue) -> None:
        """클라이언트로부터 작업 요청이 오면 이 메서드가 실행됨"""
        result = await self.agent.invoke()
        event_queue.enqueue_event(new_agent_text_message(result))
```

#### `server.py` (A2A 서버 설정)

```python
from agent_executor import HelloWorldAgentExecutor
from a2a.server.apps import A2AStarletteApplication
from a2a.server.request_handlers import DefaultRequestHandler
from a2a.server.tasks import InMemoryTaskStore
from a2a.types import AgentCapabilities, AgentCard, AgentSkill
import uvicorn

if __name__ == '__main__':
    skill = AgentSkill(id='hello_world', name='Returns hello world', ...)
    
    # 다른 에이전트에게 자신을 소개하는 '명함(Agent Card)' 정보 정의
    agent_card = AgentCard(
        name='Hello World Agent',
        description='Just a hello world agent',
        url='http://localhost:9999/',
        version='1.0.0',
        defaultInputModes=['text'],
        defaultOutputModes=['text'],
        capabilities=AgentCapabilities(streaming=True), # 스트리밍 기능 지원 명시
        skills=[skill],
    )
    
    request_handler = DefaultRequestHandler(
        agent_executor=HelloWorldAgentExecutor(),
        task_store=InMemoryTaskStore(),
    )
    
    server = A2AStarletteApplication(
        agent_card=agent_card, http_handler=request_handler
    )
    
    uvicorn.run(server.build(), host='0.0.0.0', port=9999)
```

#### `client.py` (A2A 클라이언트)

```python
from a2a.client import A2AClient
import httpx
import asyncio

async def main() -> None:
    async with httpx.AsyncClient() as httpx_client:
        # 에이전트 카드 URL을 통해 A2A 클라이언트 객체 획득
        client = await A2AClient.get_client_from_agent_card_url(
            httpx_client, 'http://localhost:9999'
        )
        
        # 스트리밍 방식으로 메시지 요청 및 응답 처리
        async for chunk in client.send_message_streaming(...):
            print(chunk.model_dump(mode='json', exclude_none=True))

if __name__ == '__main__':
    asyncio.run(main())
```

-----

### \* A2A와 MCP 요약

> **MCP**와 **A2A**는 모두 잠재력이 매우 높은 기술이지만, 아직 산업 표준으로 자리 잡는 초입 단계에 있다고 볼 수 있습니다. 그럼에도 불구하고 AI 에이전트 기술의 미래 방향성을 제시하는 만큼, 지속적으로 예의주시할 필요가 있습니다.