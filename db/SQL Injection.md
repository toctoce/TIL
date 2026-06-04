# SQL Injection

웹 애플리케이션이 사용자 입력을 SQL 문에 안전하게 처리하지 않아, 공격자가 악의적인 SQL 구문을 삽입하고 실행하게 만드는 공격이다.
이를 통해 데이터베이스의 기밀 정보를 탈취하거나 변조/삭제할 수 있다.

## 공격 방식

### 인증 우회

로그인 과정에서 입력값이 SQL 조건식으로 직접 들어가면, 공격자는 항상 참이 되는 조건을 삽입해 인증을 우회할 수 있다.

예를 들어 비밀번호를 `abc' OR '1' = '1`라고 입력하면 아래와 같은 SQL문이 실행된다.

```sql
SELECT * FROM users
WHERE username = 'admin'
AND password = 'abc' OR '1' = '1';
```

`'1' = '1'`은 항상 참이므로, 잘못 작성된 쿼리에서는 비밀번호 검증 로직이 무력화될 수 있다.

### SQL문 추가

입력값 뒤에 추가 SQL을 붙일 수 있는 환경이라면 데이터 변경이나 삭제도 가능하다.
세미콜론과 SQL문을 함께 작성할 수 있다.

id 입력할 때 `1'; DELETE FROM posts; --`를 입력한다면 아래와 같은 SQL문이 실행될 수 있다.

```sql
SELECT * FROM posts
WHERE id = '1'; DELETE FROM posts; --';
```

`;`는 기존 SQL문을 끝내고 새로운 SQL문을 시작한다.
`--` 뒤의 기존 작은따옴표는 주석 처리되므로 SQL 문법 오류를 피할 수 있다.

### Error-Based SQL Injection

공격자가 일부러 SQL 오류를 발생시켜, 에러 메시지에 노출되는 테이블명, 컬럼명, DB 종류 같은 정보를 수집하는 방식이다.

예를 들어 id 입력값에 작은따옴표를 넣어 SQL 문법 오류를 유도할 수 있다.

```sql
SELECT * FROM users
WHERE id = '1'';
```

이때 서버가 데이터베이스 에러 메시지를 그대로 응답하면 다음과 같은 정보가 노출될 수 있다.

```text
You have an error in your SQL syntax;
check the manual that corresponds to your MySQL server version
for the right syntax to use near ''' at line 2
```

또는 존재하지 않는 컬럼을 유도해 테이블 구조를 추측할 수도 있다.

```sql
SELECT * FROM users
ORDER BY 99;
```

```text
Unknown column '99' in 'order clause'
```

서버가 SQL 오류 메시지를 그대로 응답하면 공격자는 내부 데이터베이스 구조를 추측할 수 있다.

### Union-Based SQL Injection

공격자는 기존 조회 쿼리에 `UNION`을 사용해 다른 테이블의 데이터를 함께 조회하려고 시도할 수 있다.

```sql
SELECT title, content
FROM posts
WHERE id = '1'
UNION
SELECT username, password
FROM users;
```

사용자 정보, 권한 정보, 비밀번호 해시 같은 민감 데이터를 빼내는 데 사용될 수 있다.

### Blind SQL Injection

화면에 직접 데이터가 출력되지 않더라도, 응답 결과의 차이를 이용해 정보를 추측하는 방식이다.

- Boolean-Based : 조건이 참일 때와 거짓일 때 응답이 달라지는지 확인한다.
- Time-Based : 조건이 참일 때 일부러 응답을 지연시켜 시간 차이로 결과를 추측한다.

예를 들어 특정 조건이 참일 때만 응답 시간이 느려진다면, 공격자는 그 차이를 이용해 데이터를 한 글자씩 추측할 수 있다.

## 방어 방식

SQL Injection 방어는 하나의 방법으로 끝나는 것이 아니라, 여러 방어 계층을 함께 적용해야 한다.
가장 기본적인 코드 레벨 방어부터 운영 환경에서의 고급 방어까지 차례로 살펴보자.

### Prepared Statement 사용

가장 중요한 방어 방식은 Prepared Statement를 사용하는 것이다.

