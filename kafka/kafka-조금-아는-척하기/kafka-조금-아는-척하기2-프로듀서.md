목차 </br>
프로듀서
1. 프로듀서 코드
2. 프로듀서의 기본 흐름과 기본 동작
3. 처리량과 관련 주요 속성
   1. batch.size란?
   2. linger.ms란? 
4. 전송 결과 확인하는 방법 2가지? 
   1. Future
   2. Callback
5. 전송 보장, ACK, `min.insync.replicas` 설정
   1. ACK의 `0`, `1`, `ALL`설정값은?
   2. 브로커의 min.insync.replicas란?
   3. min.insync.replicas의 주의사항 2가지는?
6. 에러 유형
   1. 메세지 전송전 발생 에러
   2. 메시지 전송 과정중 발생 에러
7. 실패 대응
   1. 재시도
      1. 재시도와 메시지 중복 전송 가능성
      2. 재시도와 메세지 순서 보장
   2. 기록
8. 재시도와 메시지 중복 전송 가능성


# 프로듀서

## 프로듀서 실행 코드

```java
Properties prop = new Properties(); // note: 프로듀서 속성을 설정한다. 아래 설정 이외에 batch.size, linger.ms등 producer의 속성을 설정할 수 있다.
prop.put("bootstrap.servers", "kafka01:9092,kafka01:9092,kafka01:9092"); // note: 브로커의 주소를 설정한다.
prop.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer"); // note: 키의 직렬화 방법을 설정한다.
prop.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer"); // note: 밸류의 직렬화 방법을 설정한다.

KafkaProducer<Integer, String> producer = new KafkaProducer<>(prop); // note: 위에서 만들었던 속성을 이용해서 프로듀서를 생성한다.

producer.send(new ProducerRecord<>("topicname", "key", "value")); // note: 프로듀서로 ProducerRecord를 전송한다. ProducerRecord는 2가지 형태가 있다. 첫번째 형태 토픽-키-밸류 두번쨰 형태 토픽-밸류 형태이다.
producer.send(new ProducerRecord<>("topicname", "value"));

producer.close(); // note: 메세지를 전송했으면 프로듀서를 닫아준다.
```

## 프로듀서의 기본 흐름과 기본 동작

<img width="2394" alt="스크린샷 2023-12-27 오후 8 27 19" src="https://github.com/JinSung-Hwang/today-i-learn/assets/29647648/ee3bcbe0-ec2b-425e-a811-b1f8c3602084">

프로듀서의 메시지 전송 흐름은 아래와 같다. </br>
1. 속성을 설정한다. 
2. send()메서드를 호출해서 메시지를 보낸다.
3. 프로듀서는 메세지를 serializer를 통해서 byte[]로 변환한다.
4. partitioner가 메세지의 토픽과 키를 기반으로 파티션을 선택하고 버퍼안에 batch에 메세지를 담는다.
5. batch에 메시지가 차면 sender가 바로 메세지를 브로커에 전달한다. 물론 가득차지 않아도 linger.ms가 지나면 sender가 메세지를 브로커에 전달한다.
6. 필요에 따라서 재시도를 진행한다.

send()메서드와 sender는 서로 다른 스레드에서 동작한다. </br>
하여 send()메서드가 batch에 메세지를 담고 sender가 메세지를 브로커에 전달할떄 한쪽이 멈추는 일이 없이 둘다 동시에 동작한다. </br>

## 처리량 관련 주요 속성

batch.size: batch의 사이즈를 나타낸다. 배치에 메시지가 가득차면 sender는 메시지를 전송한다. batch사이즈가 크면 처리량이 높아진다. </br>
linger.ms: sender가 배치에 있는 메시지를 보내고난 이후에 얼마나 기다렸다가 다시 메시지를 전송할지 여부이다. 기본값은 0이다. </br>

## 전송 결과 확인하기

```java
producer.send(ProducerRecord);
```
producer는 기본적으로 전송 결과가 성공했는지 에러가 발생했는지 확인하지는 않는다. </br>
하지만 확인하려면 2가지 방법이 있다. </br>

### 1. Future를 통해서 응답 확인

```java
Future<RecordMetadata> f = producer.send(ProducerRecord);
try {
  f.get(); // note: 블로킹
} catch (Exception e) {
  // note: 재전송 또는 기록
}
```
이 방식은 send()에서 반환되는 Future를 통해서 응답을 확인하는 방식이다. </br>
이 방식을 이용하면 Future의 get()메서드를 사용하면 동기적으로 처리해서 배치에 메시지가 쌓이지 않아서 처리량이 낮아진다. </br>

### 2. Callback을 통한 응답 확인

```java
producer.send(ProducerRecord, new Callback() {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
      if (exception != null) {
        // note: 재전송 또는 기록
      }
      ... 
    }
});
```
send에 callback 객체를 전달하는 방식이다. 
callback은 비동기적으로 실행되기때문에 처리량이 높다. 

## 전송 보장, ACK, `min.insync.replicas` 설정

producer의 ack설정에 따라서 전송 보장 여부를 설정할 수 있다. </br>

ACK - 0 : 전송 보장을 설정하지 않는다. </br>
ACK - 1 : 리더만 메시지를 받아 저장하면 전송 보장 응답을 브로커에게 받는다. 리더가 장애가 생기면 팔로워가 리더로 격상되는데 이때 메세지 유실될 가능성이 있다. </br>
ACK - ALL(-1): 리더와 팔로우(min.insync.replicas)가 모두 메시지를 받으면 전송 보장 응답을 브로커에게 받는다. </br>

`min.insync.replicas`는 브로커의 설정인데 producer의 ack설정이 All일때, 전송 보장 응답을 주기위한 최소 브로커 수(리더, 팔로우)를 나타낸다.

min.insync.replicas의 설정에는 주의사항이 있다. </br>
1. min.insync.replicas 설정이 1이면 리더에만 저장되어도 전송 보장 응답을 주기때문에 ACK - 1과 같은 의미가 된다. </br>
2. min.insync.replicas 설정이 리더와 팔로우 합한 숫자가 동일하면 리더만 장애가 발생해 팔로우가 리더로 격상되면 브로커에 데이터를 저장할 수 없다. </br>

## 에러 유형

카프카는 메세지 전송 전, 전송 과정중에 에러가 발생할 수 있다. </br>

전송 전에 실패 경우
1. 배치 사이즈 초과
2. 메세지 크기 초과
전송 과정에서 실패 경우
1. 타임 아웃
2. 브로커 사이즈 초과


## 실패 대응

1. 재시도
   - 메세지 전송에 실패하면 다시 전송한다. 타임아웃 같은 문제는 재전송으로도 해결이 될 수 있다.
     - 재시도의 문제점
       1. 메세지 중복 전송 가능성(응답이 늦어서 발생할 수 있다.)(enable.idempotence를 사용하면 중복 전송을 줄여준다고 한다.)
       2. 메세지 순서 보장의 이 안되는 문제(먼저 보낸 메시지가 에러로 재전송 된 경우 발생할 수 있다.)
2. 기록
   - 메시지 전송에 실패하면 fild이나 db에 저장하고 나중에 다시 수동 또는 자동으로 처리한다.


출처: https://www.youtube.com/watch?v=geMtm17ofPY&ab_channel=%EC%B5%9C%EB%B2%94%EA%B7%A0
