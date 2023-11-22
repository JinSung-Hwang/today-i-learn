# 04장 스프링과 스프링 Web MVC

앞장에서 살펴보던 서블릿/JSP가 어떻게 변화하는지 알아보자 </br>

## 4.1 의존성 주입과 스프링

스프링의 핵심인 '의존성 주입'에 대해서 알아보자 </br>

### 스프링의 시작

요즘에 스프링은 웹 프레임워크로 많이 사용하지만 원래는 '의존성 주입 기법'을 적용할 수 있는 객체지향 프레임워크였다.  </br>
기존에 EJB와 같은 프레임워크로 개발을 진행했지만 상속을 사용하지 못하는 문제, 점점 비대해지는 프레임워크 등의 문제가 있었다.  </br>
이러한 문제를 해결하기 위해서 로드 존슨이 2002년에 집필한 'J2EE 설계 및 개발' 이라는 책의 예제코드를 기반으로 스프링 프레임워크를 만들었다.  </br>

#### 의존성 주입

의존성 주입은 '객체가 다른 객체를 사용할 때 그 객체를 직접 만들어서 사용하는 것이 아니라 주입 받아서 사용하는 방법'을 말한다. </br>
이런 의존성 주입 방법은 객체와 객체의 관계를 더 유연하게 만들기 위해서 사용합니다. </br>
보통 웹 개발할떄 controller가 service를 호출하게 되는데 이때 controller가 service를 호출하는것을 '의존적(dependent)'이라고 한다. </br>
즉 의존성이란 하나의 객체가 다른 객체의 도움이 필수적인 관계를 의미한다. </br>

스프링 프레임워크는 이러한 의존성을 관리하고 보다 객체간에 관계를 보다 유연하게 설정할 수 있도록 도와준다. </br>

#### 의존성 주입하기

스프링이(스프링 컨테이너) 관리하는 객체들을 빈(Bean)이라고 한다. </br>
어플리케이션(프로젝트)안에서 어떤 빈(Bean, 객체)를 관리할 것인지를 XML, 이나 자바 어노테이션을 통해서 설정할 수 있다. </br>

아래는 SampleDAO, SamplerService를 간단히 Bean등록하는 XML 예제이다. </br>
root-context.xml </br>
```xml
<?xml vension="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
<bean cLass="opg.zerock.sppingex.sample.SampleDAO"></bean> 
<bean cLass="org.zerock.springex.sample.SampleService"></bean>
</beans>
```
애플리케이션이 개발됨에 따라 빈 객체를 많이 등록하고 사용하기때문에 한번에 많이 등록하기보다는 등록하면서 자주 테스트하는것이 좋다. </br>

@Autowired는 '스프링에서 사용하는 의존성 주입 관련 어노테이션으로 만일 해당 타입의 빈이 존재하면 여기에 주입해주기를 원한다.'라는 의미이다. </br>
@ContextConfiguration 어노테이션은 스프링의 설정 정보를 로딩하기 위해서 사용한다. </br>

### ApplicationContext와 빈(Bean)
이전 장에서는 서블릿 객체를 관리하는 서블릿 컨텍스트에 대해서 이야기했다. </br>
스프링에서는 Bean 객체를 관리하기 위해서는 ApplicationContext를 활용한다. </br>

스프링이 실행되고 root-conext.xml을 읽으며 ApplicationContext객체가 생성된다. </br>
ApplicationContext를 읽으면 SampleDAO, SampleService 객체가 생성되고 이 객체들은 스프링이 관리하는 Bean이 된다. </br>

#### @Autowired의 의미와 필드 주입
@Autowired은 Autowired가 처리된 부분에 맞게 빈을 있는지 확인하고 빈을 주입해준다. </br>
멤버 변수에 직접 @Autowired를 선언하는 방식을 '필드 주입(Field Injection)'이라고 한다. </br>

#### @Service, @RRepository

