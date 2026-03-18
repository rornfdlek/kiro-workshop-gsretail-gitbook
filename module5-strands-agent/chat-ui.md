---
description: 예쁜 HTML 채팅 UI를 만들고 AI 에이전트와 대화합니다
---

# 채팅 UI 실행

## HTML 채팅 화면 만들기

에이전트와 대화할 수 있는 깔끔한 채팅 인터페이스를 만듭니다. 아래 코드를 `index.html`로 저장하세요.

<details>
<summary>index.html 전체 코드 (클릭해서 펼치기)</summary>

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GS25 AI 매장 도우미</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .chat-container {
            width: 480px; height: 700px; background: #fff;
            border-radius: 20px; box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            display: flex; flex-direction: column; overflow: hidden;
        }
        .chat-header {
            background: linear-gradient(135deg, #0066cc, #004499);
            color: white; padding: 20px 24px;
            display: flex; align-items: center; gap: 12px;
        }
        .chat-header .logo {
            width: 44px; height: 44px; background: #fff; border-radius: 12px;
            display: flex; align-items: center; justify-content: center;
            font-weight: 800; font-size: 11px; color: #0066cc;
        }
        .chat-header .title h1 { font-size: 17px; font-weight: 700; }
        .chat-header .title p { font-size: 12px; opacity: 0.85; margin-top: 2px; }
        .chat-messages {
            flex: 1; overflow-y: auto; padding: 20px;
            display: flex; flex-direction: column; gap: 16px; background: #f5f7fb;
        }
        .message {
            max-width: 85%; padding: 12px 16px; border-radius: 16px;
            font-size: 14px; line-height: 1.5; word-wrap: break-word; white-space: pre-wrap;
        }
        .message.bot {
            align-self: flex-start; background: #fff; color: #333;
            border: 1px solid #e8e8e8; border-bottom-left-radius: 4px;
        }
        .message.user {
            align-self: flex-end; background: linear-gradient(135deg, #0066cc, #004499);
            color: white; border-bottom-right-radius: 4px;
        }
        .message.typing {
            align-self: flex-start; background: #fff;
            border: 1px solid #e8e8e8; border-bottom-left-radius: 4px; color: #999;
        }
        .typing-dots span { animation: blink 1.4s infinite both; font-size: 20px; }
        .typing-dots span:nth-child(2) { animation-delay: 0.2s; }
        .typing-dots span:nth-child(3) { animation-delay: 0.4s; }
        @keyframes blink { 0%, 80%, 100% { opacity: 0.3; } 40% { opacity: 1; } }
        .chat-input {
            padding: 16px 20px; background: #fff;
            border-top: 1px solid #eee; display: flex; gap: 10px;
        }
        .chat-input input {
            flex: 1; padding: 12px 16px; border: 2px solid #e8e8e8;
            border-radius: 12px; font-size: 14px; outline: none;
        }
        .chat-input input:focus { border-color: #0066cc; }
        .chat-input button {
            padding: 12px 20px; background: linear-gradient(135deg, #0066cc, #004499);
            color: white; border: none; border-radius: 12px;
            font-size: 14px; font-weight: 600; cursor: pointer;
        }
        .chat-input button:disabled { opacity: 0.5; cursor: not-allowed; }
        .suggestions {
            display: flex; flex-wrap: wrap; gap: 8px;
            padding: 0 20px 12px; background: #f5f7fb;
        }
        .suggestion {
            padding: 8px 14px; background: #fff; border: 1px solid #ddd;
            border-radius: 20px; font-size: 12px; color: #555; cursor: pointer;
        }
        .suggestion:hover { border-color: #0066cc; color: #0066cc; background: #f0f4ff; }
    </style>
</head>
<body>
    <div class="chat-container">
        <div class="chat-header">
            <div class="logo">GS25</div>
            <div class="title">
                <h1>AI 매장 도우미</h1>
                <p>매장 운영 / 발주 / 상품 정보 질문하세요</p>
            </div>
        </div>
        <div class="chat-messages" id="chatMessages">
            <div class="message bot">안녕하세요! GS25 AI 매장 도우미입니다.
매장 운영, 발주 정책, 상품 정보에 대해 무엇이든 물어보세요.</div>
        </div>
        <div class="suggestions" id="suggestions">
            <span class="suggestion" onclick="sendSuggestion(this)">도시락 발주량 계산법</span>
            <span class="suggestion" onclick="sendSuggestion(this)">오전 근무 체크리스트</span>
            <span class="suggestion" onclick="sendSuggestion(this)">비 올 때 발주 정책</span>
            <span class="suggestion" onclick="sendSuggestion(this)">인기 상품 TOP5</span>
        </div>
        <div class="chat-input">
            <input type="text" id="userInput" placeholder="질문을 입력하세요..."
                   onkeydown="if(event.key==='Enter') sendMessage()">
            <button id="sendBtn" onclick="sendMessage()">전송</button>
        </div>
    </div>
    <script>
        const API_URL = "http://localhost:8000";
        const chatMessages = document.getElementById("chatMessages");
        const userInput = document.getElementById("userInput");
        const sendBtn = document.getElementById("sendBtn");
        const suggestions = document.getElementById("suggestions");

        function addMessage(text, isUser) {
            const div = document.createElement("div");
            div.className = `message ${isUser ? "user" : "bot"}`;
            div.textContent = text;
            chatMessages.appendChild(div);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }
        function showTyping() {
            const div = document.createElement("div");
            div.className = "message typing"; div.id = "typingIndicator";
            div.innerHTML = '<span class="typing-dots"><span>●</span><span>●</span><span>●</span></span>';
            chatMessages.appendChild(div);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }
        function removeTyping() {
            const el = document.getElementById("typingIndicator");
            if (el) el.remove();
        }
        async function sendMessage() {
            const text = userInput.value.trim();
            if (!text) return;
            addMessage(text, true);
            userInput.value = "";
            sendBtn.disabled = true;
            suggestions.style.display = "none";
            showTyping();
            try {
                const res = await fetch(`${API_URL}/chat`, {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ message: text })
                });
                const data = await res.json();
                removeTyping();
                addMessage(data.reply, false);
            } catch (err) {
                removeTyping();
                addMessage("서버 연결에 실패했습니다. 서버가 실행 중인지 확인해주세요.", false);
            }
            sendBtn.disabled = false;
            userInput.focus();
        }
        function sendSuggestion(el) {
            userInput.value = el.textContent;
            sendMessage();
        }
    </script>
</body>
</html>
```

</details>

---

## 파일 구조 확인

최종적으로 이런 파일들이 있어야 합니다:

```
gs25-rag-agent/
├── .venv/          # Python 가상환경
├── agent.py        # AI 에이전트 코드
├── server.py       # 웹 API 서버
└── index.html      # 채팅 화면
```

---

## 실행하기

### 서버 시작

```bash
cd gs25-rag-agent
source .venv/bin/activate
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
**축하합니다!** Bedrock Knowledge Base + Strands Agent + HTML 채팅 UI를 연결한 **완전한 RAG 에이전트 시스템**을 구축했습니다!
{% endhint %}

---

## 문제가 생기면?

<details>
<summary>ModuleNotFoundError: No module named 'strands'</summary>

가상환경이 활성화되었는지 확인하세요:

```bash
source .venv/bin/activate
pip install strands-agents strands-agents-bedrock
```

</details>

<details>
<summary>botocore.exceptions.NoCredentialsError</summary>

AWS 자격 증명이 설정되었는지 확인하세요:

```bash
aws configure
# 또는 환경 변수 설정
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
export AWS_DEFAULT_REGION=us-east-1
```

</details>

<details>
<summary>Knowledge Base 검색 결과가 없음</summary>

1. `agent.py`의 `KNOWLEDGE_BASE_ID`가 정확한지 확인
2. Knowledge Base의 Sync가 완료되었는지 확인 (Status: Available)
3. 리전이 `us-east-1`인지 확인

</details>
