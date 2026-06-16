# JOIN 처리 방식과 최적화

`JOIN`은 여러 테이블을 연결해서 데이터를 조회하는 방식이다.

`JOIN`에 참여하는 테이블은 실행 순서에 따라 `드라이빙 테이블`과 `드리븐 테이블`로 나눌 수 있다.
- `드라이빙 테이블`: `JOIN`에서 먼저 읽는 테이블
- `드리븐 테이블`: 드라이빙 테이블에서 읽은 결과를 기준으로 반복해서 읽는 테이블

```text
employees → dept_emp
```

위 순서로 `JOIN`이 처리된다면 `employees`가 드라이빙 테이블, `dept_emp`가 드리븐 테이블이다.

드라이빙 테이블은 한 번 읽고 조건에 맞는 row를 가져오면 된다.
하지만 드리븐 테이블은 드라이빙 테이블에서 읽은 row 수만큼 반복해서 조회될 수 있다.

그래서 `JOIN`에서는 드리븐 테이블을 어떻게 읽는지가 중요하다.

## 인덱스와 JOIN 처리 순서

`조인 컬럼`의 `인덱스` 유무에 따라 처리 방식이 달라질 수 있다.

### 1. 두 테이블 모두 조인 컬럼에 인덱스가 있는 경우

어느 테이블을 먼저 읽어도 인덱스로 찾을 수 있다. 이런 경우에는 MySQL `옵티마이저`가 **통계 정보**를 바탕으로 더 유리한 순서를 선택한다.

통계 정보
- 각 테이블의 row 수
- 인덱스를 사용할 때 읽을 예상 row 수
- 조건의 선택도
- 어떤 테이블을 먼저 읽을 때 비용이 낮은지

예를 들어 `employees.gender = 'M'` 조건이 전체의 절반 정도를 반환한다고 판단하면, 옵티마이저는 `employees`에서 시작할 때 약 15만 건을 읽는다고 추정할 수 있다.

### 2. 한쪽에만 인덱스가 있는 경우

보통 드리븐 테이블의 조인 컬럼에 인덱스가 있는 것이 중요하다. 드리븐 테이블은 드라이빙 테이블에서 읽은 row 수만큼 반복 조회될 수 있기 때문이다.

드라이빙 테이블은 `JOIN`의 출발점으로 먼저 읽힌다.

그 다음 드라이빙 테이블에서 읽힌 각 row마다 조인 조건에 맞는 row를 드리븐 테이블에서 찾는다.

예를 들어 `employees`에서 3건을 읽었다고 하자.

```text
employees row 1 → dept_emp에서 emp_no로 검색
employees row 2 → dept_emp에서 emp_no로 검색
employees row 3 → dept_emp에서 emp_no로 검색
```

이때 `dept_emp.emp_no`에 인덱스가 없으면 매번 전체 스캔이 발생할 수 있다.

```text
employees row 1 → dept_emp 전체 스캔
employees row 2 → dept_emp 전체 스캔
employees row 3 → dept_emp 전체 스캔
```

반대로 `dept_emp.emp_no`에 인덱스가 있으면 매번 인덱스로 검색할 수 있다.

```text
employees row 1 → dept_emp.emp_no 인덱스로 검색
employees row 2 → dept_emp.emp_no 인덱스로 검색
employees row 3 → dept_emp.emp_no 인덱스로 검색
```

### 3. 두 테이블 모두 인덱스가 없는 경우

어느 쪽을 드리븐 테이블로 선택해도 부담이 크다. 이 경우 MySQL은 상황에 따라 `조인 버퍼`나 `해시 조인`을 사용할 수 있다.

## OUTER JOIN의 성능과 주의사항

`OUTER JOIN`은 한쪽 테이블의 결과를 보존해야 한다. 이 특성 때문에 옵티마이저가 `JOIN` 순서를 자유롭게 바꾸기 어렵다.

`LEFT OUTER JOIN`에서는 왼쪽 테이블의 결과를 보존해야 하므로, 일반적으로 오른쪽 테이블이 드라이빙 테이블이 되기 어렵다.

다음 쿼리를 보자.

```sql
SELECT *
FROM employees e
LEFT JOIN dept_emp de ON de.emp_no = e.emp_no
LEFT JOIN departments d ON d.dept_no = de.dept_no AND d.dept_name = 'Development';
```

만약 모든 직원이 반드시 부서 정보를 가지고 있고, 부서가 없는 직원을 결과에 포함할 필요가 없다면 `LEFT JOIN`이 필요하지 않을 수 있다.

이 경우 `INNER JOIN`을 사용하면 옵티마이저가 더 자유롭게 `JOIN` 순서를 선택할 수 있다.

```sql
SELECT *
FROM employees e
JOIN dept_emp de ON de.emp_no = e.emp_no
JOIN departments d ON d.dept_no = de.dept_no
WHERE d.dept_name = 'Development';
```

따라서 `OUTER JOIN`은 꼭 필요할 때만 사용해야 한다.
불필요한 `OUTER JOIN`은 옵티마이저의 `JOIN` 순서 선택을 제한할 수 있다.

## 최적화: 지연된 조인

지연된 조인은 `JOIN`을 나중으로 미루는 최적화 패턴이다. 먼저 결과를 줄이고, 마지막에 필요한 테이블과 `JOIN`한다.

특히 `ORDER BY`, `GROUP BY`, `LIMIT`이 있는 쿼리에서 효과가 있을 수 있다.

다음과 같은 쿼리가 있다고 하자.

```sql
SELECT e.*
FROM salaries s
JOIN employees e ON e.emp_no = s.emp_no
WHERE s.emp_no BETWEEN 10001 AND 13000
GROUP BY s.emp_no
ORDER BY SUM(s.salary) DESC
LIMIT 10;
```

일반적인 처리 흐름은 다음과 같을 수 있다.

```text
salaries와 employees를 먼저 조인
→ GROUP BY
→ ORDER BY
→ LIMIT 10
```

문제는 `JOIN`을 먼저 하면 중간 결과가 커질 수 있다는 점이다.

지연된 조인은 먼저 필요한 결과를 줄이고, 마지막에 `JOIN`한다.

```sql
SELECT e.*
FROM (
    SELECT s.emp_no
    FROM salaries s
    WHERE s.emp_no BETWEEN 10001 AND 13000
    GROUP BY s.emp_no
    ORDER BY SUM(s.salary) DESC
    LIMIT 10
) x
JOIN employees e ON e.emp_no = x.emp_no;
```

처리 흐름은 다음과 같다.

```text
salaries에서 GROUP BY, ORDER BY, LIMIT 10 처리
→ 최종 10건의 emp_no만 남김
→ 10건만 employees와 조인
```

즉, 일반 조인은 많은 row를 먼저 조인한 뒤 줄이고, 지연된 조인은 먼저 줄인 뒤 적은 row만 조인한다.

### 주의점
다만 지연된 조인은 아무 쿼리에나 적용하면 안 된다.
`LIMIT`을 먼저 적용해도 최종 결과가 동일하다는 보장이 있어야 한다.

잘 쓰면 빠르지만, 잘못 쓰면 결과가 달라질 수 있다.

## 정렬 시 주의점

정렬이 필요한 결과라면 반드시 `ORDER BY`를 명시해야 한다.

예전에는 `Nested Loop Join`의 특성 때문에 드라이빙 테이블을 읽은 순서가 결과에도 유지되는 것처럼 보이는 경우가 있었다.

하지만 MySQL 8.0부터는 `Hash Join` 같은 다른 `JOIN` 방식이 사용될 수 있고, 이런 경우 결과 순서를 보장할 수 없다.