서블릿에서도 @WebServlet이나 @WebFilter와 같은 어노테이션으로 서블릿 객체를 등록하듯이 스프링에서도 다양한 종류의 어노테이션으로 빈을 등록할 수 있다.  </br>
(아래는 스프링 2.5이후에 사용가능한 애노테테이션이다.)  </br>
1. @Controller: MVC의 컨트롤러 객체로 사용할 수 있도록 등록  </br>
2. @Service: 서비스 객체로 등록  </br>
3. @Repository: DAO 객체로 등록  </br>
4. @Component: 일반적인 객체로 등록  </br>

이렇게 어노테이션을 사용하면 스프링이 처음 시작할때 컴포넌트 스캔이라는 작업을 통해서 애노테이션 달려있는 클래스들을 찾아서 객체 생성후 Bean등록을 진행한다.  </br>

#### 생성자 주입 방식

스프링 초기에는 @Autowired를 통한 멤버 변수에 주입하거나 Setter를 통해서 의존성 주입하는 방식을 이용했지만 스프링 3이후로는 '생성자 주입 방식'이라고 부르는 방식을 사용한다.  </br>

생성자 주입 방식은 아래의 규칙에 맞춰서 작성해야한다.  </br>
1. 주입 받아야 하는 객체의 변수는 final로 작성한다.  </br>
2. 생성자를 이용해서 해당 변수를 생성자의 파라미터로 지정해야한다.  </br>

생성자 주입 방식은 객체를 생성할때 문제가 발생하는지 미리 알수 있어서 @Autowried나 Setter 방식보다 선호되는 방식이다.  </br>
Lombok을 이용하면 @RequiredArgsConstructor 어노테이션을 손쉽게 생성자 주입 방식을 사용할 수 있다.  </br>

#### 인터페이스를 이용한 느슨한 결합

스프링을 이용해서 의존성 주입이 가능하지만 더 근본적으로는 인터페이스를 이용해서 나중에 다른 객체로 쉽게 변경할 수 있도록 하는것이 좋다.  </br>
만약 SampleService가 SampleDAO에 의존적인 상황에서 SampleDAO가 변경되면 SampleService는 변경될 수 밖에 없다.  </br>
하지만 추상화된 타입이라면 이런 문제를 피할 수 있다.  </br>

#### SmapleDAO 인터페이스로 변경하기

SampleDAO를 인터페이스로 변경하고 SmapleDAOImple과 EventSampleDAOImple객체를 만들었다고 가정해보자  </br>
여기서 SampleService는 SampleDAO 인터페이스에만 의존적이기 때문에 SampleDAOImple과 EventSampleDAOImple로 변경되어도 SampleService는 변경될 필요가 없다.  </br>
하지만 의존성 객체를 주입할때는 어떤 구현체를 주입할지 잘 명시해줘야한다.  </br>

SampleService.java
```java
@Service
@RequiredApgsConstructor 
public class SampleService {
    private final SampleDAO sampleDAO; 
}
```
만약 위와 같은 SampleService가 구현되어있으면 SampleDAO를 가져올때 SmapleDAOImple과 EventSampleDAOImple중 어떤 객체를 가져와야할지 스프링은 알지 못한다.  </br>
그래서 이런 경우 사용할려고하는 객체에 @Primary 어노테이션을 붙여주면 된다.  </br>
```java
©Repository
@Primapy
public class EventSampleDAOImpl implements SampleDAO{ }
```

이런 방식 이외에도 @Qualifier를 이용하는 방식이 있다. </br>
예시

```java
@Repository
@Qualifier("normal")
public class SampleDAOImple implements SampleDAO{ }

@Repository
@Qualifier("event")
public class EventSampleDAOImple implements SampleDAO{ }

@Service
@RequiredArgsConstructor
public class SampleService {
    @Qualifier("normal")
    private final SampleDAO sampleDAO; 
}
```

##### 스프링의 빈(Bean)으로 지정되는 객체들
스프링 프레임워크에서 객체를 빈등록을 통해서 의존성 주입을 이용할 수 있다는 사실을 알았다.  </br>
하지만 모든 객체를 Bean으로 등록하는것은 아니다.  </br>

