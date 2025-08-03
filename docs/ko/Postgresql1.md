# SELECT 쿼리 실행 순서

## 개요

PostgreSQL의 SQL 실행 순서와 옵티마이저의 동작 방식, 그리고 성능에 영향을 줄 수 있는 주요 요소에 대해 정리한다.

## 실행 순서

- SQL의 논리적 실행 순서는 아래와 같다.
- PostgreSQL은 이 순서를 기반으로 옵티마이저가 실행 계획을 구성한다.

```
1. FROM  
2. JOIN  
3. WHERE  
4. GROUP BY  
5. HAVING  
6. SELECT  
7. ORDER BY  
8. LIMIT
```

## 옵티마이저 동작

- PostgreSQL은 통계 기반 옵티마이저를 사용하여 최적의 실행 계획을 수립한다.

- 통계 정보에 따라 JOIN 순서, 인덱스 사용 여부 등을 결정한다.

- 통계가 부정확하거나 데이터 분포가 왜곡된 경우, JOIN 이후 WHERE 필터링이 발생할 수 있어 성능이 저하된다.

## 옵티마이저 오판 예시

PostgreSQL 옵티마이저는 통계 기반으로 실행 계획을 세우기 때문에,

- 통계가 부정확하거나

- 통계는 맞지만 데이터 분포나 조건이 특이하거나

- 인덱스가 없어 선택지가 제한되면
비효율적인 실행 경로를 선택해서 실제로 "JOIN → WHERE" 순서로 처리할 수도 있다.

```
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.status = 'CANCELLED';
```

옵티마이저가 'CANCELLED'가 자주 있는 값이라고 잘못 판단하거나, 통계를 오래 갱신하지 않아 status 컬럼 분포를 잘 모르면...

옵티마이저:“어차피 거의 다 CANCELLED겠지? 그럼 JOIN 먼저 해도 괜찮겠군.”

실제로는 WHERE 조건이 대부분의 데이터를 걸러낼 수 있음에도, JOIN을 먼저 수행하고 나서 필터링하는 비효율적인 경로가 선택될 수 있다.

## 주의사항

- JOIN이 필요 없는 경우에는 명시적으로 수행하지 않도록 쿼리를 구성해야 한다.

- 필터링 가능한 조건은 WHERE로 먼저 적용하는 것이 효율적이다.

- 불필요한 JOIN은 전체 성능에 악영향을 준다.

## 대책

- ANALYZE를 통해 통계를 최신 상태로 유지한다.

- 조건 컬럼에는 인덱스를 적극 활용한다.

- EXPLAIN ANALYZE를 사용해 실제 실행 계획을 확인한다.

- 필요시 JOIN 순서를 유도하기 위해 서브쿼리나 CTE를 활용할 수 있다.

## 요약

| 항목 | 내용 |
| --- | --- |
| 실행 순서 | FROM → JOIN → WHERE |
| 주의점 | 불필요한 JOIN은 하지 말 것 |
| 위험요소 | 통계 오류로 인해 비효율적 실행 가능 |
| 해결방법 | 통계 최신화, 조건 필터링 우선, 실행 계획 확인 |

--- 

# WHERE절 최적화

## 개요

WHERE 절은 실행 순서상 FROM 이후에 실행되므로, 불필요한 데이터는 최대한 빨리 제거하는 것이 핵심이다.

인덱스를 효율적으로 활용하기 위한 WHERE 절 작성 방식이 성능에 큰 영향을 미친다.

## 인덱스를 타는 조건

- 컬럼 = 값

- 컬럼 BETWEEN 값1 AND 값2

- 컬럼 IN (값1, 값2, ...)

- 컬럼 > 값, 컬럼 < 값 등 범위 조건

## 인덱스를 타지 않는 조건 (주의)

| 유형 | 비효율적인 예시 | 인덱스 활용 가능한 리팩토링 예시 | 설명 |
| -- | -- | -- | -- |
| 컬럼에 함수 사용 | LOWER(name) = 'kim' | name ILIKE 'kim' 또는 name = 'Kim' (대소문자 정확히 아는 경우) | 컬럼에 함수가 적용되면 인덱스를 사용할 수 없습니다. |
| 컬럼 연산 | created_at + INTERVAL '1 day' = NOW() | created_at >= NOW() - INTERVAL '1 day'	 | 컬럼이 연산에 포함되면 인덱스가 무시되므로, 컬럼 자체를 기준으로 조건을 작성합니다. |
| 데이터 타입 캐스팅 | created_at::DATE = '2024-01-01'	| created_at >= '2024-01-01' AND created_at < '2024-01-02' | 	타입 캐스팅도 인덱스 사용을 방해하므로, 범위 조건으로 변경합니다. |

