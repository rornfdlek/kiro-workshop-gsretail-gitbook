---
description: Kiro에게 프롬프트를 보내서 Strands RAG 에이전트를 자동 생성합니다
---

# Kiro로 RAG 에이전트 만들기

## 새 프로젝트 시작

Part 1에서 했던 것처럼, Kiro에서 새 프로젝트를 시작합니다.

1. Kiro를 실행합니다
2. **File → Open Folder**를 클릭합니다
3. 새 폴더를 만들고 선택합니다 (예: `gs25-rag-agent`)
4. 왼쪽에 채팅 패널이 보이면 준비 완료!

---

## Step 1: 프로젝트 소개 프롬프트

Kiro에게 우리가 만들 에이전트를 설명합니다:

{% code title="📋 복사해서 붙여넣기" %}
```
GS25 AI 매장 도우미 에이전트를 만들겠습니다.

Amazon Bedrock Knowledge Base에 저장된 GS25 매장 운영 문서(상품 카탈로그, 발주 정책, 매장 운영 매뉴얼)를 검색하여
점주의 질문에 근거 기반으로 답변하는 RAG 에이전트입니다.

Strands Agents SDK를 사용하여 단일 에이전트를 구현하고,
FastAPI 서버로 API를 제공하며,
예쁜 HTML 채팅 UI에서 점주가 대화할 수 있게 합니다.

지금부터 세부 내용을 설명하겠습니다. 한글로 작업해주세요.
```
{% endcode %}

{% hint style="info" %}
Part 1에서 GS25 발주 시스템을 설명했던 것과 같은 패턴입니다! 이번에는 **AI 에이전트**를 설명하는 것이에요.
{% endhint %}

---

## Step 2: 기술스택 알려주기

{% code title="📋 복사해서 붙여넣기" %}
```
기술스택은 다음과 같습니다.
- Language: Python 3.10+
- Agent Framework: Strands Agents SDK (strands-agents, strands-agents-bedrock)
- LLM: Amazon Bedrock Claude 3.5 Sonnet (anthropic.claude-3-5-sonnet-20241022-v2:0)
- RAG: Amazon Bedrock Knowledge Base (Retrieve API 사용)
- API Server: FastAPI + Uvicorn
- Frontend: 단일 HTML 파일 (채팅 UI)
- AWS SDK: boto3
```
{% endcode %}

<details>
<summary>💡 위 기술스택이 뭔지 궁금하다면?</summary>

| 용어 | 쉬운 설명 |
|------|-----------|
| **Strands Agents SDK** | AI에게 도구를 쓰는 능력을 부여하는 프레임워크 |
| **Bedrock Claude** | AI의 두뇌 역할 (답변 생성) |
| **Knowledge Base Retrieve API** | 문서를 검색하는 API |
| **FastAPI** | 채팅 UI와 에이전트를 연결하는 웹 서버 |

</details>

---

## Step 3: 에이전트 동작 방식 알려주기

{% code title="📋 복사해서 붙여넣기" %}
```
에이전트의 동작 방식입니다.

1. 에이전트 구성:
   - @tool 데코레이터로 Knowledge Base 검색 도구를 만듭니다
   - boto3의 bedrock-agent-runtime 클라이언트로 Retrieve API를 호출합니다
   - Knowledge Base ID는 환경변수 또는 상수로 설정합니다
   - BedrockModel을 사용하여 Claude 모델을 연결합니다

2. 검색 도구 (search_gs25_knowledge):
   - 입력: 검색 질문 (str)
   - bedrock-agent-runtime의 retrieve API 호출
   - numberOfResults: 5
   - 결과에서 content text, score, s3 source URI를 추출
   - 포맷팅하여 반환

3. 시스템 프롬프트:
   - 역할: GS25 편의점 매장 운영 도우미
   - 항상 search_gs25_knowledge 도구로 검색 후 답변
   - 검색 결과에 없으면 "정보가 없습니다"라고 안내
   - 한국어로 친절하게, 출처와 함께 답변

4. Agent 생성:
   - model: BedrockModel (Claude 3.5 Sonnet)
   - tools: [search_gs25_knowledge]
   - system_prompt: 위 시스템 프롬프트
```
{% endcode %}

---