스프링의 빈으로 등록되는 객체들은 쉽게 말해 '핵심 배역'으로 활동하는 객체이다.  </br>
애플리케이션안에서 주로 오래 시간 동안 프로그램 내에 상주하면서 중요한 역할을 하는 '역할'중심의 객체들을 빈등록한다.   </br>

반대로 DTO나 VO같은 '역할' 보다는 '데이터'에 중점을 두는 객체들은 스프링의 빈으로 등록되지 않는다는 것이다.  </br>

##### XML이나 어노테이션으로 처리하는 객체

스프링에서 빈으로 등록할때 XML을 사용할지 어노테이션으로 등록할지는 '코드를 수정할 수 있는가?'로 판단하면 된다.  </br>
예를 들어 jar파일로 추가되는 클래스 객체를 빈으로 등록하려면 XML을 사용해야한다. 왜냐하면 해당 코드가 존재하지 않기때문이다.  </br>
반면에 직접 작성한 클래스 객체는 어노테이션을 사용해서 빈으로 등록하는것이 좋다.  </br>

### 웹 프로젝트를 위한 스프링 준비

스프링 구조를 보면 ApplicationContext라는 객체가 존재하고 빈으로 등록된 객체들을 ApplicationContext내에 생성해서 관리되는 구조이다.  </br>
ApplicationContext가 생성되려면 스프링이 로딩할때 웹 어플리케이션 내부에 ApplicationContext를 생성하는 작업을 진행해야하는데 Web.xml을 이용해서 리스너를 설정한다.  </br>

## 4.2 Mybatis와 스프링 연동
### Mybatis소개

Mybatis는 'SQL Mapping Framework'이다. </br>
SQL Mapping은 SQL의 실행 결과를 객체지향으로 '매핑'해준다는 뜻이다. </br>

Mybatis를 이용하면 편리함점  </br>
1. PreparedStatement/ResultSet을 Mybatis가 알아서 처리해주니 기존에 비해 코드를 줄일 수 있다. </br>
2. Connection/PreparedStatement/ResultSet의 close() 처리를 자동으로 해준다. </br>
3. SQL문을 별도의 파일이나 애노테이션으로 이용해서 SQL을 선언할 수 있다. </br>

#### MyBatis의 스프링의 연동 방식
MyBatis는 독립적으로 실행 가능한 프레임워크이지만, 스프링 프레임워크는 Mybatis와 연동해서 쉽게 처리할 수 있는 API를 지원한다. </br>

Spring과 Mybatis를 연동하는 방식은 2가지가 있다. </br>

1. Mybatis가 단독으로 개발하고 스프링에서 DAO를 작성해서 처리하는 방식 - 기존 DAO에서 SQL을 처리하는 방식을 Mybatis로 이용하는 구조 </br>
2. Mybatis와 스프링을 연동하고 Mapper인터페이스만 이용하는 방식 - 스프링과 Mybatis 사이에 'mybatis-spring'이라는 라이브러릴 이용해서 스프링이 데이터베이스를 전체처리하고 Mybatis는 일부 기능 개발에 활용하는 방식  </br>

#### XML로 SQL분리하기
Mybatis를 이용할때 @Select와 같은 애노테이션을 이용할 수 도 있지만 SQL길어지면 가독성이 많이 떨어진다.  </br>
하여 XML을 이용해서 별도의 파일로 분리하는것을 권장한다. </br>

XML 매퍼 인터페이스를 같이 결합할떄는 다음과 같은 과정으로 작성한다.  </br>
1. 매퍼 인터페이스를 정의하고 메소드를 선언한다. </br>
2. 해당 XML파일을 작성하고(파일 이름과 매퍼 인터페이스 이름을 같게)하고 <Select> 와 같은 태그를 이용해서 SQL을 작성한다. </br>
3. <Select> <Insert>등의 태그에 id속성 값을 매퍼 인터페이스의 메소드 이름과 같게 작성한다. </br>

