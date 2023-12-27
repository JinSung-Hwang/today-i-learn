목차 </br>
컨슈머 
1. 컨슈머 실행 코드
2. 토픽 파티션은 컨슈머 그룹 단위 할당
    - 파티션은 컨슈머 그룹과 어떻게 할당이 이루어지는가?
    - 만약 컨슈머 처리량이 떨어지면 무엇도 같이 고려해야하는가?
3. 커밋과 오프셋
   - 컨슈머가 파티션에서 읽은 위치를 기록하는 과정은 무엇인가?
   - 만약 이전 오프셋이 없는 경우는 어떤 경우인가?
   - 오프셋이 없는 경우는 어떻게 auto.offset.reset옵션에 따라서 어떻게 처리되는가?
       - earliest
       - latest
       - none
4. 자동 커밋/ 수동 커밋
   - 자동 커밋/ 수동 커밋은 무엇인가?
   - 자동 커밋은 언제 일어나는가?
5. 수동 커밋의 방식인 동기 커밋과 비동기 커밋
   - 동기 커밋은 무엇인가?
     - 동기 커밋은 커밋 성공 실패를 알 수 있는가?
   - 비동기 커밋은 무엇인가?
     - 비동기 커밋은 커밋 성공 실패를 알 수 있는가?
6. 재처리와 순서
   - 컨슈머는 동일한 메세지를 어떤 경우에 읽을 수 있는가?
   - 멱등성이 무엇인가?
   - 컨슈머를 멱등성을 고려해서 구현해야한다면 어떻게 해야하는가?
7. 세션 타임아웃, 하트비트, 최대 poll 간격
   - 카프카는 컨슈머 숫자를 어떻게 알맞게 조정하는가?
       - 하트비트란?
       - 세션 타임아웃이란? 
       - max.poll.interval.ms란?
8. 컨슈머를 종료
   - 컨슈머를 종료하려면 어떻게 해야하는가?
9. 컨슈머 메서드 사용시 주의 사항
   - 컨슈머는 메서드 사용시 주의 사항이 있는가? poll(), wakeup()

## 컨슈머 실행 코드

```java
Properties prop = new Properties();
prop.put("bootstrap.servers", "localhost:9092");
prop.put("group.id", "group1"); // note: 이 GroupId가 같으면 컨슈머는 같은 컨슈머 그룹에 묶인다.
prop.put("key.deserializer", org.apache.kafka.common.serialization.StringDeserializer");
prop.put("value.deserializer", org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(prop);
consumer.subscribe(Collections.singleton("simple")); // note: 구독할 토픽의 목록을 지정한다. 여기서는 1개만 지정했음
while(조건) {
  ConsumerRecords<String, String> records = consumer.poll(Duration.ofMilis(100));
  for(ConsumerRecord<String, String> record: records) {
    System.out.pringln(record.value() + ":" + record.topic() + ":" + record.partition() + ":" + record.offset()); // note: 컨슈머 레코드를 읽어서 출력한다.
  }
}

consumer.close(); // note: 컨슈머 레코드를 다 처리했으면 컨슈머를 닫아준다.
```

## 토픽 파티션은 컨슈머 그룹 단위 할당

