# 2장 SQL 기초

## 6강 Select 구문

Select는 DBMS에서 데이터를 조회하는 명령어이다.  </br>

### 기본 Select 구문을 살펴보자
- Select절: 조회될 결과를 기술하는 절이다. 반드시 입력되어야한다. </br>
- From절: 조회할 테이블을 지정하는 절이다. 반드시 입력되어야한다. </br>
- Where절: 조회할 테이블에 조건에 맞는 데이터를 필터링하기 위한 조건절을 지정하는 절이다. 생략 가능하다. </br>
    - 다양한 연산자 입력이 가능하다. (=, <>, >=, >, <=, <, BETWEEN, IN, LIKE, AND, OR, NOT 등) </br>
    - Null인 값을 선택하려면 `칼럼명 is null`, `칼럼명 is not null` 키워드를 사용해야한다. 
- GroupBy절: `1. GroupBy절에 나열되는 칼럼명들을 기준으로 From, Where의 Query 결과 데이터를 묶은` 다음 `2. 묶음 데이터마다 집계하기`위한 절이다. 생략 가능하다. (집계 함수는 count, sum, avg, max, min 등이 있다.) </br>
- Having절: GroupBy절에서 묶은 데이터의 필터링을 지정하는 절이다. 생략 가능하다. </br>
- OrderBy절: 조회된 결과를 정렬 조건을 지정하기 위한 절이다. 생략 가능하다. </br>

```SQL
-- note: 위 설명을 바탕으로 만든 SQL 쿼리 문이다.

SELECT 과목, AVG(성적) AS 평균성적
FROM 학생
WHERE 학년 = 2 AND 과목 IS NOT NULL
GROUP BY 과목
HAVING AVG(성적) > 50
ORDER BY 평균성적 DESC;
```

### View와 서브쿼리

View는 하나 이상의 테이블로 만든 논리적 테이블이다. </br>
실제로 뷰 테이블 형태로 데이터를 저장하고 있지는 않는다. 하나의 SQL Select 구문을 저장한 것과 같다. </br>
가령 특정 Select문을 자주 사용하면 View라는 논리적 테이블을 만들고 이를 활용하는 용도로 사용된다. </br>

기본 문법 
```SQL
CREATE VIEW [뷰이름](칼럼명1, 칼럼명2, ...) AS ...;
```
예시
```SQL
CREATE VIEW countAddress (v_address, cnt)
AS
  SELCET address, count(*)
  FROM Address
  GROUP BY address;
```

```SQL
-- note: 만들어진 뷰를 사용하여 뷰를 조회하는 예시이다.
SELECT v_address, cnt
FROM CountAddress;

-- note: 뷰를 실행할때 SELECT 구문으로 전개된다.
SELECT v_address, cnt
FROM (
  SELECT address, count(*)
  FROM Address
  GROUP BY address) as CountAddress;
```
이 처럼 view는 Select문을 저장한것과 같다.
위 처럼 FROM절에 Select문이 들어간것을 SubQuery라고 부른다.

### 서브쿼리
서브 쿼리를 이용해서 Select문의 조건절을 동적으로 지정할 수 있다. 

```SQL
-- note: 상수 값을 이용하여 조회한 예시이다.
SELECT name
FROM address
WHERE name IN ('인성', '민', '준서');

-- note: 반면에 서브쿼리를 이용하여 조건절을 동적으로 지정한 예시이다.
SELECT name
FROM Address
WHERE name IN (SELECT name FROM Address2);
```

## 7강 조건 분기, 집한 연산, 인도우 함수, 갱신

### 1. SQL과 조건 분기
SQL에서 조건 분기는 `CASE 식`을 기준으로 한다. </br>
이 CASE 식은 `단순 CASE 식`과 `검색 CASE 식` 두가지로 나뉜다. </br>
`검색 CASE 식`은 `단순 CASE 식`을 포함하기 때문에 `검색 CASE 식`만 기억해도 된다. </br>
```SQL
-- note: 검색 CASE 식을 이용한 예시이다.
SELECT 이름, 
  CASE 
    WHEN [평가식] THEN [식]
    WHEN 점수 >= 80 THEN 'B'
    WHEN 점수 >= 70 THEN 'C'
    ELSE 'F'[식]
  END AS 성적
FROM 학생;
```
프로그래밍언어의 switch문과 SQL의 case 식은 비슷하지만 차이점은 `case 식`은 특정한 값(상수)를 리턴한다. </br>
**즉 case 식은 특정 값으로 교환한다** 라고 생각하면 된다. ex) 점수 -> 'A'

### 2. SQL과 집합 연산
집한 연산은 테이블을 하나의 집합으로 두고 이 테이블을 대상으로 집합 연산을 하는것을 말한다. </br>
집한 연산에는 `합집합(union)`, `교집합(intersect)`, `차집합(except)`가 있다. </br>
```SQL
-- note: 합집합 연산을 이용한 예시이다. 중복 포함하려면 union all을 사용하면 된다.
SELECT * FROM address
UNION
SELECT * FROM address2;

-- note: 교집합 연산을 이용한 예시이다.
SELECT * FROM address
INTERSECT
SELECT * FROM address2;

-- note: 차집합 연산을 이용한 예시이다. address - address2와 같기때문에 순서가 중요하다.
SELECT * FROM address
EXCEPT
SELECT * FROM address2;

```

### 3. 윈도우 함수
윈도우 함수는 데이터를 가공하기 때문에 중요하다. </br>
이것을 한마디로 표현하면 group by에 집약 기능이 없는 함수라고 생각하면 된다. </br>
윈도우 함수는 Over절을 작성하고 PartitionBy또는 OrderBy절을 사용한다. </br>

```SQL
-- note: 윈도우 함수를 이용한 예시이다.
SELECT address, 
  COUNT(*) OVER(Partition BY address) 
FROM Address;

SELECT name, age,
  RANK() OVER(Order By age DESC) as rnk
FROM Address;
-- note: 집약 기능이 없이때문에 SELECT * FROM Address와 레코드 수가 동일하다.

address | count
--------|------
속초시   | 1
--------|------
인천시   | 2
인천시   | 2
--------|------
서울시   | 3
서울시   | 3
서울시   | 3
```

### 4. 갱신

SQL에서 갱신은 3가지가 있다.
1. INSERT: 데이터를 추가하기 명령어이다.
```SQL
INSERT INTO 테이블이름([필드1], [필드2], ...) 
VALUES ([값1], [값2], ...);

-- note: 예시
INSERT INTO Address(name, age, city) 
VALUES('홍길동', 20, '서울시')
```
2. DELETE: 데이터를 삭제하는 명령어이다. (레코드의 일부만 삭제할 수 없다.)
```SQL
DELETE FROM 테이블이름
WHERE [조건];

-- note: 예시
DELETE 
FROM Address
WHERE city = '서울시';
```
3. UPDATE: 데이터를 수정하는 명령어이다.
```SQL
UPDATE 테이블이름
SET [필드1] = [값1], [필드2] = [값2], ...
WHERE [조건];

-- note: 예시
UPDATE Address
SET phone = '01012341234', age=20, ...
WHERE city = '서울시';
```