예시
TimeMapper2.java
```java
public interface TimeMapper2 {
    String getNow(); 
}
```
TimeMapper2.xml
```xml
<?xmL version="1.0" encoding="UTF-8" ?> 
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.zerock.springex.mapper.TimeMapper2">
<select id="getNow" resultType="string"> 
    select now() 
</select>
</mapper>
```

## 4.3 스프링의 Web MVC 기초

#### 스프링 Web MVC의 특징

스프링 Web MVC(이하 스프링 MVC)는 Web MVC 패턴으로 모델, 뷰, 컨트롤러를 그대로 사용한다. </br>
다만 약간의 변화된 부분이 있는데 아래와 같다. </br>
1. Front-Controller패턴을 이용해서 모든 흐름의 사전/사후 처리를 가능하도록 설계되었다.  </br>
2. 어노테이션을 적극적으로 활용해서 최소한의 코드로 많은 처리가 가능하도록 만들었다. </br>
3. HttpServletRequest/HttpServletResponse를 직접 다루지 않고도 처리가 가능하도록 설계되었다. </br>

> Client ---> Front-Controller ---> Controller ---> Service ... ---> Controller ---> Front-Controller ---> View Template

#### DispatcherServlet과 FrontController 
스프링 MVC의 가장 중요한 사실은 모든 요청은 DispatcherServlet객체를 통해 실행된다. </br>
기존에 각 Controller에서 공통적으로 처리하는 로직을 FrontContoller인 DispatcherServlet에서 처리한다. </br>
프론트 컨트롤러가 사전/사후에 대한 처리를 하면서 약간식 다른 부분만 @Controller를 이용해서 처리한다. </br>

#### 스프링 MVC 사용하기

servlet-context.xml은 프로젝트 설정을 위한 파일이다.  </br>
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>
...
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"> 
      <property name="prefix" value="/WEB-INF/views/"></property>
      <property name="suffix" value=".jsp"></property> 
    </bean>
</beans>
```
InternalResourceViewResolver는 스프링 MVC에서 View를 어떻게 처리할지에 대한 설정을 담당한다. </br>
prefix는 앞에 붙일 경로를 의미하고 suffix는 확장자를 추가한것을 확인할 수 있다.  </br>
하여 view이름에 경로와 확장자를 붙이지 않아도 된다. </br>

#### 스프링 MVC 컨트롤러

기존 서블릿으로 Controller를 개발했을때와 스프링 MVC를 이용해서 Controller를 개발했을때의 차이점은 다음과 같다. </br>
1. 상속이나 인터페이스를 사용하지 않고 어노테이션으로만 처리가 가능하다. </br>
2. 오버라이드 없이 필요한 메서드를 정의가 가능하다. (doGet, doPost 등) </br>
3. 메서드의 파라미터를 기본 자료형이나 객체 자료형을 마음대로 지정 가능하다.  </br>
4. 메소드의 리턴타입도 void, String등 다양한 타입을 사용할 수 있다. </br>

예시
```java
@Controller // note: 스프링 빈으로 등록되기 위해서 붙인다. 메서드에서 String을 리턴하면 view를 찾아서 처리한다.
@Log4j2
public class SampleController {
    @GetMapping("/hello")
    public void hello() {
        log.info("hello........");
    }
}
```

#### @RequestMapping과 파생 어노테이션들

- @RequestMapping(컨트롤러, 메서드 선언부): 특정한 경로의 요청을 지정하기 위해서 사용한다. Get, Post둘다 처리할수있고 method 속성을 이용해 지정한 HTTP Method Type만 처리할 수도 있다. </br>
- @GetMapping(스프링4 추가, 메서드 선언부): GET방식의 요청을 처리하기 위해서 사용한다.  </br>
- @PostMapping(스프링4 추가, 메서드 선언부): POST방식의 요청을 처리하기 위해서 사용한다.  </br>

### 파라미터 자동 수집과 변환
파라미터 자동 수집은 DTO나 VO등을 메소드의 파라미터로 설정하면 자동으로 HttpServletRequest의 파라미터로 수집해주는 기능이다.  </br>
단순히 문자열 뿐만아니라 숫자, 배열, 리스트, 첨부파일 등 모두 가능하다. </br>
파라미터 자동 수집은 다음과 같이 동작한다.  </br>
- 기본 자료형은 자동으로 형변환처리가 가능하다. </br>
- 객체 자료형은 리플렉션 메서드를 통해서 getter/setter를 이용해서 자동으로 수집한다. </br>
- 객체 자료형의 경우 생성자가 없거나 파라미터가 없는 생성자가 필요하다.  </br>

#### 파라미터 관련된 애노테이션들
- @RequestParam은 파라미터로 값을 받을때 사용되는 애노테이션인데 간혹 파라미터가 없을때 default value를 지정할 수 있다.  </br>
- Formatter를 이용한 파라미터의 커스텀 처리: 컨트롤러에서 날짜 String값을 Date타입으로 변환하려면 Formatter를 이용한다. Formatter<LocaLDate>인터페이스를 구현해서 하고 빈등록해서 사용한다. </br>
  하지만 요즘에는 @JsonFormat 를 이용한다.

#### 객체 자료형의 파라미터 수집

아래와같이 DTO를 선언해두고 파라미터 타입에 DTO를 사용하면 자동으로 파라미터를 수집해준다. </br>

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class TodoDTO {
  private String tno;
  private boolean title;
  private LocalDate dueDate;
}

©Controller
@Log4j2
@RequestMapping("/todo")
public class TodoController {
  @PostMapping("/register")
  public void registerPost(TodoDTO todoDTO) {
    Log.info("POST todo register.... ");
    Log.info(todoDTO);
  }
}
```

