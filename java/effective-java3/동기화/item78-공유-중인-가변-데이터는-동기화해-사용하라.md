# Item78 공유 중인 가변 데이터는 동기화해 사용하라

> 결론적으로 **여러 스레드간에 공유 중인 가변 데이터(변수, 객체)가 있다면 동기화해서(Synchronized, Volatile, Atomic, 동기화 컬렉션 객체) 사용해야한다. </br>**
> **사실 가장 좋은 방법은 여러 스레드간에 불변 데이터(변수, 객체)만 사용하거나 가변 데이터에는 단일 스레드에서만 쓰도록 하는것이 좋다. </br>**

synchronized키워드는 해당 **메서드나 블록을** 한번에 한 스레드씩 수행하도록 보장하는 키워드이다. </br>
이 synchronized 키워드는 한 스레드가 가변 데이터가 변경 중일때 다른 스레드가 수정중인 가변 데이터를 읽어가는 것을 막아준다. (배타적 수행)</br>
또한 synchronized으로는 한 스레드가 만든 데이터 변화를 다른 스레드에서 바로 확인할 수 있게 한다. (동기화 통신) </br>
synchronized키워드 동작 방식은 어떤 메서드가 가변 데이터에 접근하면 가변 데이터에 락을 걸고 락을 건 메서드가 데이터를 확인하고 필요할 시 수정을 진행한다. </br>
따라서 이 방식은 동기화가 재대로 작동하지만 락을 많이 걸어서 비용이 많이든다.

synchronized를 사용하지 않으면 한 스레드가 데이터 변화하고 다른 스레드에서 변경된 데이터를 바로 확인하지 못할 수 있다. </br>
아래 예를 살펴보자. </br>
```java
public class StopThread {
  private static boolean stopRequested; // note: false로 초기화
  public static void main(String[] args) throw InterruptedException {
    Thread backGroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested) 
        i++;
    });
    
    backGroundThread.start();
    
    TimeUnit.SECONS.sleep(1);
    stopRequested = true;
  }
}
```
main 스레드가 실행하고 backGroundThread는 언제 종료 될거라 생각되는가? </br>
1초후에 backGroundThread가 종료될거라 생각되지만 계속해서 동작한다. 왜그럴까? </br>

main스레드에서 stopRequested를 변경했지만 backGroundThread에서는 main스레드에서 변경한 데이터를 언제 다시 읽어갈지는 보증될 수 없다. </br>
(찾아 보니 CPU cache에 데이터를 계속 읽어와서 변경된 데이터를 언제 가져올지 모른고 synchronized, volatile은 데이터를 읽을때 메모리를 참조해서 데이터를 읽는다고 한다.) </br>
또한 JVM 버전에 따라 아래와 같이 hoisting 최적화를 진행할 수 도 있다.  </br>
```java
// note: 원래 코드
while(!stopRequested)
  i++;

// note: 최적화된 코드
if (!stopRequested)
  while(true)
    i++;
```

기존 코드를 synchronized를 이용해서 수정하면 아래와 같이 수정할 수 있다.
```java
public class StopThread {
  private static boolean stopRequested;
  
  private static synchronized void requestStop() { // note: 읽기 쓰기 메서드에 모두 synchonized 키워드가 붙여있지않으면 동기화가 보장되지 않는다.
    stopRequested = true;
  }
  
  private static synchronized boolean getStopRequested() {
    return stopRequested;
  }
  
  public static void main(String[] args) {
    Thread backGroundThread = new Thread(() -> {
      int i = 0;
      while (!getStopRequested()) {
        i++;
      }
    });
    backGroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
```

----------

synchronized는 매번 락을 걸어 비용이 비싸기에 volatile이라는 키워드를 이용해서 비용을 좀 낮출 수 있다. </br>
volatile 배타적 수행과는 상관이 없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다. (동기화 통신만 지원)  </br>

앞서 말했듯이 이 방식은 배타적 수행하지 않아 사용시 주의가 필요하다.
```java
private static volatile int nextSerialNumber = 0;

public void int generateSerialNumber() {
  return nextSerialNumber++;
}
```
위 코드에서 동시성 문제가 발생한다. </br>
첫번째 스레드에서 nextSerialNumber를 읽어와서 데이터를 수정한다. </br>
데이터를 읽고 수정되는 그 틈사이에 두번째 스레드에서 nextSerialNumber데이터를 읽어서 값을 수정하고 반환한다. </br>
이러면 첫번쨰 두번째 스레드가 같은 데이터를 반환하게 된다. </br>
이렇게 잘못된 결과를 내는 것은 안전 실패(safety failure)라고 부른다. </br>

위 코드 수정하려면 volatile을 제거하고 메서드에 synchonized를 붙여 수정할 수도 있고 아니면 아래와 같이 Atomic패키지를 사용할 수 도 있다. </br>
```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
  return nextSerialNum.getAndIncremt();
}
```
이렇게 하면 synchronized처럼 lock을 사용하지 않고도 동일한 결과를 얻을 수 있다.

하지만 이 위의 모든 문제를 피하는것이 가장 좋다. 
따라서 가변 데이터를 여러 스레드에서 공유하지말고 가변 데이터는 단일 스레드에서만 쓰는것이 좋다.

> 결론적으로 **여러 스레드간에 공유 중인 가변 데이터(변수, 객체)가 있다면 동기화해서(synchronized, Volatile, Atomic, 동기화 컬렉션 객체) 사용해야한다. </br>**
> **사실 가장 좋은 방법은 여러 스레드간에 불변 데이터(변수, 객체)만 사용하거나 가변 데이터에는 단일 스레드에서만 쓰도록 하는것이 좋다. </br>**
