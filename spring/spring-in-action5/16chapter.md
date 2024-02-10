1. 스프링 부트 액추에이터 활성화하기
2. 액추에이터 엔드포인트 살펴보기
3. 엑추에이터 커스터마이징
4. 액추에이터 보안 처리하기

목차
1. 

# 16.1 스프링 부트 액추에이터 사용하기

# 스프링 액추에이터란
스프링 부트 어플리케이션의 상태, 모니터, 매트릭과 관련된 기능을 HTTP와 JXM을 통해 살펴볼 수 있는 기능이다. </br>
스프링 부트 내부 상태를 볼 수 있게하며 애플리케이션의 작동 방법을 제어할 수 있다.

내부 상태를 볼 수 있는 것들
1. 애플리케이션 환경에서 사용할 수 있는 구성 속성들
2. 애플리케이션의 포함된 다양한 패키지의 로깅 레벨
3. 애플리케이션이 사용 중인 메모리
4. 지정된 엔드포인트가 받은 요청 횟수
5. 애플리케이션의 health 상태 정보

# 액추레이터 사용법 
아래 의존성을 추가해야한다.
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## 액추레이터 엔드포인트

/health: 애플리케이션의 건강 상태 정보를 반환한다.
/info: 개발자가 정의한 애플리케이션의 관한 정볼르 반환한다.
/metrics: 애플리케이션의 메트릭 정보를 반환한다.

16.1.1 액추에이터의 기본 경로 구성하기
액추레이터 엔드포인트 경로에 기본적으로 /actuator가 붙는다. 
이 기본으로 붙는 경로를 변경하려면 application.properties 파일에 아래와 같이 설정한다.
```yml
management:
  endpoints:
    web:
      base-path: /management
```

16.1.2 액추에이터 엔드포인트의 활성화 비활성화
액추에이터 /health, /info만 기본 활성화되어있다. </br>
나머지 경로는 보안상의 이유로 비활성화 되어있다.

```yml
management:
  endpoints:
    web:
      exposure:
        include: '*' # 모든 엑추레이터 경로 활성화
        exclude: threaddump, heapdump # 특정 엑추레이터 경로 비활성화
```

16.2 액추에이터 엔드포인트 소비하기

curl을 통해서 기본 액추에이터 엔드포인트를 요청하면 활성화된 액추에이터 엔드포인트의 정보를 얻을 수 있다.
```
curl localhost:8081/actuator
{
  "_links": {
    "self": {
      "href": "http://localhost:8081/actuator",
      "templated": false
    },
    "health": {
      "href": "http://localhost:8081/actuator/health",
      "templated": false
    },
    "info": {
      "href": "http://localhost:8081/actuator/info",
      "templated": false
    }
    ...
  }
}
```
/info와 /health는 가장 기본되는 앤드포인트이며 /info는 애플리케이션의 정보를 /health는 애플리케이션의 health 상태를 반환한다. </br>

application.yml에 info 속성에 아래와 같이 설정하면 /info 앤드포인트에 정보를 반환한다.
```yml
info:
  contact:
    email: js@gmail.com
    phone: 01012341234
```

애플리케이션의 건강 상태 살펴보기
curl localhost:8081/actuator/health
```
{
  "status": "UP" | # 접근 가능한 상태
            "down" | 접근 할 수 없는 상태.
            "unknown" | # 외부 시스템의 상태가 분명하지 않는 상태
            "out_of_service" # 외부에서 접근할 수 있지만 현재 사용할 수 없는 상태
}
```

### 16.2.2 구성 상세 정보 보기

/beans: 애플리케이션 컨텍스트에 있는 모든 빈의 상세 정보를 얻을 수 있다. </br>
/conditions: 자동-구성 내역을 알아볼 수 있다. 스프링과 달리 스프링부트는 빈설정이 자동으로 구성되는데 이것을 통해서 자동빈구성내역을 살펴볼 수 있다.  </br>
/env: application.properties, application.yml, 환경 변수 jvm 시스템 속성등을 살펴 볼 수 있다.


### 16.4 액추에이터 보안 처리하기
액추에이터는 애플리케이션에 치명적인 환경 속성 정보를 볼수도 있고 로깅 레벨을 변경할 수도 있기때문에 보안처리하는것이 좋다.
보안 기능은 액추에이터의 책임이 아니며 스피링 시큐리티의 책임이기에 스프링 시큐리티의 설정을 통해서 보안처리를 해야한다.
```java
public class WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    // note: V1 액추에이터 보안 설정 방법
    http
        .authorizeRequests()
          .antMatchers("/actuator/**").hasRole("ADMIN") // note: 액추에이터 엔드포인트가 application.yml에서 변경되면 /actuator/**도 같이 변경해야한다.
        .and()
        .httpBasic();
    
    // note: V2 모든 액추에이터 경로를 열고 특정 액추에이터 경로를 제외하는 방법
    http
        .requestMatcher(EndpointRequest.toAnyEndpoint().excluding("health", "info")) // note: EndpointRequest.toAnyEndpoint()는 어떤 액추에이터 엔드포인트와 일치하는 요청을 반환한다. 특정 액추에이터를 제외하고 싶으면 excluding 메서드를 사용한다.
        .authorizeRequests()
          .antMatchers().hasRole("ADMIN")
        .and()
        .httpBasic();
    
    // note: V2 특정 액추에이터 경로만 보안성정을 지정하는 방법
    http
        .requestMatcher(EndpointRequest.to("beans", "threaddump", "loggers"))
        .authorizeRequests()
          .antMatchers().hasRole("ADMIN")
        .and()
        .httpBasic();
  }
}
```