#### Model이라는 특별한 파라미터
스프링 MVC에는 Model이라는 특별한 객체가 있다.  </br>
Model은 컨트롤러에서 처리되는 데이터를 View에 전달할때 사용되는 객체이다. </br>
Model에는 addAttribute()라는 메서드가 있는데 이 메서드를 이용해서 Model에 데이터를 추가할 수 있다. </br>

##### java beans와 @ModelAttribute
Model에 데이터를 추가할때는 addAttribute()를 이용해서 추가할 수 있지만 @ModelAttribute를 이용해서도 추가할 수 있다. </br>

```java
@GetMapping("/ex-url")
public void ex4Extra(@ModelAttribute("dto") TodoDTO todoDTO, Model model){
    Log.info(todoDTO); 
}
// note: 위 코드는 dto파라미터로 TodoDTO를 받아서 Model에 "dto"라는 이름으로 추가하는 코드이다.

@GetMapping("/ex-url")
public void ex4Extra(@ModelAttribute("todoDTO") TodoDTO todoDTO, Model model){
    Log.info(todoDTO);
}
// note: 위 코드는 아래와 같이 생략된 형태로 사용할 수 있다.
@GetMapping("/ex-url")
public void ex4Extra(TodoDTO todoDTO, Model model){
    Log.info(todoDTO);
}
```

#### RedirectAttributes와 리다이렉션
스프링 MVC에서 PRG 패턴을 적용하기 위해서는 RedirectAttributes를 이용해서 처리해야한다. </br>
Model과 마찬가지로 속성명과 값들을 넣어주면된다.  </br>
Redirect의 중요한 메서드는 아래와 같다.  </br>
1. addAttribute(키, 값): 리다이렉트할때 쿼리스트링이 되는 값을 지정한다. </br>
2. addFlashAttribute(키, 값): 일회용으로만 데이터를 전달하고 삭제되는 값을 지정한다. 일회용이란 값이 view에 전달되었을떄는 값을 사용하고 보이지만 새로고침하면 값이 보이지 않는다. </br>

```java
@GetMapping("/ex5")
public String ex5(RedirectAttributes redirectAttributes){
    redirectAttributes.addAttributeC'name"，"ABC"); 
    redirectAttributes.addFlashAttribute("result"， "success");
    return "redirect:/ex6"; 
}
```

#### 다양한 리턴 타입

