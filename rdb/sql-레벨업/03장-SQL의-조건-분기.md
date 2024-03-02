# 3장 SQL의 조건 분기

## 8강 UNION을 사용한 쓸데없이 긴 표현
### 1. UNION을 사용한 조건 분기와 관련된 간단한 예제

SQL에서는 case식와 UNION을 활용하여 조건분기를 할 수 있다. </br>
SQL초보자들은 case문이 잘 와닿지 않기때문에 UNION을 사용하여 조건분기를 하는 경우가 많다. </br>
하지만 UNION으로 조건분기를 사용하면 select문을 여러번 실행하여 IO비용이 늘어나 성능이 저하되며 SQL 가독성이 떨어진다. </br>
따라서 UNION을 사용하면 case문으로 리펙토링을 진행해야하고 UNION은 필요한 경우에만 신중히 사용해야한다. </br>

8강에서는 잘못 사용된 UNION을 case 식으로 SQL 리펙토링하는 예제를 살펴보자

Item 테이블

| id | name | year | price_tax_ex | price_tax_in |
|----|------|------|--------------|--------------|
| 1  | A    | 2019 | 1000         | 1100         |
| 2  | A    | 2019 | 2000         | 2200         |
| 3  | B    | 2020 | 3000         | 3300         |
| 4  | B    | 2020 | 4000         | 4400         |

먼저 상품 테이블이 있고 상품 테이블은 각 년도에 `세금 포함 가격`과 `세금 미포함 가격`이 있다. </br>
이때 한국 법이 개정되어서 2020년 이후 부터는 각 웹사이트에는 세금이 포함되는 가격을 표시해야한다면 SQL문을 어떻게 만들어야할까? </br>

```SQL
SELECT id, name, year, price_tax_ex as price
FROM item
WHERE year < 2020
UNION ALL
SELECT id, name, year, price_tax_in as price
FROM item
WHERE year >= 2020;
-----------------------------------------------------------
Append  (cost=0.00..2.11 rows=200 width=50)
  ->  Seq Scan on item  (cost=0.00..1.12 rows=200 width=50)
        Filter: (year < 2020)
  ->  Seq Scan on item  (cost=0.00..1.12 rows=200 width=50)
        Filter: (year >= 2020)
```

이렇게 union을 통해서 item 테이블을 2020년 이전과 이후로 나누어서 조회할 수 있다. </br>
데이터를 베타적으로 조회하기때문에 UNION을 사용했지만 이 SQL은 2가지 문제점가 있다. </br>
1. SQL문이 길어져서 가독성이 떨어진다.
2. SELECT문을 2번 실행하기때문에 IO비용이 늘어나 성능이 저하된다.

</br>
이 UNION을 CASE식으로 리펙토링하면 다음과 같다.

```SQL
SELECT id, name, year,
  CASE
    WHEN year < 2020 THEN price_tax_ex
    WHEN year >= 2020 THEN price_tax_in 
  END AS price
FROM item;
-----------------------------------------------------------
Seq Scan on item  (cost=0.00..1.12 rows=200 width=50)
  Filter: (year < 2020 OR year >= 2020)
```
이렇게 case식으로 리펙토링을 진행하면 SQL 가독성이 높아지고 성능이 향상된다.

### 2. UNION을 사용하고 싶어질때는 CASE문을 먼저 고려하자

위 예제처럼 UNION을 사용하고 싶어질때는 case문을 먼저 고려해보자. </br>
절차지향에 익숙한 SQL초보자는 프로그래밍의 if문을 사용하고 싶을때 SELECT와 UNION을 사용하고 싶을것이다. </br>
그럴떄마다 **"SQL의 CASE로는 어떻게 해결할 수 있지?"** 라고 의도적으로 생각하는것이 중요하다. </br>

"SQL격언중에 WHERE, HAVING으로 조건분기하면 SQL초보자이고 SELECT문에서만 분기하면 SQL고수이다"라고 한다. </br>

## 9강 집계와 조건 분기
### 1. 집계와 조건 분기

| prefecture(지역이름) | sex(성별) | pop(인구) | 
|:-----------------|---------|---------|
| 성남             | 1       | 60      |
| 성남             | 2       | 40      |
| 수원             | 1       | 90      |
| 수원             | 2       | 100     |
| 용인             | 1       | 70      |
| 용인             | 2       | 60      |

Population(인구)테이블에 위와같이 데이터가 저장되어있다.
이때 각 지역별 성별 인구수를 조회하고 싶으면 어떻게 할 수 있을까?
```SQL
SELECT prefecture, pop as pop_men, null as pop_wom
FROM population
where SEX = 1
UNION ALL
SELECT prefecture, null as pop_men, pop as pop_wom
FROM population
where SEX = 2;
-----------------------------------------------------------
Sort  (cost=100.00..100.02 rows=6 width=40)
  Sort Key: prefecture
  ->  Append  (cost=0.00..99.88 rows=6 width=40)
        ->  Seq Scan on population  (cost=0.00..49.92 rows=3 width=40)
            Filter: sex = 1
        ->  Seq Scan on population  (cost=0.00..49.92 rows=3 width=40)
            Filter: sex = 2
-----------------------------------------------------------
prefecture	pop_men	pop_wom
성남	60	NULL
성남	NULL	40
수원	90	NULL
수원	NULL	100
용인	70	NULL
용인	NULL	60
```
이렇게 실행계획을 살펴보면 2번의 테이블 풀스캔이 일어나는 것을 확인할 수 있다.

