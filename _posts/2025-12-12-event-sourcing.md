---
title: "[CS] 이벤트 소싱(Event Sourcing)과 함께 등장한 핵심 개념들"
excerpt: "Event Sourcing의 기본 개념부터 읽기 성능, Projection, Snapshot, 멱등성, CQRS까지 흐름으로 정리"

categories:
  - CS
tags:
  - [CS, Event-Sourcing, CQRS]

permalink: /categories1/event-sourcing/
toc: true
toc_sticky: true

date: 2025-12-12
last_modified_at: 2025-12-12
---

## ⚡ 한 줄 요약
Event Sourcing은 **이벤트를 진실의 원천으로 삼는 대신 읽기 성능 문제가 발생하는 구조**이기 때문에,  
이를 보완하기 위해 **집계 테이블(Projection), 스냅샷(Snapshot), 멱등성, CQRS** 같은 개념이 함께 사용된다.

---

## 🧩 Event Sourcing이란?

Event Sourcing은 데이터의 **최종 상태(State)** 를 저장하는 대신,  
상태를 변경시킨 **모든 이벤트(Event)의 이력**을 순서대로 저장하는 방식이다.

전통적인 데이터 저장 방식은  
“현재 상태가 무엇인가”에 집중한다.

반면 Event Sourcing은  
“어떤 일이 어떤 순서로 발생했는가”를 저장한다.

즉, 상태는 결과일 뿐이고  
**진짜 데이터는 상태를 만들어낸 이벤트의 흐름**이라는 관점이다.

---

## 🛒 주문·결제 서비스 예제로 이해하기

온라인 쇼핑몰의 주문 서비스를 예로 들어보자.  
주문 데이터를 저장하는 방법에는 크게 두 가지가 있다.

### 🟦 1) 최종 상태를 저장하는 방식

주문 테이블에 현재 상태만 저장한다.

예시)
order_id = 101  
status = COMPLETED  
total_price = 50,000  

이 방식은 현재 상태를 빠르게 조회할 수 있고 구조가 단순하다.  
하지만 주문이 결제 실패를 겪었는지, 재시도가 있었는지,  
중간에 취소 요청이 있었는지와 같은 **과정과 이력은 알기 어렵다**.

---

### 🟩 2) 이벤트를 저장하는 방식 (Event Sourcing)

주문의 최종 상태를 직접 저장하지 않고,  
주문 상태를 변경시킨 이벤트를 순서대로 기록한다.

예시)
[1] OrderCreated  
[2] PaymentRequested  
[3] PaymentFailed  
[4] PaymentSucceeded  
[5] OrderCompleted  

이벤트를 처음부터 순서대로 재생하면  
언제든지 특정 시점의 주문 상태를 정확히 복원할 수 있다.

이처럼 **상태의 결과가 아니라, 상태를 만든 과정 자체를 저장하는 방식**이  
Event Sourcing이다.

---

## ❓ 왜 Event Sourcing은 읽기 성능이 떨어질까?

전통적인 방식에서는 이미 계산된 현재 상태를 바로 조회한다.

SELECT balance FROM account WHERE account_id = 1;

하지만 Event Sourcing에서는  
현재 상태가 데이터베이스에 바로 저장되어 있지 않다.

예시 이벤트)
[1] Deposit  +100  
[2] Withdraw -30  
[3] Deposit  +50  

현재 잔액을 알기 위해서는  
이벤트를 처음부터 끝까지 재생하며 누적 계산해야 한다.

이벤트 수가 많아질수록  
읽기 비용은 급격히 증가한다.

결론적으로,  
Event Sourcing은 **쓰기(write)는 단순하지만 읽기(read)는 계산 비용이 큰 구조**를 가진다.

---

## 🧮 집계 테이블(Projection)이란?

이 읽기 성능 문제를 해결하기 위해 등장한 개념이  
**집계 테이블(Projection)** 이다.

Projection은  
이벤트를 기반으로 미리 계산해 둔 **읽기 전용 결과 테이블**이다.

이벤트는 진실의 원천(Source of Truth)으로 유지하고,  
조회는 Projection만 수행한다.

예시)
account_id | balance  
-----------|--------  
1          | 120  

화면이나 API 요청은 Projection만 조회하기 때문에  
읽기 성능을 크게 개선할 수 있다.

---

## 📸 스냅샷(Snapshot)이란?

이벤트가 계속 쌓이면  
Projection을 다시 만드는 작업조차 오래 걸릴 수 있다.

이를 보완하기 위해  
**특정 시점의 상태를 통째로 저장하는 방식**이 스냅샷이다.

예시)
Snapshot (version = 1000)  
balance = 120  

이후에는  
스냅샷 이후의 이벤트만 재생하면 되므로  
전체 이벤트를 처음부터 재생할 필요가 없다.

정리하면,
- Projection은 조회 성능을 위한 장치이고
- Snapshot은 이벤트 재생 성능을 위한 장치이다.

---

## 🔁 멱등성(Idempotency)은 왜 중요한가?

멱등성이란  
**같은 요청을 여러 번 처리해도 결과는 한 번 처리한 것과 같아야 하는 성질**이다.

실무 환경에서는
- 네트워크 타임아웃
- 메시지 재시도
- API 중복 호출

같은 상황이 빈번하게 발생한다.

멱등성이 없으면  
포인트 중복 적립, 이중 지급과 같은 치명적인 문제가 발생할 수 있다.

Event Sourcing에서는  
- idempotency_key
- (ref_id + event_type)에 대한 UNIQUE 제약
- 이벤트 중복 저장 방지

와 같은 방식으로 멱등성을 보장한다.

금융·은행 시스템에서는  
멱등성은 선택이 아니라 **필수 조건**이다.

---

## 🔀 CQRS란?

CQRS(Command Query Responsibility Segregation)는  
**쓰기(Command)와 읽기(Query)를 분리하는 설계 패턴**이다.

Event Sourcing에서는
- 쓰기: 이벤트 저장
- 읽기: 계산된 결과 조회

로 역할이 명확히 나뉘기 때문에  
자연스럽게 CQRS 구조가 잘 어울린다.

구조적으로는 다음과 같다.

Command 요청  
→ 이벤트 저장(Event Store)  
→ Projection 업데이트  

Query 요청  
→ Projection 조회  

쓰기 모델은 정합성과 비즈니스 규칙에 집중하고,  
읽기 모델은 조회 성능에 최적화된다.

Event Sourcing이 CQRS를 필수로 요구하지는 않지만,  
실무에서는 거의 함께 사용된다.

---

## 🔄 전체 흐름 정리

1. Command 요청이 발생한다  
2. 멱등성 체크로 중복 처리를 방지한다  
3. 이벤트를 append-only 방식으로 저장한다  
4. Projection(집계 테이블)을 업데이트한다  
5. 조회는 Projection만 사용한다  
6. 필요 시 Snapshot부터 이벤트를 재생한다  

---

## 🧾 마무리 정리

- Event Sourcing은 **상태가 아닌 이벤트를 저장하는 방식**이다  
- 이벤트 재생 구조로 인해 **읽기 성능 이슈가 발생**한다  
- 이를 해결하기 위해 **Projection과 Snapshot**이 사용된다  
- **멱등성**은 중복 처리 방지를 위한 핵심 개념이다  
- **CQRS**는 읽기와 쓰기를 분리해 각자에 최적화한다  

Event Sourcing은 강력하지만  
모든 시스템에 적합한 해법은 아니다.  
도메인의 특성과 요구사항에 맞게 선택하는 것이 가장 중요하다.