스프링 MVC 컨트롤러 메서드의 다양한 리턴타입이 있다.  </br>
- void: 동일한 화면을 보여줄떄 사용한다. </br>
- 문자열: view로 전환할떄 사용하거나 또는 아래와 같이 특별한 방법을 사용하기 위해서 사용한다. </br>
    - 문자열을 리턴할때 아래와 같은 접두어를 사용할 수 있다. </br>
        - forward: return "forward:/ex1" :: 다른 url로 리다이렉션할때 사용한다. </br>
        - redirect: return "redirect:/ex1" :: url은 고정으로하고 내부적으로 다른 url로 처리하는 경우 사용한다. (특별한 경우 아니면 잘 사용하지 않음) </br>
- 객체나 배열, 기본 자료형: JSON 응답을 줄 때 사용함 </br>
- ResponseEntity: JSON 응답을 줄떄 사용함 </br>

#### 스프링 MVC에서 주로 사용하는 어노테이션들

컨트롤러 선언부에 사용하는 어노테이션 </br>
- ©Controller: 스프링빈의처리됨을명시 </br>
- @RestController: REST 방식의 처리를 위한 컨트롤러임을 명시 </br>
- @RequestMapping: 특정한 URL 패턴에 맞는 컨트롤러인지를 명시 </br>

메소드 선언부에 사용하는 어노테이션 </br>
- @GetMapping/@PostMapping/@DeleteMapping/@PutMapping...: HTTP 전송 방식 (method)에 따라 해당 메소드를 지정하는 경우에 사용, 일반적으로 @GetMapping과 @PostMapping을 주로 사용 </br>
- @RequestMapping: GET/POST 방식 모두를 지원하는 경우에 사용 </br>
- @ResponseBody: REST 방식에서 사용 </br>

메소드의 파라미터에 사용하는 어노테이션 </br>
- @RequestParam: Request에 있는 특정한 이름의 데이터를 파라미터로 받아서 처리하는 경우에 사용 </br>
- @PathVariable: URL 경로의 일부를 변수로 삼아서 처리하기 위해서 사용 </br>
- @ModelAttribute: 해당 파라미터는 반드시 Model에 포함되어서 다시 뷰(View)로 전달됨을 명시(주로 기본 자료형이나 Wrapper 클래스, 문자열에 사용) </br>
- 기타: @SessionAttribute, @Valid, @RequestBody 등 </br>

### 스프링 MVC의 예외처리

스프링 MVC에서 일반적인 예외처리 방법은 @ControllerAdvice를 이용해서 처리한다. </br>
@ControllerAdvice도 역시 빈으로 등록된다. </br>
```java
@ControLlerAdvice
@Log4j2
public class CommonExceptionAdvice {
}
```
#### @ExceptionHandler
@ControllerAdvice는 @ExceptionHandler라는 어노테이션을 사용할 수 있다. </br>
@ExceptionHandler는 전달되는 Exception 객체들을 지정하고 메소드의 파라미터를 이를 이용할 수 있다. </br>

```java
©ControllerAdvice
@Log4j2
public class CommonExceptionAdvice {
    @ResponseBody // note: view를 호출하지 않고 JSON데이터를 그대로 전송할때 사용하는 어노테이션이다.
    @ExceptionHandler(NumberFormatException.class) // note: 지정한 예외가 발생하면 아래 메서드가 실행되고 아래의 파라미터로 예외를 전달 받을 수 있다.
    public String exceptNumber(NumberFormatException numberFormatException) {
        log.error("-------------------------------------- "); 
        log.error(numberFonmatException.getMessage());
        return "NUMBER FORMAT EXCEPTION"; 
    }

    @ResponseBody
    @ExceptionHandler(Exception.class) // note: 이렇게 예외의 최상위 타입인 Exception을 지정하여 모든 예외를 처리할 수 있다.
    public String exceptCommon(Exception exception) {
        log.error("-------------------------------------------------"); 
        log.error(exception.getMessage());
        return "EXCEPTION"; 
    }

    @ExceptionHandlep(NoHandlerFoundException.class)  // note: 없는 URL을 호출했을떄 발생하는 예외를 지정한다.
    @ResponseStatus(HttpStatus.NOT_FOUND) // note: 응답 상태 코드를 지정할 수 있다.
    public String notFound() {
        return "custom404"; 
    }
}
```

