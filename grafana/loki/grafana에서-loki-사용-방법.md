# Grafana에서 Loki 사용 방법

Grafana에서 Loki 사용 방법을 기록한다.

![grafana-login](https://github.com/JinSung-Hwang/today-i-learn/assets/29647648/11d95902-1e0e-45a1-b11a-d5db3b515cef)

1. grafana에 접속하여 로그인을 진행한다.

![grafana-explore](https://github.com/JinSung-Hwang/today-i-learn/assets/29647648/657757a9-419f-4372-81b9-a8f3c073ce96)

2. 왼쪽 상단에 Toggle Menu를 클릭하고 `Explore` 버튼을 클릭한다.

![loki-logql](https://github.com/JinSung-Hwang/today-i-learn/assets/29647648/08635094-caf0-4e8b-8d50-b5d4b1935ae5)

3. 번호 순서대로 값을 변경하여 Log를 조회한다.
   1. data source picker 선택: grafana에서 보고싶은 data source를 선택한다. 여기서는 loki를 선택한다.
   2. time range 선택: 조회한 log의 기간을 선택한다.
   3. code pannel 선택: builder보다는 logQL을 직접 작성할 수 있는 code pannel을 이용한다.
   4. logQL작성: selector와 log pipeline을 이용해서 원하는 로그를 필터링하여 살펴본다. </br>
      ```{ app_name="application_name", app_stage="production" } |~ "lineFilterValue" | json``` </br>
      이미지에서는 위와 같이 selector, lineFilterExpression, parserExpression을 이용하여 logQL을 작성했다.
   5. run query 클릭: data source에 time range와 logQL에 맞는 로그들을 조회한다.