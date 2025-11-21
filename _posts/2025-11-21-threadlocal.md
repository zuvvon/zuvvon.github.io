---
title: "[Java] ThreadLocal(쓰레드 로컬): Thread의 개인 수납장"
excerpt: "각 스레드마다 독립적인 데이터를 저장하는 ThreadLocal의 원리와 Spring에서의 활용, 그리고 스레드풀 환경에서 반드시 주의해야 할 점 알아보기"

categories:
  - Java
tags:
  - [Java, ThreadLocal]

permalink: /categories1/threadlocal/ # 포스트 URL

toc: true
toc_sticky: true

date: 2025-11-21
last_modified_at: 2025-11-21
---

## ⚡요약
ThreadLocal은 스레드별 저장소이며 스레드풀에서는 remove()가 필요하다.
@RequestScope는 HTTP 요청에서만 동작하므로, Spring 내부 동작 대부분은 ThreadLocal 기반이다.

## 🧩 ThreadLocal 이란?

ThreadLocal은 Java에서 각 스레드마다 독립적인 변수를 저장할 수 있도록 도와주는 클래스이다. 보통 여러 스레드가 공유 자원을 사용하면 동시성 문제가 발생할 수 있는데,ThreadLocal을 사용하면 스레드별로 데이터를 분리할 수 있어 동기화 없이 안전하게 활용할 수 있다. 각 스레드는 자신만의 ThreadLocalMap을 가지고 있고 ThreadLocal을 키로 사용하여 값을 저장한다. Spring에서는 트랜잭션 동기화 관리, 사용자 인증 정보 관리, 웹 요청의 attribute 관리 등의 기능을 제공하고 있다.

즉, ThreadLocal은 스레드별로 독립적인 저장소를 제공해 동시성 문제를 피할 수 있다.

---

## 🚨 ThreadLocal을 사용할 때 주의할 점

스레드풀을 사용하면 스레드가 재사용 되는데, 이때 ThreadLocal에 이전 스레드의 값이 남아있으면 재사용된 스레드가 올바르지 않은 값을 참조할 수 있다. 따라서 스레드가 끝나는 시점에 remove() 메서드를 호출하여 저장된 값을 제거해줘야 한다.

비동기 작업을 수행할 때 ThreadLocal이 예상대로 동작하지 않을 수 있다. 예를 들어, @Async 어노테이션을 사용하면 새로운 스레드에서 비동기 작업이 실행되는데, 비동기 스레드는 기존 스레드에서 ThreadLocal에 저장한 값을 참조할 수 없기 때문이다. 하지만 Spring 4.3 에서 제공하는 TaskDecorator를 사용하면 기존 스레드의 ThreadLocal 값을 비동기 스레드에 복사하는 방식으로 해결할 수 있다.

---

## 🔧 ThreadLocal을 대체할 수 있는 방법은?

ThreadLocal은 스레드마다 독립된 데이터를 저장할 수 있어 편리하지만, 꼭 ThreadLocal을 사용하지 않아도 비슷한 목적을 달성하는 대안이 존재한다.

| 대체 방법 | 설명 | 장점 | 단점 |
|----------|------|------|------|
| 메서드 인자 전달 | 필요한 값을 메서드 파라미터로 직접 전달 | 명확하고 안전함 | 호출 깊어질수록 파라미터 증가, 유지보수 어려움 |
| ConcurrentHashMap | 스레드 ID를 key로 사용하는 thread-safe Map 사용 | ThreadLocal 누수 문제 직접 방지 가능 | 관리 복잡, 성능 저하, 거의 실무에서 안 씀 |
| @RequestScope (Spring) | HTTP 요청마다 객체 생성/관리 | 생명주기 자동 관리, HTTP 요청 단위 데이터 처리에 적합 | HTTP 요청에서만 동작, 비HTTP 흐름(Filter, AOP, @Async 등)에는 사용 불가 |

이런 단점들로 대안이 존재해도 실무에서는 여전히 ThreadLocal을 사용하는 경우가 많다. 특히 traceId, userId 같은 컨텍스트 정보를 메서드 인자로 계속 전달하지 않아도 되고, 성능도 가볍기 때문에 다양한 환경에서 자연스럽게 적용되는 장점이 있다.

---

## 🏷️ NamedThreadLocal이란?

NamedThreadLocal은 Spring에서 제공하는 ThreadLocal의 확장 클래스로, 디버깅을 쉽게 하기 위해 이름을 부여할 수 있도록 설계되었다. 기본적인 기능은 ThreadLocal과 같지만, 여러 개의 ThreadLocal을 사용할 때 이름을 명확히 설정하면 어떤 목적의 ThreadLocal인지 구분할 수 있어 디버깅이 용이하다.

