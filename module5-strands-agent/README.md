---
description: Kiro를 사용해서 Strands RAG 에이전트를 만들고 채팅 UI로 대화해봅니다
---

# Module 5: Strands RAG 에이전트

## 이 모듈에서 할 것

Module 4에서 만든 Knowledge Base를 활용하는 **AI 채팅 에이전트**를 만듭니다. 이번에는 Part 1과 다른 방식으로 — **스펙 파일 하나 + 프롬프트 하나**로 바이브 코딩합니다!

```mermaid
graph LR
    A["스펙 파일<br/>만들기"] --> B["프롬프트 하나로<br/>바이브 코딩"]
    B --> C["실행하면<br/>채팅 UI 완성!"]

    style A fill:#43e97b,color:#fff
    style B fill:#4facfe,color:#fff
    style C fill:#fa709a,color:#fff
```

{% hint style="info" %}
**Part 1과의 차이**: Part 1에서는 여러 프롬프트 → 요구사항 → 설계 → 태스크를 거쳤습니다. 이번에는 **스펙 파일을 미리 준비해두고 프롬프트 하나**로 끝내는 바이브 코딩 방식을 체험합니다!
{% endhint %}

## 사전 준비

* Python 3.10+ 설치
* AWS CLI 설정 완료 (Module 4에서 했음)
* Module 4에서 메모한 **Knowledge Base ID**

## 모듈 구성

1. [Strands Agent 소개](strands-intro.md) — AI 에이전트가 뭔지 알아보기
2. [Kiro로 RAG 에이전트 만들기](rag-agent.md) — Kiro에게 프롬프트로 요청하기
3. [채팅 UI 실행](chat-ui.md) — 에이전트와 대화하기
