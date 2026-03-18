---
description: Knowledge Base를 연결한 Strands 에이전트 코드를 작성합니다
---

# RAG 에이전트 구현

## 프로젝트 준비

새 폴더를 만들고 필요한 도구를 설치합니다.

```bash
mkdir gs25-rag-agent
cd gs25-rag-agent
```

### Python 가상환경 만들기

```bash
python3 -m venv .venv
source .venv/bin/activate   # Mac/Linux
# .venv\Scripts\activate    # Windows
```

### 패키지 설치

```bash
pip install strands-agents strands-agents-bedrock fastapi uvicorn boto3
```

<details>
<summary>각 패키지가 하는 일</summary>

| 패키지 | 역할 |
|--------|------|
| `strands-agents` | AI 에이전트를 만드는 핵심 도구 |
| `strands-agents-bedrock` | AWS Bedrock의 Claude 모델 연결 |
| `fastapi` | 웹 API 서버 (채팅 UI와 통신) |
| `uvicorn` | FastAPI 서버를 실행하는 도구 |
| `boto3` | AWS 서비스를 Python에서 호출 |

</details>

---

## 에이전트 코드 작성

### agent.py

아래 코드를 `agent.py`로 저장합니다.

{% hint style="warning" %}
**`YOUR_KB_ID_HERE`를 Module 4에서 메모한 Knowledge Base ID로 교체하세요!**
{% endhint %}

{% code title="agent.py" %}
```python
import boto3
from strands import Agent, tool
from strands.models.bedrock import BedrockModel

# ============================================
# 설정 - 본인의 Knowledge Base ID로 교체하세요
# ============================================
KNOWLEDGE_BASE_ID = "YOUR_KB_ID_HERE"
AWS_REGION = "us-east-1"

# Bedrock Agent Runtime 클라이언트
bedrock_agent_runtime = boto3.client(
    "bedrock-agent-runtime",
    region_name=AWS_REGION
)


@tool
def search_gs25_knowledge(query: str) -> str:
    """GS25 매장 운영, 상품, 발주 정책에 대한 정보를 Knowledge Base에서 검색합니다.

    Args:
        query: 검색할 질문 또는 키워드
    """
    response = bedrock_agent_runtime.retrieve(
        knowledgeBaseId=KNOWLEDGE_BASE_ID,
        retrievalQuery={"text": query},
        retrievalConfiguration={
            "vectorSearchConfiguration": {
                "numberOfResults": 5
            }
        }
    )

    results = []
    for item in response.get("retrievalResults", []):
        text = item["content"]["text"]
        score = item.get("score", 0)
        source = item.get("location", {}).get("s3Location", {}).get("uri", "unknown")
        results.append(f"[출처: {source}] (관련도: {score:.2f})\n{text}")

    if not results:
        return "관련 정보를 찾지 못했습니다."

    return "\n\n---\n\n".join(results)


def create_agent() -> Agent:
    """RAG 에이전트를 생성합니다."""
    model = BedrockModel(
        model_id="anthropic.claude-3-5-sonnet-20241022-v2:0",
        region_name=AWS_REGION,
    )

    system_prompt = """당신은 GS25 편의점 매장 운영을 도와주는 AI 어시스턴트입니다.

## 역할
- GS25 매장 운영, 상품, 발주 정책에 대한 질문에 답변합니다.
- 항상 search_gs25_knowledge 도구를 사용하여 Knowledge Base에서 정보를 검색한 후 답변합니다.

## 규칙
1. 반드시 Knowledge Base 검색 결과를 근거로 답변하세요.
2. 검색 결과에 없는 내용은 "해당 정보는 현재 Knowledge Base에 없습니다"라고 안내하세요.
3. 답변 시 출처를 함께 언급하세요.
4. 한국어로 친절하게 답변하세요.
5. 가능하면 구체적인 수치와 절차를 포함하세요."""

    return Agent(
        model=model,
        tools=[search_gs25_knowledge],
        system_prompt=system_prompt,
    )


if __name__ == "__main__":
    agent = create_agent()
    print("GS25 RAG 에이전트가 시작되었습니다. 'quit'를 입력하면 종료합니다.\n")

    while True:
        user_input = input("질문: ")
        if user_input.lower() in ("quit", "exit", "종료"):
            break
        response = agent(user_input)
        print(f"\n답변: {response}\n")
```
{% endcode %}

---

## API 서버 코드 작성

### server.py

채팅 UI에서 에이전트를 호출할 수 있도록 웹 서버를 만듭니다.

{% code title="server.py" %}
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import FileResponse
from pydantic import BaseModel
from agent import create_agent

app = FastAPI(title="GS25 RAG Agent API")

# CORS 설정 (로컬 개발용)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# 에이전트 인스턴스 생성
agent = create_agent()


class ChatRequest(BaseModel):
    message: str


class ChatResponse(BaseModel):
    reply: str


@app.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    """에이전트에게 질문을 보내고 답변을 받습니다."""
    response = agent(request.message)
    return ChatResponse(reply=str(response))


@app.get("/")
async def root():
    """채팅 UI를 제공합니다."""
    return FileResponse("index.html")


@app.get("/health")
async def health():
    return {"status": "ok"}
```
{% endcode %}

---

## 터미널에서 먼저 테스트

API 서버 없이 에이전트만 먼저 테스트해봅니다:

```bash
python agent.py
```

이런 질문들을 입력해보세요:

```
질문: 도시락 발주량은 어떻게 계산하나요?
질문: 오전 근무자가 해야 할 일은?
질문: 비가 올 때 어떤 상품을 더 발주해야 해?
```

{% hint style="success" %}
에이전트가 Knowledge Base에서 검색한 정보를 바탕으로 답변하는 것을 확인할 수 있습니다!
{% endhint %}
