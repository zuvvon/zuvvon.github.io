---
title: "[CS] SQL Injection"
excerpt: "SQL 인젝션 발생 원리, 대표 공격 패턴, PreparedStatement와 JPA 방어 방식 정리"
categories:
  - CS
tags:
  - [CS, SQL-Injection, PreparedStatement, JPA]

permalink: /categories1/sql-injection/
toc: true
toc_sticky: true

date: 2025-12-02
last_modified_at: 2025-12-02
---

## ⚡요약

SQL Injection은 **사용자 입력이 SQL 구조에 그대로 포함될 때** 발생하는 보안 취약점이다.  
공격자는 `' OR '1'='1` 같은 문자열로 **인증 우회**, **데이터 탈취**, 심지어 **테이블 삭제**까지 수행할 수 있다.

---

## 🧨 어떻게 발생할까?

문자열 연결로 SQL을 만들면 입력값이 SQL 구문으로 해석된다.

```java
String sql = "SELECT * FROM users WHERE username = '" + username + "'";
```

공격자 입력:
```
' OR '1'='1
```

실제 실행되는 쿼리:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1'
```

➡ **항상 true가 되어 비밀번호 없이 로그인된다.**

---

## 🔥 대표 페이로드

| 페이로드 | 의미 |
|---------|------|
| `' OR '1'='1` | 인증 우회 |
| `' UNION SELECT * FROM accounts --` | 다른 테이블 데이터 노출 |
| `'; DROP TABLE users; --` | 테이블 삭제 |

---

## 🛡️ PreparedStatement는 어떻게 막을까?

**핵심 원리**

- SQL 구조(statement)와 입력값(parameter)을 **완전히 분리**
- 입력값은 절대 SQL 코드로 해석되지 않음
- `'`, `--`, `;` 같은 위험 문자는 내부적으로 자동 escape 처리

예시:

```java
String sql = "SELECT * FROM users WHERE username = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, username);
```

- **쿼리 구조 고정**
- **입력값은 순수 데이터로 처리됨**

➡ 따라서 **SQL Injection을 원천 차단**한다.

---

## 🟪 JPA(Hibernate)는 왜 안전할까?

JPA는 내부적으로 **PreparedStatement 기반**으로 SQL을 실행한다.

```java
em.createQuery("SELECT u FROM User u WHERE u.username = :name")
  .setParameter("name", username)
  .getSingleResult();
```

- `.setParameter()`는 **파라미터 바인딩 방식**
- SQL 구조는 절대 입력값에 의해 변경되지 않는다  
→ 인젝션 발생 가능성이 매우 낮다.

⚠️ 단, 문자열을 직접 조합하면 인젝션이 발생할 수 있다.

```java
// ❌ 위험한 코드
String q = "SELECT u FROM User u WHERE u.username = '" + username + "'";
```

---

## 🧭 실무 방어 체크리스트

- PreparedStatement 사용  
- JPA/MyBatis **파라미터 바인딩** 사용  
- 문자열로 SQL 직접 생성하지 않기  
- DB 계정 최소 권한(DROP/ALTER 금지)  
- 오류 메시지 외부 노출 금지  

---

## ✨ 한 줄 결론

<mark>**SQL Injection 방어 핵심: “쿼리와 데이터를 절대 섞지 않는다.”**</mark>
