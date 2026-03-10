---
description: Kiro가 자동으로 요구사항, 설계, 태스크 문서를 만들어주는 과정입니다
---

# Module 2: Spec-driven Development

## 이 모듈에서 할 것

이전 모듈에서 Kiro에게 프로젝트를 충분히 설명했습니다. 이제 Kiro가 **3가지 문서를 자동으로 만들어**줍니다:

```mermaid
graph LR
    R["📋 요구사항\nrequirements.md"] --> D["🏗️ 설계\ndesign.md"]
    D --> T["✅ 태스크\ntasks.md"]

    style R fill:#f5576c,color:#fff
    style D fill:#4facfe,color:#fff
    style T fill:#43e97b,color:#fff
```

{% hint style="info" %}
**Spec이란?** Specification(명세서)의 줄임말로, "무엇을, 어떻게 만들지 정리한 문서"를 뜻합니다. 건축에서 설계 도면과 같은 것이에요.
{% endhint %}

### 각 문서가 하는 역할

| 문서 | 쉬운 설명 | 비유 |
|------|-----------|------|
| **요구사항** (requirements.md) | "이런 기능이 필요해요" 목록 | 음식 주문서 📝 |
| **설계** (design.md) | "이렇게 구조를 잡겠습니다" 설계도 | 요리 레시피 📖 |
| **태스크** (tasks.md) | "이 순서로 만들겠습니다" 체크리스트 | 요리 순서도 ✅ |

여러분이 할 일은 **각 문서를 만들어달라고 프롬프트를 보내는 것**뿐입니다!

## 모듈 구성

1. [Requirements - 요구사항 문서](requirements.md) — "뭘 만들지" 정리
2. [Design - 설계 문서](design.md) — "어떻게 만들지" 설계
3. [Tasks - 구현 태스크](tasks.md) — "어떤 순서로 만들지" 분해
