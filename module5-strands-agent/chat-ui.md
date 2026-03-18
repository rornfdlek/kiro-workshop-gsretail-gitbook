---
description: Kiro가 만든 에이전트를 실행하고 채팅 UI에서 대화합니다
---

# 채팅 UI 실행

## 실행 준비

### Python 가상환경 만들기

Kiro 채팅에서 실행을 요청하기 전에, 먼저 터미널에서 가상환경을 준비합니다:

**Mac/Linux:**

```bash
cd gs25-rag-agent
python3 -m venv .venv
source .venv/bin/activate
pip install strands-agents fastapi uvicorn boto3
```

**Windows (PowerShell):**

```powershell
cd gs25-rag-agent
python -m venv .venv
.venv\Scripts\Activate
pip install strands-agents fastapi uvicorn boto3
```

{% hint style="warning" %}
**macOS에서 Homebrew Python을 사용하는 경우**, 시스템 Python에 직접 패키지를 설치할 수 없습니다 (`externally-managed-environment` 오류). **반드시 가상환경을 만든 후** 패키지를 설치해야 합니다!
{% endhint %}

{% hint style="info" %}
`strands-agents-bedrock`은 별도 패키지가 아니라 `strands-agents`에 포함되어 있으므로 따로 설치할 필요 없습니다.
{% endhint %}

---

## 실행하기

Kiro 채팅에 실행을 요청합니다:

{% code title="📋 복사해서 붙여넣기" %}
```
서버를 실행시켜주세요. uvicorn server:app --host 0.0.0.0 --port 8000 으로 실행하면 됩니다.
```
{% endcode %}

또는 터미널에서 직접 실행해도 됩니다:

```bash
uvicorn server:app --host 0.0.0.0 --port 8000
```

이런 메시지가 나오면 성공:

```
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### 브라우저에서 접속

웹 브라우저를 열고 주소창에 입력:

```
http://localhost:8000
```

---

## 테스트해보기

채팅 화면에서 다양한 질문을 해보세요!

| 질문 유형 | 예시 질문 |
|-----------|-----------|
| 상품 정보 | "카페25 아메리카노 가격이 얼마야?" |
| 발주 정책 | "안전재고는 어떻게 설정되어 있어?" |
| 운영 절차 | "야간 근무자의 업무는?" |
| 특수 상황 | "행사 기간에는 발주를 어떻게 해?" |
| 복합 질문 | "비오는 주말에 도시락 발주량은 어떻게 계산해?" |

### 잘 되고 있는지 체크

| 항목 | 확인 |
|------|------|
| Knowledge Base에서 검색된 정보로 답변하는가? | ☐ |
| 문서에 없는 내용에 대해 "정보가 없다"고 하는가? | ☐ |
| 한국어로 자연스럽게 답변하는가? | ☐ |
| 여러 문서의 정보를 조합하여 답변하는가? | ☐ |

{% hint style="success" %}
**축하합니다!** Kiro가 만들어준 코드로 **Bedrock Knowledge Base + Strands Agent + 채팅 UI**를 연결한 완전한 RAG 에이전트 시스템을 구축했습니다! 🎉
{% endhint %}

---

## 문제가 생기면?

<details>
<summary>🆘 에이전트가 "정보를 찾지 못했습니다"라고만 해요</summary>

1. `agent.py`의 **Knowledge Base ID**가 정확한지 확인
2. Knowledge Base의 **Sync**가 완료되었는지 확인 (Status: Available)
3. AWS CLI의 **리전**이 `us-east-1`인지 확인

</details>

<details>
<summary>🆘 ModuleNotFoundError가 나와요</summary>

가상환경이 활성화되었는지 확인하세요:

**Mac/Linux:**
```bash
source .venv/bin/activate
pip install strands-agents fastapi uvicorn boto3
```

**Windows (PowerShell):**
```powershell
.venv\Scripts\Activate
pip install strands-agents fastapi uvicorn boto3
```

</details>

<details>
<summary>🆘 AWS 자격 증명 오류가 나와요</summary>

```bash
aws configure
# AWS Access Key ID: [본인의 Access Key]
# AWS Secret Access Key: [본인의 Secret Key]
# Default region name: us-east-1
```

</details>

<details>
<summary>🆘 Kiro가 만든 코드에 오류가 있어요</summary>

Kiro에게 알려주면 됩니다:

{% code title="📋 복사해서 붙여넣기 (에러 메시지 부분을 교체)" %}
```
실행했는데 오류가 발생했어요. 터미널에 이런 에러가 나옵니다: [에러 메시지를 여기에 복사해서 붙여넣기]
```
{% endcode %}

Part 1에서 했던 것처럼, Kiro가 오류를 분석하고 자동으로 수정합니다!

</details>
