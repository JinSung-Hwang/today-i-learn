# LogQL이란?

LOKI에 수집된 로그를 필터링하기 위한 query language이다. </br>
logQL은 log 집계를 위해서 여러 source에 distributed grep을 진행한다. </br>
logQL에서 라벨과 명령어는 필터링을 위해서 사용된다. </br>
logQL은 `LogQueries`와 `MetricQueries`로 나누어진다. </br>
- logQueries는 로그 내용을 반환한다. </br>
- metricQueries는 logQueries가 반환한 쿼리 결과를 계산하기위해 확장되었다.

# LogQL 사용법

# LogQueries 사용법

![스크린샷 2024-01-01 오후 4 34 36](https://github.com/JinSung-Hwang/today-i-learn/assets/29647648/df2c04b8-9e44-4bfb-8c2d-f06da88997f4)

기본적 사용법은 `Log Stream Selector`으로 로그를 선택한 다음 `파이프라인`으로 로그를 필터링하는것이다.

## Log Stream Selector

```logQL
{ app="mysql", name="mysql-backup" }
```
기본 형식은 `{ label=value }`의 형태이다. </br>
셀렉터에 여러 라벨을 지정하려면 `,`로 여러 라벨을 추가할 수 있다. </br>
셀렉터에 사용할 수 있는 라벨 매칭 연산자는 아래와 같다.
- `=`: 정확히 일치하는 라벨을 선택한다.
- `!=`: 일치하지 않은 라벨을 선택한다.
- `=~`: 정규식으로 일치하는 라벨을 선택한다. 예시: `{name =~ "mysql.+"}`
- `!~`: 정규식에 일치하지 않는 라벨을 선택한다. 예시: `{name !~ `mysql-\d+`}`

주의: `filter regex expressions`와 달리 정규식은 라벨 값이 정확히 일치해야한다.

## Log Pipeline

log pipeline은 selector를 선택하고 추가적으로 붙일 수 있다.

log pipeline은 아래와 같이 3가지 카테고리로 나누어진다. 
1. filtering expressions - line or labe filter expressions
2. parsing expressions
3. formatting expressions - line or labe format expressions

### line filter expressions

셀렉터로 선택된 로그를 필터링할때 아래 연산자를 이용해서 로그를 필터링한다.

- `|=`: 로그에 특정 문자열이 포함되어 있는 로그를 수집한다.
- `!=`: 로그에 특정 문자열이 포함되어 있지 않으면 로그를 수집한다.
- `|~`: 정규식으로 매칭되는 로그가 있으면 로그를 수집한다.
- `!~`: 정규식으로 매칭되는 로그가 아니면 로그를 수집한다.

#### line filter expressions 예시

```logQL
{job="mysql"} |= "error"
```

```logQL
{job="mysql"} |= "error" != "timeout"
```

```logQL
{name="kafka"} |~ "tsdb-ops.*io:2003"
```



출처: https://grafana.com/docs/loki/latest/query/