이 코드를 CASE식의 응용 방법인 `표측/표두 레이아웃 이동 문제형식`으로 리펙토링하면 아래와 같이 바꿀 수 있다.

```SQL
SELECT prefecture, 
  SUM(CASE WHEN sex = 1 THEN pop ELSE 0 END) AS pop_men,
  SUM(CASE WHEN sex = 2 THEN pop ELSE 0 END) AS pop_won,
FROM population
GROUP BY prefecture;
```

### 2. 집약 결과로 조건 분기

| emp_id(직원ID) | team_id(팀 ID) | emp_name(직원 이름) | team(팀) |
|-------------:|--------------:|----------------:|--------:|
|          201 |             1 |             Joe |    상품기획 |
|          201 |             2 |             Joe |      개발 |
|          201 |             3 |             Joe |      영업 |
|          202 |             2 |             JIM |      개발 |
|          203 |             3 |            Carl |      영업 |
|          204 |             1 |            Bree |   상품 기획 |
|          204 |             2 |            Bree |      영업 |

SQL로 만들고 싶은 결과는 아래와 같다.

| emp_name | team       |
|----------|------------|
| JIM      | 개발         |
| Bree     | 2개 이상을 겸무  |
| Joe      | 3개 이상을 겸무  |

UNTINO을 사용해서 조회할때
```SQL
SELECT emp_name, '3개 이상을 겸무' as team
FROM employees
GROUP BY emp_id  (책에서는 emp_name으로 되어있지만 emp_id로 해야한다.)
HAVING COUNT(*) >= 3;
UNION
SELECT emp_name, '2개 이상을 겸무' as team
FROM employees
GROUP BY emp_id  (책에서는 emp_name으로 되어있지만 emp_id로 해야한다.)
HAVING COUNT(*) >= 2;
UNION
SELECT emp_name, max(team) as team
FROM employees
GROUP BY emp_id  (책에서는 emp_name으로 되어있지만 emp_id로 해야한다.)
HAVING COUNT(*) >= 1;
-----------------------------------------------------------
Append  (cost=0.01..0.07 rows=7 width=40)
  ->  HashAggregate  (cost=0.01..0.03 rows=3 width=40)
        Group Key: emp_id
        Filter: COUNT(*) >= 3
        ->  Seq Scan on employees  (cost=0.01..0.02 rows=7 width=40)
  ->  HashAggregate  (cost=0.01..0.03 rows=2 width=40)
        Group Key: emp_id
        Filter: COUNT(*) = 2
        ->  Seq Scan on employees  (cost=0.01..0.02 rows=7 width=40)
  ->  HashAggregate  (cost=0.01..0.03 rows=2 width=40)
        Group Key: emp_id
        Filter: COUNT(*) = 1
        ->  Seq Scan on employees  (cost=0.01..0.02 rows=7 width=40)
```
UNION으로 조회하면 테이블 풀스캔을 3번이나 진행을 한다. </br>
하지만 이것도 마찬가지로 CASE식으로 리펙토링을 진행할 수 있다.
```SQL
SELECT emp_name,
  CASE
    WHEN COUNT(*) >= 3 THEN '3개 이상을 겸무'
    WHEN COUNT(*) >= 2 THEN '2개 이상을 겸무'
    ELSE max(team)
  END AS team
FROM employees
GROUP BY emp_id;
-----------------------------------------------------------
HashAggregate  (cost=0.00..0.05 rows=7 width=40)
  Group Key: emp_id
  ->  Seq Scan on employees  (cost=0.00..0.02 rows=7 width=40)
```
이렇게 리펙토링을 진행하면 가독성이 좋아지고 테이블 풀스캔을 1번만 진행하게 된다.

## 10강 그래도 UNION이 필요한 경우
### 1. UNION을 사용할 수 밖에 없는 경우

서로 다른 테이블에서 데이터를 가져와 데이터를 merge하는 경우 UNION을 사용해야한다. 
```SQL
SELECT col_1
FROM Table_A
WHERE col_2 = 'A'
UNION ALL
SELECT col_3
FROM Table_B
WHERE col_4 = 'B';
```

### 2. UNION을 사용하는 것이 성능적으로 더 좋은 경우

인덱스가 있고 테이블에 데이터가 매우 많은 경우 case식보다 UNION을 사용하는것이 성능이 더 좋을 수 있다. </br>
이때 UNION방식은 index를 활용하여 데이터를 조회해서 여러번 조회해도 성능이 좋을 수 있다.  </br>
반면 case식으로 조회할때 데이터가 많아서 1번만 풀스캔을 진행해도 성능이 좋지 않을 수 있다. </br>
따라서 이것은 인덱스와 테이블의 데이터 양의 따라서 달라질 수 있기 때문에 성능 테스트를 진행해서 UNION을 사용해야할지 case식을 사용해야할지 판단해야한다. </br>

# 마치며

- SQL 성능은 IO를 얼마나 감소시킬지가 열쇠이다 </br>
- UNION을 조건 분기로 활용한다면 case식을 사용해서 바꿀 수 있지 않을까? 라고 의도적으로 생각해보자. </br>