![스크린샷 2023-12-31 오전 1 30 21](https://github.com/JinSung-Hwang/today-i-learn/assets/29647648/ef763f38-9b24-4436-930c-b18a87e29a46)


파티션은 컨슈머 그룹 단위 할당이 이루어진다. </br>
하나의 파티션은 컨슈머 그룹에 하나의 컨슈머에만 할당된다. </br>
하나의 파티션이 컨슈머 그룹에 여러 컨슈머에는 할당될 수 없다. </br>
이렇기 때문에 `파티션`보다 `컨슈머 그룹안에 컨슈머`가 많으면 컨슈머는 놀게 된다. </br>
**만약 처리량이 떨어져서 컨슈머를 늘려야하면 파티션도 같이 늘리려야 하는것을 고려해야한다.** </br>

## 커밋과 오프셋(컨슈머가 파티션에서 읽은 위치를 기록하는 과정)

컨슈머가 파티션에서 메세지를 읽는 과정
1. 컨슈머가 파티션의 이전 오프셋 위치를 찾고 그 오프셋 이후로 메세지를 읽는다. 
2. 컨슈머가 마지막으로 메세지를 읽은 곳에 오프셋 위치를 커밋하여 기록해둔다.
3. 1~2번 과정을 반복하며 메세지를 읽는다.

만약 처음 접근해서 오프셋이 없는 경우, 커밋된 오프셋이 사라진 경우, 컨슈머는 오프셋 위치를 어디로 두고 읽어야할까? </br>
auto.offset.reset을 통해서 관련 설정을 할 수 있다.

- auto.offset.reset = earliest: 파티션 처음부터 메세지를 읽는다.
- auto.offset.reset = latest: 파티션의 마지막을 오프셋으로 사용하여 읽는다. (기본값)
- auto.offset.reset = none: 이전 커밋 offset이 없으면 익셉션이 발생한다. (잘 사용하지는 않음)

## 자통 커밋/ 수동 커밋

enable.auto.commit 
- true: 컨슈머가 읽은 후 자동으로 읽은 위치를 커밋한다. (기본값)
- false: 수동으로 커밋을 지정한다.

auto.commit.interval.ms: 자동 커밋 주기를 지정한다. poll()과 close()시 자동 커밋이 일어난다. (기본값 5000ms) 

## 수동 커밋의 방식인 동기 커밋과 비동기 커밋
동기 커밋 
```java
ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
for (ConsumerRecord<String, String> record: records) {
}
try {
  consumer.commitSync();
} catch(Exception e) {
  // 커밋 실패시 에러 발생
}
```
비동기 커밋
```java
ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
for (COnsumerRecord<String, String> record: records) {
    ... 처리
}
consumer.commitAsync(); // note: commitAsync(OffsetCommitCallback callback) 성공 실패를 알기위해서는 콜백 메서드를 넣어서 확인해야한다.

```

## 재처리와 순서

컨슈머는 동일한 메세지를 아래와 같은 이유로 읽을 가능성이 있다. </br>
- 일시적 커밋 실패 이유로 메세지를 다시 읽는 경우
- 컨슈머 리밸런스(컨슈머가 추가 또는 삭제되어)로 메세지를 다시 읽는 경우
- 그 외의 경우

따라서 컨슈머는 멱등성을 고려해서 구현되어야한다. </br>
멱등성이란 동일한 메세지를 여러번 처리해도 동일한 결과가 나오도록 하는 것을 말한다. </br>
동일한 메세지를 읽어도 처리가 중복되지 않도록 메세지의 일련번호나 타임스탬프등을 활용해서 멱등성이 처리 되도록 해야한다. </br>

## 컨슈머 설정 [컨슈머 조회에 영향을 주는 설정]

fetch.min.bytes: consumer.poll() 메서드 사용지 브로커가 전송하는 최소 데이터 크기를 지정한다. 이 값이 크면 대기시간은 늘지만 처리량도 높아진다. </br>
fetch.max.wait.ms: 최소 데이터 크기가 될때까지 기다리는 한계 시간이다. 이 시간이 지나면 최소 데이터 크기가 되지 않더라도 조회를 해온다. </br>
max.partition.fetch.bytes: 파티션당 브로커 서버가 리턴할 수 있는 최대 크기를 지정한다. 이 값보다 메세지가 커지면 바로 리턴한다. </br>

## 세션 타임아웃, 하트 비트, 최대 poll간격

카프카는 컨슈머 그룹을 알맞게 유지하기 위해서 몇가지 장치가 있다. </br>
브로커는 컨슈머를 하트비트를 통해 컨슈머 상태를 체크하고 응답이 없으면 컨슈머를 그룹에서 뺴고 리밸런스를 진행한다.

- session.timeout.ms: 세션 타임 아웃 시간 </br>
- heartbeat.interval.ms: 하트비트 전송 주기

- max.poll.interval.ms: poll() 메서드의 최대 호출 간격을 지정한다. </br>
  (이 시간 간격 동안 poll()메서드가 호출되지 않으면 컨슈머를 제거하고 리밸런스를 진행한다.)

## 종료 처리

while문의 무한 루프를 벗어나고 싶으면 wakeup()메서드를 호출하면 된다.</br> 
단 같은 스레드에서 wakeup()메서드를 호출 할 순 없고 다른 스레드에서 wakeup()메서드를 호출해야한다.</br>
그러면 poll()메서드에서 exception이 발생하고 while문을 감싸는 try-catch문에서 exception을 처리해주면 된다. </br>

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(prop);
consumer.subscribe(Collections.singleton("simple")); // note: 구독할 토픽의 목록을 지정한다. 여기서는 1개만 지정했음
try {
  while(true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMilis(100)); // note: kafka를 닫고 싶으면 다른 스레드에서 wakeup()메서드를 호출해서 poll()에서 exception이 발생하도록 한다.
    for(ConsumerRecord<String, String> record: records) {
      System.out.pringln(record.value() + ":" + record.topic() + ":" + record.partition() + ":" + record.offset()); // note: 컨슈머 레코드를 읽어서 출력한다.
    }
  }
} catch (Exception ex) {
  ...
} finally {
  consumer.close(); 
}
```

## 주의점

KafkaConsumer는 thread safe하지않다. </br>
따라서 여러 스레드에서 동시에 wakeup()메서드를 제외한 다른 메서드(poll(), ...)을 호출하면 안된다. </br>
단 wakeup()메서드는 다른 스레드에서 호출해야한다. </br>

출처: https://www.youtube.com/watch?v=xqrIDHbGjOY