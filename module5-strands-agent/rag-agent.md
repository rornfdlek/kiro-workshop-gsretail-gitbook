---
description: Kiro에게 프롬프트 하나로 Strands RAG 에이전트를 바이브 코딩합니다
---

# Kiro로 RAG 에이전트 만들기

## 새 프로젝트 시작

Part 1에서 했던 것처럼, Kiro에서 새 프로젝트를 시작합니다.

1. Kiro를 실행합니다
2. **File → Open Folder**를 클릭합니다
3. 새 폴더를 만들고 선택합니다 (예: `gs25-rag-agent`)
4. 왼쪽에 채팅 패널이 보이면 준비 완료!

---

## Step 1: 프로젝트 스펙 파일 만들기

{% hint style="info" %}
Part 1에서는 **여러 번의 프롬프트**로 Kiro에게 설명한 뒤 Spec 문서를 생성했습니다. 이번에는 다른 방식으로 해봅니다 — **스펙을 미리 파일로 만들어두고**, 프롬프트 하나로 바로 코드를 생성하는 **바이브 코딩** 방식입니다!
{% endhint %}

프로젝트 폴더에 `PROJECT_SPEC.md` 파일을 만들고, 아래 내용을 복사해서 붙여넣으세요:

{% code title="PROJECT_SPEC.md — 전체를 복사해서 파일로 저장" %}
```markdown
# GS25 AI 매장 도우미 에이전트

## 프로젝트 개요
Amazon Bedrock Knowledge Base에 저장된 GS25 매장 운영 문서(상품 카탈로그, 발주 정책, 매장 운영 매뉴얼)를 검색하여
점주의 질문에 근거 기반으로 답변하는 RAG 에이전트입니다.

## 기술스택
- Language: Python 3.10+
- Agent Framework: Strands Agents SDK (strands-agents)
- LLM: Amazon Bedrock Claude Sonnet 4 (us.anthropic.claude-sonnet-4-20250514-v1:0)
- RAG: Amazon Bedrock Knowledge Base (Retrieve API 사용)
- API Server: FastAPI + Uvicorn
- Frontend: 단일 HTML 파일 (채팅 UI)
- AWS SDK: boto3

> 중요: 모델 ID는 반드시 cross-region inference profile ID를 사용합니다.
> `anthropic.claude-...` 형태의 기본 모델 ID가 아니라 `us.anthropic.claude-sonnet-4-20250514-v1:0`을 사용해야 합니다.

## 파일 구조
파일 3개만 만듭니다. 복잡한 구조 없이 최소한의 파일로 빠르게 구축합니다.
1. agent.py - Strands 에이전트 + Knowledge Base 검색 도구
2. server.py - FastAPI 서버
3. index.html - 채팅 UI

## 에이전트 동작 방식

### 1. 에이전트 구성
- @tool 데코레이터로 Knowledge Base 검색 도구를 만듭니다
- boto3의 bedrock-agent-runtime 클라이언트로 Retrieve API를 호출합니다
- Knowledge Base ID는 환경변수 또는 상수로 설정합니다
- BedrockModel을 사용하여 Claude 모델을 연결합니다
- 모델 ID는 cross-region inference profile ID를 사용합니다 (us.anthropic.claude-sonnet-4-20250514-v1:0)

### 2. 검색 도구 (search_gs25_knowledge)
- 입력: 검색 질문 (str)
- bedrock-agent-runtime의 retrieve API 호출
- numberOfResults: 5
- 결과에서 content text, score, s3 source URI를 추출
- 포맷팅하여 반환

### 3. 시스템 프롬프트
- 역할: GS25 편의점 매장 운영 도우미
- 항상 search_gs25_knowledge 도구로 검색 후 답변
- 검색 결과에 없으면 "정보가 없습니다"라고 안내
- 한국어로 친절하게, 출처와 함께 답변

### 4. Agent 생성
- model: BedrockModel (Claude Sonnet 4, cross-region inference profile ID)
- tools: [search_gs25_knowledge]
- system_prompt: 위 시스템 프롬프트

## FastAPI 서버

- POST /chat: ChatRequest(message: str) → ChatResponse(reply: str)
  - 에이전트에게 message를 전달하고 응답을 reply로 반환
- GET /: index.html 파일을 FileResponse로 서빙
- GET /health: 상태 확인 엔드포인트

CORS는 모든 origin을 허용합니다 (로컬 개발용).
에이전트 인스턴스는 서버 시작 시 한 번 생성합니다.

## 채팅 UI (index.html)

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

{% hint style="warning" %}
**Knowledge Base ID를 잊지 마세요!** 이 파일에는 KB ID 플레이스홀더가 없지만, Kiro가 코드를 생성한 후에 `agent.py`에서 본인의 KB ID로 교체해야 합니다.
{% endhint %}

---

## Step 2: 바이브 코딩 — 프롬프트 하나로 만들기

`PROJECT_SPEC.md` 파일이 저장되었으면, Kiro 채팅에 아래 프롬프트를 보냅니다:

{% code title="📋 복사해서 붙여넣기" %}
```
PROJECT_SPEC.md 파일을 읽고, 그대로 구현해주세요.
한글로 작업해주세요.
```
{% endcode %}

Kiro가 스펙 파일을 읽고 **agent.py, server.py, index.html** 3개 파일을 자동으로 만들어줍니다.

{% hint style="info" %}
**Part 1과의 차이점**: Part 1에서는 프롬프트를 여러 번 나눠서 보내고, 요구사항 → 설계 → 태스크 문서를 거쳤습니다. 이번에는 **스펙 파일 하나 + 프롬프트 하나**로 끝! 파일이 3개뿐인 작은 프로젝트에서는 이 방식이 더 빠르고 효율적입니다.
{% endhint %}

<details>
<summary>💡 바이브 코딩 vs Spec-driven, 언제 뭘 쓸까?</summary>

| 상황 | 추천 방식 |
|------|-----------|
| 파일 3~5개 이하의 작은 프로젝트 | **바이브 코딩** — 빠르게 만들고 바로 테스트 |
| 파일 10개 이상, 복잡한 구조 | **Spec-driven** — 체계적으로 문서화하고 단계별 구현 |
| 프로토타입 / 해커톤 | **바이브 코딩** — 속도가 중요 |
| 팀 협업 / 프로덕션 | **Spec-driven** — 문서가 커뮤니케이션 역할 |

</details>

---

## Step 3: Knowledge Base ID 설정

Kiro가 생성한 `agent.py` 파일을 열어서, Knowledge Base ID를 Module 4에서 메모한 값으로 교체합니다.

{% hint style="warning" %}
**`YOUR_KB_ID_HERE`** 또는 비슷한 플레이스홀더를 찾아서 **본인의 Knowledge Base ID**로 바꿔주세요!
{% endhint %}

---

## 완성 확인

프로젝트에 이런 파일들이 만들어져 있으면 성공입니다:

```
gs25-rag-agent/
├── PROJECT_SPEC.md  # 스펙 파일 (우리가 만든 것)
├── agent.py         # AI 에이전트 코드
├── server.py        # 웹 API 서버
└── index.html       # 채팅 화면
```

{% hint style="success" %}
코드가 모두 생성되었습니다! 다음 단계에서 실행해봅시다!
{% endhint %}
