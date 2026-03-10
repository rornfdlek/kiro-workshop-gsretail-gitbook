---
description: 설계 문서 기반 구현 태스크 자동 분해
---

# Tasks - 구현 태스크 정의

## 태스크 문서 작성 요청

설계 문서가 완성되었으면, Kiro에게 구현 태스크를 정의하도록 요청합니다.

{% code title="프롬프트" %}
```
Task를 정의해주세요.
```
{% endcode %}

Kiro가 `design.md`를 기반으로 `tasks.md` 파일을 자동 생성합니다.

## 생성되는 태스크 구조

Kiro는 설계 문서를 분석하여 구현에 필요한 태스크를 자동으로 분해합니다. 일반적으로 다음과 같은 태스크들이 생성됩니다:

```mermaid
graph TD
    T1[Task 1<br/>프로젝트 초기 설정] --> T2[Task 2<br/>DB 스키마 및 초기 데이터]
    T2 --> T3[Task 3<br/>Backend API 구현]
    T3 --> T4[Task 4<br/>Frontend 컴포넌트 구현]
    T4 --> T5[Task 5<br/>발주 자동 계산 로직]
    T5 --> T6[Task 6<br/>대시보드 및 차트]
    T6 --> T7[Task 7<br/>판매 시뮬레이터]

    style T1 fill:#e8f5e9
    style T2 fill:#e1f5fe
    style T3 fill:#fff3e0
    style T4 fill:#f3e5f5
    style T5 fill:#fce4ec
    style T6 fill:#e0f2f1
    style T7 fill:#fff8e1
```

### 예상 태스크 목록

| 태스크 | 설명 |
|--------|------|
| 프로젝트 초기화 | package.json, TypeScript 설정, 디렉토리 구조 |
| DB 스키마 생성 | SQLite 테이블 생성 및 시드 데이터 |
| REST API 구현 | 상품/재고/발주/판매 CRUD 엔드포인트 |
| Frontend 구현 | React 컴포넌트 및 라우팅 |
| 발주 계산 엔진 | 7일 판매 데이터 기반 자동 발주량 계산 |
| 대시보드 | Chart.js 기반 판매 분석 차트 |
| 판매 시뮬레이터 | 가상 판매 데이터 생성 기능 |

{% hint style="info" %}
태스크의 수와 내용은 Kiro가 자동으로 결정하므로, 실습자마다 다를 수 있습니다.
{% endhint %}

## 태스크 문서 검토

`tasks.md` 파일이 생성되면 다음을 확인합니다:

* 각 태스크에 체크박스(`[ ]`)가 있는지
* 태스크 간 의존성이 합리적인지
* 누락된 기능이 없는지

{% hint style="warning" %}
태스크가 너무 많거나 세분화되어 있다면, Kiro에게 "태스크를 좀 더 통합해주세요"라고 요청할 수 있습니다.
{% endhint %}
