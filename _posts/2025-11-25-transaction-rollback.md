---
title: "[CS] 트랜잭션 롤백"
excerpt: "Spring @Transactional의 예외 처리 규칙과 Checked/Unchecked 예외별 롤백 동작을 정리"

categories:
  - CS
tags:
  - [CS, Transaction, JPA]

permalink: /categories1/transaction-rollback/
toc: true
toc_sticky: true

date: 2025-11-25
last_modified_at: 2025-11-25
---

## ⚡ 요약
Spring 트랜잭션(@Transactional)은 기본적으로 **Unchecked Exception(RuntimeException, Error)**이 발생하면 롤백하고,  
**Checked Exception(Exception 계열)**이 발생하면 커밋하는 규칙을 따른다. 필요하면 `rollbackFor`, `noRollbackFor`로 동작을 재정의할 수 있다.

---

## 🧩 트랜잭션 롤백 기본 규칙

Spring의 기본 예외 처리 방식은 다음과 같다.

### 🟦 Unchecked Exception → **롤백됨**
- **RuntimeException** : 모든 런타임 오류의 상위 클래스이며 실행 중 발생하는 예외이다.
- **NullPointerException** : null 객체에 접근할 때 발생한다.
- **IllegalArgumentException** : 메서드에 잘못된 인자가 전달될 때 발생한다.
- **IllegalStateException** : 현재 객체 상태로는 요청된 동작을 수행할 수 없을 때 발생한다.
- **DataAccessException** : Spring이 JDBC/JPA 예외를 런타임 예외로 변환한 공통 데이터 접근 예외 계층이다.
- **Error** : OutOfMemoryError 등 JVM의 치명적 오류로 무조건 롤백된다.

### 🟩 Checked Exception → **롤백되지 않음**
- **Exception** : 모든 Checked Exception의 상위 클래스이다.
- **IOException** : 파일, 네트워크 등 I/O 처리 중 발생하는 예외이다.
- **SQLException** : JDBC DB 오류이며 Spring이 DataAccessException으로 변환할 경우 롤백된다.
- **ParseException** : 날짜/문자열 파싱 형식이 맞지 않을 때 발생한다.

---

## 🧪 롤백 동작 커스텀 예시

### ✔ Checked 예외도 롤백시키기
```java
@Transactional(rollbackFor = Exception.class)
public void save() throws Exception {
    throw new Exception("Checked 예외");
}
```

### ✔ 특정 Unchecked 예외는 롤백 제외하기
```java
@Transactional(noRollbackFor = IllegalArgumentException.class)
public void update() {
    throw new IllegalArgumentException("기본은 롤백되지만 제외함");
}
```

---

## 🚨 주의할 점

### 1) Spring은 SQLException을 자동으로 RuntimeException으로 변환한다
→ DataAccessException으로 감싸서 전달되므로 기본적으로 롤백된다.

### 2) @Transactional은 프록시 기반이다  
같은 클래스 내부 호출은 트랜잭션이 적용되지 않을 수 있다.

### 3) Checked Exception을 롤백시키고 싶다면 반드시 rollbackFor를 지정해야 한다  
기본 규칙으로는 커밋된다.

---

## 📌 마무리 정리

- Spring 기본 규칙  
  - **Unchecked → 롤백 / Checked → 커밋**
- 하위 계층(JDBC, JPA) 예외는 Spring이 DataAccessException(런타임)으로 바꾼다  
  → DB 예외는 대부분 롤백된다  
- `rollbackFor`, `noRollbackFor`로 원하는 방식으로 조정 가능하다

트랜잭션 롤백 기준을 정확히 이해하면 서비스 안정성과 예외 처리 흐름을 훨씬 강하게 만들 수 있다.