## 예제

```
-- 인덱스 사용 안 됨
SELECT * FROM users WHERE created_at::DATE = '2024-01-01';

-- 인덱스 사용됨
SELECT * FROM users 
WHERE created_at >= '2024-01-01' AND created_at < '2024-01-02';
```

---

# JOIN 최적화

## 개요

- JOIN은 여러 테이블의 데이터를 조합할 수 있도록 해주는 중요한 기능이지만 잘못 사용하면 성능에 큰 영향을 미친다.

- PostgreSQL 기준으로 통계 기반 옵티마이저가 실행 계획을 결정한다.

- 하지만 항상 최선의 선택을 하는 것은 아니다. 따라서 개발자는 JOIN의 동작 방식과 성능 특성을 이해하고 있어야 한다.

참고: SQL에서 JOIN은 INNER JOIN과 동일한 의미로 처리된다. 하지만 가독성과 명확성을 위해 INNER JOIN을 명시적으로 사용한다.

## JOIN 종류와 성능 특성

| JOIN 종류 | 설명 | 성능 영향 |
| -- | -- | -- |
| INNER JOIN | 조건에 일치하는 데이터만 조인 | 일반적으로 가장 빠름 |
| LEFT OUTER JOIN | 왼쪽 테이블은 무조건 유지 + 오른쪽 조인 | 	불필요한 row까지 유지됨 → 성능 저하 가능 |
| RIGHT OUTER JOIN | 오른쪽 테이블 유지 | 자주 사용되지 않음 |
| FULL OUTER JOIN | 양쪽 테이블 모두 유지 |	가장 비용이 큼 |
| CROSS JOIN | 조건 없이 전체 조합 | 카디널리티 폭발 위험 → 주의 필요 |

## 조인 순서에 따른 성능 전략

- 여러 개의 JOIN이 사용될 경우, 그 순서에 따라 성능이 달라질 수 있다.

- PostgreSQL은 자동으로 최적의 순서를 찾으려 하지만 다음 원칙들을 고려하면 더 나은 성능을 유도할 수 있다

- 데이터가 적은 테이블부터 조인하는 것이 유리하다.

- 옵티마이저가 비효율적인 조인 순서를 선택하는 경우, 서브쿼리나 CTE로 명시적으로 순서를 유도할 수 있다.

예시: 3개 테이블 조인 시 순서에 따른 성능 차이
```
-- 비효율적인 조인 순서
SELECT *
FROM huge_table_a a
INNER JOIN medium_table_b b ON a.id = b.a_id
INNER JOIN small_table_c c ON b.id = c.b_id;

-- 효율적인 조인 순서 유도 (작은 테이블부터 시작)
SELECT *
FROM small_table_c c
INNER JOIN medium_table_b b ON b.id = c.b_id
INNER JOIN huge_table_a a ON a.id = b.a_id;
```

첫 번째 쿼리는 중간 결과가 커질 수 있는 구조이며, 두 번째 쿼리는 작은 테이블부터 조인하여 중간 결과 크기를 줄이므로 성능상 유리하다.

## 정리

- JOIN의 성능은 조인 조건, 조인 순서, 데이터 크기, 통계 정확도에 따라 결정됨

- PostgreSQL 옵티마이저는 통계 기반이므로 항상 최적은 아님

- 필요시 서브쿼리, CTE, 인덱스 등을 활용하여 실행 경로 유도 가능

- 작은 테이블부터 조인, 조인 순서에 따른 중간 결과 크기 감소는 중요한 최적화 전략임

---

# SQL 쿼리 안티패턴

## SQL 쿼리 안티 패턴 모음

SQL을 작성할 때 자주 발생하는 쿼리 안티패턴들을 정리하고, 각각에 대해 왜 문제가 되는지, 어떻게 개선할 수 있는지 예제를 통해 설명합니다.

## SELECT * 사용

- 문제점: 불필요한 모든 컬럼을 조회하게 되어 성능 저하 발생

- 개선: 필요한 컬럼만 명시적으로 지정

