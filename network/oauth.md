용어 정리

1. Resource Owner: 사용자, 웹 서비스를 이용하려는 유저, 자원(개인정보)을 소유하는 자
2. Resource Server: 사용자의 개인정보를 가지고있는 애플리케이션 회사 서버(Google, Facebook, Naver, Kakao 등)
3. Service Provider(Client): 서비스 제공자

# OAuth 란?

Open Authorization(인가, 허가)의 약자이다. </br>
`Resource Owner`이 `Service Provider`에 아이디와 비밀번호를 제공하지 않고 `Resource Server`의 `Resource Owner`의 정보에 대해 접근 권한을 부여할 수 있는 수단으로써 사용되는 개방형 표준이다.

# OAuth 사용 이유

사용자들이 서비스 제공자에 비밀번호을 제공하지 않아도 되고 번거로운 회원가입 절차를 다시 진행하지 않아도 된다. </br>
그러면서도 서비스 제공자는 유저의 정보를 얻을 수 있어 사용자와 서비스 제공자 모두 만족스럽다.

`Client`가 `Resource Server`로부터 `Resource Owner`의 정보를 받는 가장 간단한 방법은 무엇일까? </br> 

`Resource Owner`가 직접 `Resource Server`에 등록된 ID, Password를 직접`Service Provider`에게 전달하면, `service provider`가 `Resource Server`에 접속하여 정보를 가져오는 것이다.

그러나 이러한 방식은 3개의 주체 모두에게 불편한 결과를 초래한다. 각각의 입장에서 다음과 같은 문제점을 가지고 있다.

> Resource Owner: 내 ID, Password를 누군지도 모르는 Service Provider에게 넘겨야 하는게 불안하다

> Service Provider: Resource Owner로부터 ID, Password를 받았지만 이 정보를 해킹당해서 정보를 유출시킬 리스크가 부담스럽다

> Resource Server: 누군지도 모르는 Service Provider에게 내가 가진 Resource Owner의 개인정보를 함부러 제공하기 불편하다

# OAuth 2.0 절차

OAuth는 4가지 절차 타입으로 구현된다.

1. **Authorization Code Grant Type:** </br>
   사용자가 로그인후 Authorization Code를 Service Provider에게 전달한다. </br> 
   Service Provider는 Authorization Cdoe를 인증하여 AccessToken을 받는다.

2. **Implicit Grant Type:** </br>
   Authorization Code Grant Type에서 Authorization Code 절차가 없고 바로 AccessToken을 받는 방식이다.</br>
   (특별히 안전한 저장공간이 없는 Javascript SPA(Single Page Application)에 사용하기 위해 만들어졌지만 권장하지는 않는 유형이다.)

3. **Resource Owner Password Credentials Grant Type**: </br>
   사용자의 유저 아이디와 패스워드를 모두 Service Provider에게 넘기기고 Service Provider가 직접 인증하고 Access Token을 받는 방식

4. **Client Credential Grant Type**: </br>
   Service Provider없이 앱에서만 Client에서만 OAuth를 사용하는 방식


4번 Authorization Code Grant Type 방식을 살펴보자. </br> 

[그림]

위 그림 설명

1. Client는 Service Provider이다.
2. Redirect URI는 HTTP STATUS 301, 302응답으로 주면 바로 리다이렉트 시킬 수 있다.
3. 8번에서 Authorization Code를 검사하려면 Client Secret Key가 있어야한다.
