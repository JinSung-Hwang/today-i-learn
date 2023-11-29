# 6장 AJAX와 JSON

## 6.1 REST 방식의 서비스 

브라우저외에도 다른 종류의 클라이언트가 늘면서 서버 사이드 프로그래밍은 화면보다는 순수 데이터를 가공, 적재하는 방식으로 변화하고 있다. </br>

### Ajax와 REST 방식의 이해

예전에는 화면 단위로 서버와 통신을 진행했다.  </br>
하지만 Ajax가 등장하면서 화면 전환 없이 서버와 통신할 수 있게 되었다. </br>
한편 예전에는 Ajax + XML을 통해서 서버와 통신을 했지만 요즘에는 Axios(HTTP Library) + JSON을 이용해서 서버와 통신을 한다. </br>

#### 클라이언트 중심의 개발

웹의 클라이언트 사이트 랜더링 기술이 발전하고 모바일은 모바일 플랫폼 안에서 개발이 이루어지다보니 서버는 순수 데이터만 전달하게 되었다. </br>
이러면서 자연스럽게 클라이언트와 서버의 역할 분배가 일어나게 되었다. </br>

#### JSON 문자열

'서버에서 순수한 데이터를 보내고 클라이언트가 데이터를 받아서 화면을 그린다'라는 개발 방식의 핵심은 "문자열" 이다. </br>
문자열은 어떤 프로그래밍 언어나 플랫폼에 종속되지 않는다. </br> 
문자열로 데이터를 표현하면 언어나 플랫폼에는 종속적이지 않지만 복잡한 데이터 구조를 같이 표현하기 위해서는 규칙이 필요하다. </br>
이런 규칙으로 데이터 구조를 표현한 것이 XML과 JSON이다.</br>
기존 XML은 JSON에 비해서 여는 태그 닫는 태그가 꼭들어가야해서 무겁고 복잡했다.</br>

#### REST 방식

REST(Representational State Transfer)방식은 통신시 자원(Resource)과 행동에 대한 인터페이스를 정의 내리기 위한 아키텍처 가이드이다.</br>
이 규칙을 잘 따르면 Restful하다고 한다.</br>

#### REST 방싁의 URL 설계

REST방식의 설계는 URL와 HTTP Method를 통해서 이루어진다.</br>
URL은 리소스는 정의하며 HTTP Method는 행동을 정의한다.</br>
리소스 정의에 대한 규칙은 아래와 같다.</br>
- 리소스는 복수로 표현한다. (개인적으로는 리소스는 단수로 사용하고 Bulk Create API에 -s를 붙이는것을 선호한다.)</br>
- 유일한 리소스를 특정할때는 url에 pathParameter로 id를 넣는다.</br>

HTTP Method는 사용 규칙은 아래와 같다.</br>
- GET: 데이터 조회 API에 사용한다.</br>
- POST: 데이터 생성 API에 사용한다.</br>
- PUT: 데이터 수정 API에 사용한다.</br>
- DELETE: 데이터 삭제 API에 사용한다.</br>

#### ReplyController 준비

@PostMapping에는 consumes라는 속성으로 지정한 타입으로 데이터를 받을 수 있다.</br>
@RestController는 @Controller와 @ResponseBody를 합친 어노테이션이다. </br>
String을 리턴해도 View를 호출하지 않으며 보통 JSON으로 데이터를 리턴할떄 사용한다.</br>

#### @Valid와 @RestControllerAdvice

서버를 호출하고 에러가 발생하면 어디에서 어떤 에러가 발생했는지 알아보기 어렵다.</br>
그런 이유로 @Valid를 사용하고 @RestControllerAdvice를 이용하면 컨트롤러에서 발생하는 예외에 대한 처리를 한곳에서 할 수 있다.</br>
물론 여기에서 일괄적으로 로그를 찍어서 확인할 수도 있다.</br>

## 6.2 다대일 Many To One 연관관계 실습

다대일 관계를 JPA로 어떻게 처리하는지 확인하여 보자.</br>

### 연관관계를 결정하는 방법

#### FK를 기준으로 결정
관계형 데이터베이스에서는 PK와 FK를 통해서 연관관계를 표현한다.</br>
반면 객체 지향에서는 객체의 참조를 통해서 연관관계를 표현한다.</br>
이 데이터베이스와 객제지향의 차이를 해결하기위해 JPA에서는 연관관계를 세팅해줘야한다.</br>
연관관계 세팅을 할떄는 DB에 FK가 있는 엔티티를 연관관계의 주인으로 결정하면 된다.</br>

#### 단방향과 양반향

JPA를 통해서 연관관계를 설정할때는 단방향과 양방향이 있다.</br>

양반향: 객체가 서로 참조하고 있는 형태이다. 양쪽 객체에 동일하게 적용해야하는 불편함이 있다. 하지만 객체 그래프 탐색을 할때 편리하다.</br>
단반향: 한쪽의 객체가 참조하고 있는 형태이다. 구현이 단순하고 에러 발생 여지가 줄어든다. 하지만 객체 그래프 탐색하기가 어렵다.</br>

단방향 참조 관계를 유지하면서 다른 엔티티를 사용해야 할때는 JPQL을 통해서 조인 처리를 사용할 수 있다.</br>
개인적으로는 양방향 엔티티를 이용하는것을 선호한다.</br>

### 다대일 연관 관계의 구현

```java
@Entity
@Table(name = "Reply", indexes = {
    @Index(name = "idx_reply_board_bno", columnList = "board_bno") // note: 쿼리 조건으로 자주 사용되는 칼럼에는 인덱스를 생성해둔다.
})
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@ToString(exclude = "board") // note: toString할때 참조객체가 있으면 참조 객체를 추가 조회하기때문에 exclude를 사용해서 제외시켜준다. 만약 board도 같이 출력하도록하면 board 테이블에 추가적인 쿼리를 실행한다.
public class Reply extends BaseEntity {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY) 
    private Long rno;
    @ManyToOne(fetch = FetchType.LAZY)  // note: fetch 속성은 보통 Lazy로 잡고 꼭 필요할때만 Eager로 잡는다.
    private Board board;
    private String replyText;
    private String replyer;
}
```

###### 댓글 조회/수정/삭제

JPA에서 엔티티 관계 방향을 단방향으로 구성하면 관리가 편하다.</br>
양방향으로 구성하면 양쪽 객체를 모두 구현해야하고 트랜잭션도 신경을 써야한다.</br>
다만 단방향이면 객체 그래프 탐색이 안되어서 MyBatis, JPQL, QueryDSL을 사용해야한다.</br>
사실 양방향을 잘 설정하면 에러도 나지 않고 객체 그래프 탐색도 편하게 할 수 있기에 양방향을 선호한다.</br>

### 댓글 서비스 계층의 구현

#### 특정 게시물의 댓글 목록

@PathVariable을 사용하면 URL의 값을 직접 파마미터 변수로 처리할 수 있다.</br>