## 4.4 스프링 Web MVC 구현하기

이전 장에서 Servlet으로 개발했던 Todo예제를 스프링 MVC로 개발해보자 </br>

### 프로젝트의 구현 목표와 준비

@Configuration: 스프링의 설정 파일임을 명시한다. </br>
@Bean: 메서드의 실행 결과를 객체를 스프링의 빈으로 등록한다. </br>

### MyBatis와 스프링을 이용한 영속 처리

MyBatis와 스프링을 이용해서 개발하면 기존 JDBC를 이용할떄보다 코드의 양이 줄어들어 편리하다. </br>

MyBatis로 개발하면 아래의 단계로 개발하게 된다. </br>
1. VO선언
```java
public class TodoVO {
    private Long tno;
    private String title; 
    private LocalDate dueDate; 
    private String writer; 
    private boolean finished;
}
```
2. Mapper 인터페이스의 개발
```java
public interface TodoMapper {
    String getTime();
}
```
3. XML의 개발
```xml
<?xml version="1.0" encoding="UTF-8" ?> 
<!DOCTYPE mapper 
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd"> 
<mapper namespace="org.zerock.springex.mapper.TodoMapper">
<select id="getTime" resultType="string">
    select now() 
    </select>
</mapper>
```
4. 테스트 코드의 개발
```java
@Log4j2
@ExtendWith(SpringExtension.class) 
@ContextConfiguration(locations="file:src/main/webapp/WEB-INF/root-context.xml") 
public class TodoMapperTests {
    @Autowired(required = false) 
    private TodoMapper todoMapper;
    
    ©Test
    public void testGetTime() {
        log.info(todoMapper.getTime());  // note: 테스트 코드는 최대한 로그로 파악하기보다는 assert로 확인해야 사람이 직접 틀린곳을 일일히 파악하지 않는다.
    }
}
```

### TodoMapper 인터페이스와 XML

SQL실핼 로그를 자세히 살펴 보고 싶다면 패키지 로그를 TRACE 레벨로 기록하도록 Log4j2.xml을 만들고 아래 코드를 추가해야한다. </br>

```xml
<loggers>
    <logger name="org.springframework" level="INFO" additivity="false">
        <appender-ref ref="console" /> 
    </logger>
  
    <logger name="org.zerock" level="INFO" additivity="false"> 
        <appender-pef ref="console" />
    </logger>

    <logger name="org.zerock.springex.mapper" level="TRACE" additivity="false">
        <appender-ref ref="console" />
    </logger>
    
    <root level="INFO" additivity="false"> 
        <AppenderRef ref="console"/>
    </root> 
</loggers>
```

### Todo 기능 개발

TodoMapper -> TodoService -> TodoController -> JSP 순서로 개발한다. (예전에 데이터 중심 개발을 할때의 개발 순서이다. 요즘에는 테이블 보다는 도메인 객체를 먼저 만든다.) </br>
여기서 중요한 프로세스는 TodoMapper와 TodoService를 만들고 테스트를 돌려본다. </br>
그리고 테스트가 성공하면 TodoController를 만들고 테스트를 돌려본다. </br>

MyBatis에서는 SQL에 ? 대신해서 #{variable} 형태로 파라미터를 지정한다. </br>

#### @Valid를 이용한 서버사이드 검증

Client(Web, App, PC, KIOSK)등 Client가 다양해지면서 서버사이드 유효성 검증은 필수적이고 사용자의 편의성을 위해서 Client에서도 유효성 검증을 해야한다. </br>
스프링 MVC에서는 @Valid과 BindingResult를 이용해서 서버사이드 유효성 검증을 처리할 수 있다. </br>