Prepared Statement는 SQL 구조와 사용자 입력값을 분리한다. 따라서 사용자 입력값이 SQL 문법으로 해석되지 않고, 단순한 값으로만 처리된다.

```sql
SELECT * FROM users
WHERE username = ?
AND password = ?;
```

이 방식에서는 사용자가 `' OR '1' = '1` 같은 값을 입력해도 SQL 조건식이 아니라 문자열 값으로 처리된다.

### ORM 또는 Query Builder 사용

JPA, MyBatis, QueryDSL 같은 도구를 사용하면 SQL 문자열을 직접 조립하는 일을 줄일 수 있다.
다만 ORM을 사용하더라도 문자열을 직접 이어 붙여 동적 쿼리를 만들면 SQL Injection이 발생할 수 있다.

### 입력값 검증

입력값 검증은 애플리케이션이 기대하는 형식의 값만 받도록 제한하는 방식이다.

예를 들어 다음과 같은 규칙을 적용할 수 있다.

- 숫자 ID는 숫자만 허용한다.
- 이메일은 이메일 형식만 허용한다.
- 비밀번호는 일정 길이 이상, 특수문자 포함 등의 조건을 둔다.
- 페이지 크기, 날짜, 상태값 등은 기대하는 형식만 허용한다.

### Sanitization

Sanitization은 입력값에서 위험한 문자나 불필요한 값을 제거하거나 정리하는 방식이다.

예를 들어 SQL에서 문제가 될 수 있는 문자를 제거하거나 이스케이프할 수 있다.

- `'` : 문자열 구분 문자
- `;` : SQL 문 구분 문자
- `--` : 한 줄 주석
- `/* */` : 블록 주석

하지만 Sanitization만으로 SQL Injection을 완전히 막기는 어렵다.
공격 패턴은 다양하고, 모든 위험한 입력 조합을 미리 예측하기 어렵기 때문이다.

또한 너무 엄격하게 제거하면 정상 사용자의 입력까지 손상될 수 있다.

따라서 Sanitization은 Prepared Statement를 대체하는 방식이 아니라, 입력값 검증과 함께 사용하는 보조 방어선으로 봐야 한다.

### 에러 메시지 숨기기

SQL 오류 메시지를 사용자에게 그대로 보여주면 데이터베이스 구조가 노출될 수 있다.

사용자에게는 일반적인 에러 메시지만 보여주고, 자세한 SQL 오류는 서버 로그에만 남겨야 한다.

```text
사용자 응답: 요청을 처리하는 중 문제가 발생했습니다.
서버 로그: SQL syntax error ...
```

### SQL 명령 범위 제한과 최소 권한 원칙

애플리케이션이 사용하는 DB 계정에는 필요한 권한만 부여해야 한다.

- 일반 서비스 계정에 `DROP`, `ALTER` 같은 권한을 주지 않는다.
- 관리자용 계정과 서비스용 계정을 분리한다.

SQL Injection이 발생하더라도 DB 계정 권한이 제한되어 있으면 피해 범위를 줄일 수 있다.

예를 들어 게시글 조회 기능에서 사용하는 계정에 `DELETE`, `DROP` 권한이 없다면, 공격자가 SQL Injection으로 삭제 쿼리를 삽입하더라도 실제 삭제까지 이어지지 않을 수 있다.

### WAF 사용

WAF(Web Application Firewall)는 애플리케이션 앞단에서 요청을 검사하고, 알려진 SQL Injection 패턴을 차단하는 보안 장비 또는 서비스이다.

WAF는 입력값을 악성 SQL 패턴 목록과 비교해 의심스러운 요청을 막을 수 있다.

```text
' OR '1' = '1
UNION SELECT
DROP TABLE
```

이런 패턴 목록은 새로운 공격 방식에 대응하기 위해 계속 업데이트된다.

다만 WAF도 완벽한 해결책은 아니다.
WAF는 애플리케이션 코드의 취약점을 고치는 방법이 아니라, 공격 요청을 줄여주는 보조 방어선이다.


## REFERENCE
- https://www.fortinet.com/resources/cyberglossary/sql-injection
- https://learn.microsoft.com/ko-kr/sql/relational-databases/security/sql-injection?view=sql-server-ver17