```
-- Bad
SELECT * FROM users;

-- Good
SELECT id, name, email FROM users;
```

## 불필요한 DISTINCT 사용

- 문제점: DISTINCT는 내부적으로 정렬(SORT) 또는 해시(HASH) 연산이 필요하여 대용량 데이터에서 성능을 크게 떨어뜨릴 수 있습니다.

- 개선: 중복이 발생하는 이유를 먼저 분석

- DISTINCT 는 정말 필요한 경우에만 사용

- DISTINCT를 자주 사용한다면 쿼리 로직 재검토 및 데이터 모델 개선 필요

```
-- Bad
SELECT DISTINCT customer_id
FROM orders
WHERE order_date > '2023-01-01';

-- Good
-- 실제 중복이 없어야 한다면, GROUP BY로 의도 명확화
SELECT customer_id
FROM orders
WHERE order_date > '2023-01-01'
GROUP BY customer_id;
```

## LIKE '%...%' 패턴 검색

- 문제점: 앞에 와일드카드를 사용하는 경우 인덱스 사용 불가

- 개선: 접두어 검색 또는 GIN, trigram 인덱스 사용

- gin 인덱스도 3글자부터 속도 빠름

```
-- Bad
SELECT * FROM articles WHERE title LIKE '%postgresql%';

-- Good
-- 인덱스를 활용하려면 접두어 방식 또는 특수 인덱스
SELECT * FROM articles WHERE title LIKE 'postgresql%';
```

## 컬럼에 함수 또는 연산 적용

- 문제점: 인덱스를 사용할 수 없어 풀스캔 발생

- 개선: 함수나 연산은 컬럼이 아닌 값에 적용

```
-- Bad
SELECT * FROM users WHERE LOWER(name) = 'kim';

-- Good
SELECT * FROM users WHERE name ILIKE 'kim';
```

```
-- Bad
SELECT * FROM logs WHERE created_at + INTERVAL '1 day' = NOW();

-- Good
SELECT * FROM logs WHERE created_at >= NOW() - INTERVAL '1 day';
```

## 암시적 타입 변환

- 문제점: 정수와 문자열 비교 등에서 인덱스가 무시될 수 있음

- 개선: 명시적으로 타입을 일치시킴

```
-- Bad
SELECT * FROM users WHERE id = '123';

-- Good
SELECT * FROM users WHERE id = 123;
```

## JOIN 남용

- 문제점: 필요 없는 테이블까지 조인해서 불필요한 연산 증가

- 개선: JOIN 대상은 최소화하고, EXISTS 등으로 대체 고려

```
-- Bad
SELECT * FROM users u JOIN orders o ON u.id = o.user_id JOIN products p ON o.product_id = p.id;

-- Good
-- 필요 없는 JOIN 제거
SELECT u.* FROM users u WHERE EXISTS (
  SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

---

# 파티션 테이블 사용 시 주의사항

## 파티션 테이블 사용 시 주의사항

PostgreSQL에서 파티션 테이블에 대한 쿼리 작성 시 반드시 파티션 키를 활용한 조건절을 포함해야 원하는 성능을 얻을 수 있다. 그렇지 않으면 성능 저하뿐 아니라 심각한 시스템 리소스 낭비 및 락 경합(lock contention) 문제로까지 이어질 수 있다.

## 파티션 키를 활용한 쿼리 작성

파티션 테이블에서는 파티션 키를 WHERE 절에서 직접 명시해야 해당 파티션만 조회할 수 있다 (partition pruning).

예시
```
-- ❌ 파티션 pruning이 되지 않음 (함수 사용)
SELECT * FROM logs WHERE DATE(created_at) = CURRENT_DATE;

-- ✅ 파티션 pruning 가능 (범위 조건 사용)
SELECT * FROM logs 
WHERE created_at >= CURRENT_DATE 
  AND created_at < CURRENT_DATE + INTERVAL '1 day';