hibernate-validator을 사용하는 대표적인 어노테이션은 다음과 같다.  </br>
- @NotNull: null이 아닌 값이어야 한다. </br>
- @Null: null만 입력 가능하다. </br>
- @NotEmpty: null이 아니고 값이 비어있지("") 않아야 한다. </br>
- @NotBlank: null이 아니고 값이 비어있지("") 않고 공백이(" ") 아니어야 한다. </br>
- @Size(min=, max=): 문자열, 배열의 길이가 min과 max사이여야 한다. </br>
- @Pattern(regex=): 정규식을 만족해야 한다. </br>
- @Max(num): num보다 작거나 같아야 한다. </br>
- @Min(num): num보다 크거나 같아야 한다. </br>
- @Future: 현재보다 미래여야 한다. </br>
- @Past: 현재보다 과거여야 한다. </br>
- @Positive: 양수여야 한다. </br>
- @PositiveOrZero: 양수이거나 0이어야 한다. </br>
- @Negative: 음수여야 한다. </br>
- @NegativeOrZero: 음수이거나 0이어야 한다. </br>

아래와 같이 어노테이션을 붙여 간단히 유효성 검증을 할 수 있다. </br>

```java
@...
public class TodoDTO {
  private Long tno;
  @NotEmpty
  private String title;
  @Future
  private LocalDate dueDate;
  private boolean finished;
  ©NotEmpty
  private String writer;
}

@Controller
public class TodoController {
      @PostMapping("register")
      public String registerPost(@Valid TodoDTO todoDTO, BindingResult bindingResult, RedirectAttributes redirectAttributes){
          // note: @Valid를 이용해서 유효성 검증을 하고 BindingResult 파라미터를 추가해서 에러를 확인할 수 있다.
          if(bindingResult.hasErrors()) { 
            log.info("has errors......"); 
            return "/todo/register";
          }
          ...
      }
}
```

### 페이징 처리를 위한 TodoMapper

사용자가 다 사용하지 않을지도 모르는 데이터를 DB에서 한번에 조회해서 넘겨주는것은 비효율적이다. </br>
그래서 페이징 처리를 해서 사용자가 필요한 만큼만 데이터를 가져오는것이 효율적이다. </br>

#### 페이징을 위한 SQL 연습

limit
```sql
select * from tbl_todo order by tno desc limit 10; -- 10개씩 가져온다.
select * from tbl_todo order by tno desc limit 20, 10; -- 20개씩 건너뛰고 10개씩 가져온다.
select * from tbl_todo order by tno desc limit 20 offset 10; -- 20개씩 건너뛰고 10개씩 가져온다.
```

limit의 단점은 식 사용이 불가능하고 오직 숫자만 입력이 가능하다.

### 검색/필터링 조건의 정의

#### types에 따른 동적 쿼리

검색 필터링을 하기위해서는 조건에 따라 변화되는 SQL인 동적쿼리를 작성해야한다. </br>
MyBatis에서는 동적 쿼리를 지원하기 위해서 아래의 문법을 지원한다.  </br>
- if: 조건에 따라서 SQL을 추가하거나 무시하기 위해서 사용한다. </br>
- trim: 조건에 따라서 불필요하게 생성 될 수 있는 SQL을 제거하는데 사용한다. </br>
- choose: switch문과 비슷하다. </br>
- forEach: 반족적으로 처리할떄 사용한다. </br>

#### types에 따른 동적 쿼리

`<forEach>`태그의 대상은 List, Map, Set과 같은 컬렉션 계열이랑 같이 사용한다.
`<where>`태그는 `<where>` 태그 안쪽 문자가 생성되어야만 where절이 생성된다.

```xml
<select id="selectList" resultType="org.zerock.springex.domain.TodoVO">
  select * from tbl_todo
  <where>
  <forEach collection="types" item="type" open="(" close=")" separatop=" OR ">
    <if test="type == 't'.toString()">
      title like concat('%', #{keyword}, '%')
    </if>
    <if test="type == 'w'.toString()">
      writer like concat('%', #{keyword}, '%') 
    </if>
  </foreach>
  order by tno desc limit #{skip}, #{size}
  </where>
</select>
```

`<sql>`과 `<include>`: `<sql>`은 SQL문을 정의하고 `<include>`는 정의된 SQL문을 가져와서 사용한다. </br>