---
title: "[Java] 스레드 풀 포화 정책"
excerpt: "스레드 풀(ThreadPoolExecutor)이 포화 상태가 되었을 때 동작하는 RejectedExecutionHandler의 4가지 정책 알아보기"
categories:
  - Java
tags:
  - [Java, Saturation-Policy]

permalink: /categories1/saturation-policy/ # 포스트 URL

toc: true
toc_sticky: true

date: 2025-11-24
last_modified_at: 2025-11-24
---

## ⚡요약

스레드 풀은 일정 수준의 부하까지는 빠르게 작업을 소화하지만, **maxPoolSize까지 스레드가 증가하고 workQueue까지 가득 찬 순간**에는 더 이상 작업을 받을 수 없다. 이 상태를 *포화(Saturation)*라고 하고, Java는 이때 실행할 행동을 **RejectedExecutionHandler**, 즉 *포화 정책*으로 정의한다.   
포화 정책은 4가지가 기본 제공되며, 상황에 따라 다양한 방식으로 서비스 안정성을 유지할 수 있다.

<br><br>
---

## 🧩 스레드 풀 포화 정책이란?

**스레드 풀(ThreadPoolExecutor)이 더 이상 작업을 처리할 수 없을 때 선택하는 대응 전략**을 의미한다. 스레드 풀은 아래 요소로 구성된다.

- **corePoolSize**: 항상 유지되는 스레드 수  
- **maxPoolSize**: 최대로 늘릴 수 있는 스레드 수  
- **workQueue**: 실행 대기 작업이 쌓이는 큐  

스레드 수가 `maxPoolSize`까지 증가하고 `workQueue`까지 꽉 찬 상태가 되면,  
새로운 작업은 바로 실행할 수 없고 **RejectedExecutionHandler**, 즉 포화 정책이 실행된다.

---

## 🚨 제공되는 4가지 포화 정책

ThreadPoolExecutor는 아래 네 가지 정책을 기본적으로 제공한다.

### 1️⃣ AbortPolicy (기본값)
- 새 작업을 거부하고 **RejectedExecutionException을 발생**
- 클라이언트가 예외를 감지해 후속 처리 가능

### 2️⃣ DiscardPolicy
- **예외 없이** 새 작업을 조용히 버림
- 일부 작업 손실이 허용되는 시스템에 적합

### 3️⃣ DiscardOldestPolicy
- 대기열에서 **가장 오래된 작업을 제거**  
- 제거한 자리에는 새 작업을 삽입  
- 최신 작업 우선 처리 전략에 맞는 정책

### 4️⃣ CallerRunsPolicy
- 새 작업을 제출한 **호출 스레드에서 직접 실행**  
- 작업 처리 속도를 자연스럽게 늦추는 *백프레셔* 역할

<mark>즉, 포화 정책은 단순한 예외 처리를 넘어,서비스 안정성과 처리 전략을 결정하는 중요한 구성 요소다.</mark>

---

## 🛠 커스텀 포화 정책 구현

직접 정책을 정의해야 한다면 `RejectedExecutionHandler`를 구현하면 된다.

```java
class CustomPolicy implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        // 모니터링, 로그 출력, 대안 큐 적재 등 원하는 처리
        System.out.println("작업 거부됨: " + r.toString());
    }
}
```