```

## 올바른 파티션 키 활용법 요약

| 작성 방식 | 예시 | 결과 |
| -- | -- | -- |
| 함수 사용 ❌ | DATE(created_at) = '2024-01-01' | 파티션 키 사용 실패 → pruning 실패 |
| 연산 사용 ❌ | created_at + INTERVAL '1 day' = CURRENT_DATE | pruning 실패 |
| 타입 캐스팅 ❌ | created_at::DATE = '2024-01-01' | pruning 실패 |
| 범위 조건 ✅ | created_at >= '2024-01-01' AND created_at < '2024-01-02'	 | pruning 성공 |
| OR 조건 ❌ | created_at = '2024-01-01' OR user_id = 1 | 일부 조건만 파티션 키여도 pruning이 되지 않음 |
| 서브쿼리 사용 ❌ | created_at IN (SELECT date FROM ref_dates) | pruning 불확실 |
| CASE WHEN 사용 ❌ | CASE WHEN flag THEN created_at ELSE NULL END = '2024-01-01'	 | pruning 어려움 |
| JOIN 조건만 사용 ❌ | JOIN ... ON logs.created_at = dim.date | pruning 불가, WHERE절에 파티션 키 명시 필요 |

## 파티션 키를 타지 못했을 때 발생하는 문제점

| 문제 | 설명 |
| -- | -- |
| 모든 파티션 풀스캔 | 조건이 파티션 키를 활용하지 못하면 PostgreSQL은 모든 파티션을 순차적으로 검사함 |
| 실행 속도 저하 | 수십~수백 개의 파티션을 다 뒤지기 때문에 쿼리 시간이 급격히 증가 |
| 비효율적인 인덱스 사용 | 인덱스가 있어도 파티션 pruning이 안되면 인덱스 탐색의 효과가 제한됨 |
| 불필요한 I/O 증가 | 관련 없는 파티션까지 디스크에서 읽으면서 I/O 부하 증가 |
| 플래너의 잘못된 판단 | 파티션 수가 많을수록 플래너가 잘못된 실행 계획을 선택할 가능성 증가 |
| 메모리 사용 증가 | 더 많은 데이터를 처리하려다 work_mem, temp buffer 사용량 증가 |
| 락 경합(Lock Contention) | 단일 파티션만 조회하면 해당 파티션에만 락을 걸지만, pruning이 안 되면 모든 파티션에 락이 분산되어 걸리며 → 락 대기나 데드락 발생 가능성 증가 |

## 정리

- 파티션 테이블을 사용할 땐 항상 파티션 키 조건을 명시적으로 작성해야 함

- 함수, 연산, 타입 변환 등을 파티션 키에 사용하면 pruning이 불가능해짐

- pruning이 안 되면 성능 저하뿐만 아니라 락 이슈로까지 이어질 수 있음

```
-- 추천 방식
WHERE partition_key_column >= '2024-01-01' AND partition_key_column < '2024-01-02'
```

PostgreSQL에서 파티션 성능을 극대화하려면 단순히 파티션을 나누는 것만이 아니라 쿼리 설계 방식 자체가 중요하다.

---

# 쿼리 계획 분석 가이드

## PostgreSQL 쿼리 계획 분석 가이드

PostgreSQL 쿼리 계획을 이해하고 성능 최적화를 위한 기본적인 분석 방법에 대해 작성한다.

## 쿼리 계획이란?

쿼리 계획(Query Plan)은 데이터베이스가 쿼리를 실행하기 위한 전략을 나타냅니다. 이 계획은 PostgreSQL이 쿼리를 어떻게 처리할 것인지에 대한 정보를 제공합니다. 실행 계획을 분석하면 쿼리 성능을 개선하는 데 중요한 힌트를 얻을 수 있습니다.

쿼리 계획을 분석하려면 EXPLAIN 명령어를 사용하여 쿼리 실행 계획을 확인할 수 있습니다. 

예시:
```
EXPLAIN SELECT * FROM employees WHERE salary > 50000;
```

이 명령을 실행하면 PostgreSQL이 해당 쿼리를 어떻게 처리할 것인지에 대한 실행 계획을 출력합니다.

## 쿼리 계획 항목 이해하기

### 주요항목
쿼리 계획을 분석할 때 중요한 항목들은 다음과 같습니다

| 항목 | 설명 |
| -- | -- |
| Cost (비용) | 쿼리 실행 계획에서 각 노드가 예상하는 비용입니다. 비용이 높은 부분은 성능 저하를 의미합니다. 비용이 높은 노드를 최적화해야 합니다. |
| Actual Time (실제 실행 시간) | 각 노드에서 실제로 수행한 실행 시간입니다. 예상 시간과 실제 실행 시간이 차이 나는 부분은 성능 최적화의 힌트를 제공합니다. |
| Scan Type (스캔 방식) | 데이터를 검색하는 방법입니다 (Seq Scan, Index Scan, Bitmap Index Scan 등). 잘못된 스캔 방식은 성능 저하의 주요 원인입니다. |
| Rows (행 수) | 각 노드에서 예상되는 행 수입니다. 많은 행이 반환되는 노드는 성능에 큰 영향을 미칩니다. |
| Join Type (조인 방식) | 두 테이블 간 결합 방식입니다 (Nested Loop, Hash Join, Merge Join). 잘못된 조인 방식은 성능을 크게 저하시킬 수 있습니다. |
| Filter (필터 조건) | 필터 조건이 효율적으로 적용되고 있는지 확인합니다. 많은 행이 필터링되는 경우 성능이 떨어질 수 있습니다. |

## 주요 스캔 방식

PostgreSQL에서는 여러 가지 스캔 방식을 사용하여 데이터를 검색합니다. 각 방식의 특징과 성능을 이해하면 쿼리 성능을 최적화할 수 있습니다.

| 스캔 유형 | 설명 | 성능 | 유리한 상황 |
| -- | -- | -- | -- |
| Index Only Scan (인덱스 전용 스캔) | 인덱스에서 필요한 데이터를 모두 가져오는 방식입니다. | 매우 빠름 | 디스크 접근 없이 인덱스에서 직접 데이터를 가져오므로 성능이 매우 빠릅니다. 인덱스에 모든 컬럼이 포함되어야 합니다. |
| Index Scan (인덱스 스캔) | 인덱스를 사용하여 데이터를 검색하는 방식입니다. | 빠름 | 인덱스를 사용할 수 있으면 성능이 크게 향상됩니다. 인덱스가 없거나 잘못된 인덱스를 사용할 경우 성능 저하가 발생할 수 있습니다. |
| Bitmap Index Scan (비트맵 인덱스 스캔) | 여러 인덱스를 결합하여 데이터를 검색하는 방식입니다. | 빠름 | 여러 조건을 결합할 때 성능을 향상시킬 수 있습니다. 하지만 메모리 사용량이 많을 수 있고, 작은 데이터셋에서는 비효율적일 수 있습니다. |
| Seq Scan (순차 스캔) | 테이블의 모든 행을 순차적으로 읽는 방식입니다.	 | 느림 (테이블 크기가 클수록 성능 저하) | 인덱스를 사용할 수 없거나, 전체 테이블을 읽어야 할 경우 사용됩니다. 데이터가 적을 때는 괜찮지만, 큰 테이블에서는 성능 저하가 심각할 수 있습니다. |

### 성능 최적화 팁

1. 인덱스 사용: Seq Scan을 피하고 Index Scan을 사용할 수 있도록 쿼리와 테이블을 최적화하세요. 특히 자주 조회되는 컬럼에는 인덱스를 생성하는 것이 중요합니다.

2. 인덱스 전용 스캔 활용: 가능한 경우, Index Only Scan을 활용하여 디스크 접근 없이 빠르게 데이터를 가져오세요.

3. 비트맵 인덱스 스캔: 여러 조건을 결합할 때 성능을 높일 수 있으니, 조건이 많은 쿼리에서 비트맵 인덱스를 고려해보세요.

## 주요 조인 방식

PostgreSQL에서 조인을 실행할 때 여러 가지 방식이 있습니다. 각 방식의 특징을 이해하면 쿼리 성능을 크게 향상시킬 수 있습니다.

| 조인 유형 | 설명 | 성능 | 유리한 상황 |
| -- | -- | -- | -- |
| Nested Loop Join (중첩 루프 조인) | 두 테이블을 하나씩 순차적으로 탐색하며 결합하는 방식입니다. | 느림 (큰 테이블에서는 비효율적) | 작은 테이블 간의 조인이나, 작은 범위에서만 데이터를 찾는 경우. |
| Hash Join (해시 조인) | 해시 테이블을 만들어 두 테이블을 결합하는 방식입니다.	 | 빠름 (대규모 데이터셋에서 유리) | 큰 테이블에서 조인할 때, 두 테이블이 모두 충분히 커야 효과적. |
| Merge Join (병합 조인) | 두 테이블을 정렬된 상태에서 병합하는 방식입니다. | 빠름 (정렬된 테이블에 유리) | 두 테이블이 정렬되어 있을 때 매우 효율적입니다. |
| Index Nested Loop Join (인덱스 중첩 루프 조인) | 중첩 루프 조인에 인덱스를 사용하는 방식입니다. | 빠름 (인덱스를 잘 활용하면 빠름) | 작은 테이블에서 큰 테이블을 조인할 때 효율적입니다. 인덱스를 잘 활용할 수 있는 경우에 유리합니다. |

### 성능 최적화 팁

1. 조인 방식 선택: 데이터셋 크기에 따라 적절한 조인 방식을 선택하세요. 큰 테이블 간의 조인에는 Hash Join이나 Merge Join이 더 효율적일 수 있습니다.

2. 인덱스 활용: Index Nested Loop Join을 활용하여 작은 테이블에서 큰 테이블을 조인할 때 성능을 크게 향상시킬 수 있습니다.

3. 조인 순서: 가능한 한 작은 테이블부터 조인하는 것이 좋습니다. 작은 테이블을 먼저 처리하면 성능을 최적화할 수 있습니다.

## 쿼리 계획 예시

예시 1: 기본 Seq Scan

```
EXPLAIN SELECT * FROM employees WHERE salary > 50000;
```

쿼리 계획:

```
Seq Scan on employees  (cost=0.00..431.00 rows=1000 width=48)
  Filter: salary > 50000