## Step 4: API 서버 설명

{% code title="📋 복사해서 붙여넣기" %}
```
FastAPI 서버 구성입니다.

- POST /chat: ChatRequest(message: str) → ChatResponse(reply: str)
  - 에이전트에게 message를 전달하고 응답을 reply로 반환
- GET /: index.html 파일을 FileResponse로 서빙
- GET /health: 상태 확인 엔드포인트

CORS는 모든 origin을 허용합니다 (로컬 개발용).
에이전트 인스턴스는 서버 시작 시 한 번 생성합니다.
```
{% endcode %}

---

## Step 5: 채팅 UI 설명

{% code title="📋 복사해서 붙여넣기" %}
```
채팅 UI는 단일 HTML 파일(index.html)로 만듭니다.

디자인:
- GS25 브랜드 컬러(파란색 #0066cc) 사용
- 480x700 크기의 채팅 카드 (가운데 정렬)
- 보라색 그라데이션 배경
- 상단에 GS25 로고와 "AI 매장 도우미" 타이틀
- 채팅 메시지 영역 (봇: 흰색 말풍선, 사용자: 파란색 말풍선)
- 타이핑 인디케이터 (점 3개 깜빡임 애니메이션)
- 하단에 입력창과 전송 버튼
- 추천 질문 버튼 4개: "도시락 발주량 계산법", "오전 근무 체크리스트", "비 올 때 발주 정책", "인기 상품 TOP5"

기능:
- Enter 키 또는 전송 버튼으로 메시지 전송
- http://localhost:8000/chat 으로 POST 요청
- 응답 대기 중 타이핑 인디케이터 표시
- 추천 질문 클릭 시 자동 전송
- 서버 연결 실패 시 에러 메시지 표시
```
{% endcode %}

---

## Step 6: 요구사항 간소화

{% code title="📋 복사해서 붙여넣기" %}
```
요구사항을 간소화합니다.
파일 3개만 만들면 됩니다:
1. agent.py - Strands 에이전트 + Knowledge Base 검색 도구
2. server.py - FastAPI 서버
3. index.html - 채팅 UI

복잡한 구조 없이 최소한의 파일로 빠르게 구축해주세요.
```
{% endcode %}

---

## Step 7: Spec 문서 생성

Part 1에서 했던 것처럼, Kiro에게 Spec 문서를 만들어달라고 요청합니다:

{% code title="📋 복사해서 붙여넣기" %}
```
requirements를 작성해주세요.
```
{% endcode %}

요구사항 문서가 생성되면 확인하고:

{% code title="📋 복사해서 붙여넣기" %}
```
이대로 디자인 문서를 만들어주세요.
```
{% endcode %}

설계 문서가 생성되면:

{% code title="📋 복사해서 붙여넣기" %}
```
Task를 정의해주세요.
```
{% endcode %}

{% hint style="info" %}
Part 1과 똑같은 흐름입니다! **프롬프트 → 요구사항 → 설계 → 태스크** 순서로 Kiro가 자동으로 문서를 만들어줍니다.
{% endhint %}

---

## Step 8: 태스크 실행

`tasks.md`에 있는 체크박스를 **순서대로 클릭**하여 코드를 생성합니다.

{% hint style="warning" %}
⚠️ Part 1에서 했던 것처럼, **위에서 아래로 순서대로** 실행하세요!
{% endhint %}

태스크 실행이 모두 끝나면, 프로젝트에 이런 파일들이 만들어져 있을 것입니다:

```
gs25-rag-agent/
├── agent.py        # AI 에이전트 코드
├── server.py       # 웹 API 서버
├── index.html      # 채팅 화면
└── requirements.txt 또는 pyproject.toml
```

---

## Step 9: Knowledge Base ID 설정

Kiro가 생성한 `agent.py` 파일을 열어서, Knowledge Base ID를 Module 4에서 메모한 값으로 교체합니다.

{% hint style="warning" %}
**`YOUR_KB_ID_HERE`** 또는 비슷한 플레이스홀더를 찾아서 **본인의 Knowledge Base ID**로 바꿔주세요!
{% endhint %}

{% hint style="success" %}
🎉 코드가 모두 생성되었습니다! 다음 단계에서 실행해봅시다!
{% endhint %}