```

이 경우 Seq Scan이 사용되고 있습니다. 테이블의 모든 행을 읽고 필터를 적용하는 방식이므로, 성능이 비효율적일 수 있습니다. salary 컬럼에 인덱스를 추가하면 성능이 향상될 수 있습니다.

예시 2: Index Scan

```
EXPLAIN SELECT * FROM employees WHERE salary > 50000 AND department_id = 10;
```

쿼리 계획:

```
Index Scan using employees_salary_idx on employees  (cost=0.29..12.12 rows=200 width=48)
  Index Cond: (salary > 50000)
  Filter: (department_id = 10)
```

Index Scan이 사용되고 있습니다. 인덱스를 활용하여 salary 컬럼을 빠르게 검색하고 있습니다. 추가 조건(department_id = 10)에 대한 필터는 이후에 적용됩니다.

## 결론

쿼리 계획을 분석하면 성능을 최적화하는 데 중요한 정보를 얻을 수 있습니다. 신입 개발자는 쿼리 계획에서 비용, 실제 실행 시간, 스캔 방식, 조인 방식 등을 집중적으로 확인하고, 성능이 비효율적인 부분을 최적화하는 것이 중요합니다. 이를 통해 데이터베이스의 성능을 향상시킬 수 있습니다.

쿼리 성능을 최적화하려면:

- 적절한 인덱스 활용: 인덱스를 활용하여 Seq Scan을 피하세요.

- 효율적인 조인 방식 선택: 데이터셋 크기에 맞는 조인 방식을 선택하세요.

---

# 인덱스 설계 가이드

## PostgreSQL 인덱스 설계 가이드

인덱스는 쿼리 성능을 크게 향상시키지만, 잘못 설계하면 저장 공간 낭비와 유지비용 증가로 이어질 수 있다. PostgreSQL의 주요 인덱스 종류, 사용 사례, 제한사항, 멀티컬럼 인덱스 전략을 정리한다.

## 인덱스 종류와 사용 사례

PostgreSQL에서 제공하는 주요 인덱스 종류와 각각의 특성, 사용 사례, 제한사항은 다음과 같다.

| 인덱스 종류 | 설명 | 사용 사례 | 장점 | 제한사항 |
| -- | -- | -- | -- | -- |
| B-Tree | 균형 트리 구조, 기본 인덱스 | 등치(=), 범위(>, <), IN, ORDER BY, LIKE 'prefix%' | 범용적, 대부분의 쿼리에 적합	| LIKE '%text%' 같은 비접두어 검색, 복잡한 데이터 구조(배열, JSONB)에 부적합 |
| GIN | Generalized Inverted Index, 복합 데이터용 | JSONB, 배열 포함(@>),全文 검색(tsvector, tsquery), trigram 검색 | 복잡한 데이터 검색에 효율적 | 2글자 이하 검색 시 성능 저하, 생성/유지 비용 높음, 업데이트 잦은 테이블에 부적합 |
| GiST | Generalized Search Tree, 유연한 검색 | 공간 데이터(PostGIS), 유사성 검색(LIKE, trigram), 범위 겹침 | 복잡한 패턴/공간 검색 지원 | 복잡한 쿼리에서 느릴 수 있음, B-Tree보다 인덱스 크기 큼 |
| BRIN | Block Range Index, 순차 데이터용 | 시계열 데이터, 순차 증가 ID | 저장 공간 적음, 대용량 테이블 효율적 | 무작위 데이터, 잦은 업데이트 환경에 부적합 |
| Hash | 해시 기반, 등치 연산 전용 | 정확한 등치 검색(=) | 빠른 등치 검색 | 범위 검색 불가, PostgreSQL 10+에서 안정적, 업데이트 성능 저하 가능 |

## 멀티컬럼 인덱스 전략

멀티컬럼 인덱스는 여러 컬럼을 조합해 WHERE, JOIN, ORDER BY에서 사용되는 경우 효율적이다.

### 설계 원칙

- 컬럼 순서:

    - 필터링 우선: 자주 사용되는 WHERE 조건 컬럼 먼저 배치 (예: WHERE col1 = X AND col2 > Y → (col1, col2)).

    - 카디널리티 고려: 고유 값이 많은 컬럼(고카디널리티)을 앞에 배치.

    - ORDER BY 고려: 정렬 컬럼은 마지막에 배치.

- 제한사항:

    - 너무 많은 컬럼 포함 시 유지비용 증가, 인덱스 크기 커짐.

    - 앞쪽 컬럼이 조건에 포함되지 않으면 인덱스 사용 불가.

예시
```
CREATE INDEX idx_orders_date_status ON orders (order_date, status);
-- 유용: SELECT * FROM orders WHERE order_date = '2024-01-01' AND status = 'CANCELLED';
-- 비효율적: SELECT * FROM orders WHERE status = 'CANCELLED'; (order_date 조건 없음)
```

### 실전 팁

쿼리 분석: EXPLAIN으로 인덱스 사용 여부 확인.

부분 인덱스: 특정 조건에만 적용되는 인덱스 생성.
```
CREATE INDEX idx_orders_cancelled ON orders (order_date) WHERE status = 'CANCELLED';
```

커버링 인덱스: 쿼리에서 필요한 모든 컬럼을 포함해 디스크 접근 최소화.
```
CREATE INDEX idx_users_covering ON users (id, name, email) INCLUDE (created_at);
```

### 인덱스 설계 주의사항

- 과도한 인덱스: 인덱스는 쓰기 성능(INSERT, UPDATE, DELETE) 저하. 테이블당 5~10개 이하 권장.

- 통계 유지: ANALYZE로 통계 최신화, 잘못된 통계는 비효율적 인덱스 선택 유발.

- 유지비용: GIN, GiST는 생성/유지 비용 높음, 잦은 업데이트 테이블에 주의.

- 쿼리 패턴 분석: pg_stat_statements로 실제 쿼리 패턴 파악 후 인덱스 설계.

- GIN 트라이그램 제한:

    - 2글자 이하 검색은 인덱스 효율 낮음. 예: LIKE '%db%'는 GIN 인덱스 비효율적.

    - 3글자 이상(예: LIKE '%data%')에서 GIN 트라이그램 인덱스 효과적.

    - 설정: CREATE EXTENSION pg_trgm;, CREATE INDEX idx_trgm ON table USING GIN (column gin_trgm_ops);.

- GiST 트라이그램 제한:

    - GIN보다 유사성 검색(%)에서 느림.

    - PostGIS 등 공간 데이터에 특화.

테스트: EXPLAIN ANALYZE로 인덱스 효과 확인, 실제 데이터로 테스트 필수.

## 요약

- B-Tree: 범용적, 등치/범위 검색에 최적.

- GIN: JSONB, 배열,全文 검색. 3글자 이상 검색에서 효율적, 2글자 이하 비효율.

- GiST: 공간 데이터, 유사성 검색. GIN보다 트라이그램 검색 느림.

- BRIN: 시계열/순차 데이터, 대용량 테이블에 적합.

- Hash: 등치 검색 전용, 제한적 사용.

멀티컬럼 인덱스: 필터링 순서, 카디널리티 고려, 커버링 인덱스 활용.

주의: 인덱스 유지비용, 통계 갱신, 쿼리 패턴 분석 필수.