## 🌈 Chapter 16 : CompletableFuture : 안정적 비동기 프로그래밍
이번 장에서는 사용할 수 있는 여러 다중 처리 리소스(CPU 등)를 고려해    
프로그램이 이들 자원을 고수준으로 효과적으로 활용할 수 있도록 최신 동시성 기법을 살펴봤다.

자바 8, 9 에서는 `CompletableFuture`와 리액티브 프로그래밍 패러다임 두 가지 API를 제공한다.

16장에서는 실용적인 예제를 토해 자바 8에서 제공하는 `CompletableFuture`와 리액티브 프로그래밍의   
역할을 설명하겠다.

## 📚 LESSION 1 : Future의 단순 활용
자바 5부터는 비동기 계산을 모델링하는 데 `Future`를 이용할 수 있으며,    
`Future`는 계산이 끝났을 때 결과에 접근할 수 있는 참조를 제공한다.

시간이 걸릴 수 있는 작업을 `Future`내부로 설정하면 호출자 스레드가 결과를 기다리는 동안    
다른 유용한 작업을 수행할 수 있다.

- `Future`로 오래 걸리는 작업을 비동기적으로 실행하기
```java
ExecutoreServie executor = Executors.newCachedThreadPool();    // 스레드 풀에 태스크를 제출하기위한 ExecutorService
Future<Double> future = executor.submit(new Callable<Double>() {    // Callable을 ExectorService로 제출
  public Double call() {
    return doSomeLongComputation();    // 시간이 오래걸리는 작업은 다른 스레드에서 비동기적으로 실행
  }
});

doSomeThingElse();    // 비동기 작업을 수행하는 동안 다른작업 수행

try{
  Double result = future.get(1, TimeUnit.SECONDS);    // 비동기 작업의 결과를 가져옴
}catch(ExecutionException ee){                        // 결과가 준비되어 있지 않으면 호출 스레드가 블록됨 (최대 1초까지 기다림)
  // 계산 중 예외 발생
}catch(InterruptedException ie){
  // 현재 스레드에서 대기 중 인터럽트 발생
}catch(TimeoutException te){
  // Future가 완료되기 전에 타임아웃 발생
}
```
![image](https://github.com/Songdoeon/Book_Study/assets/96420547/1bcabe52-40a2-44cc-9d4e-628296731575)
이 시나리오에 무슨 문제가 있을까?

바로 오래 걸리는 작업이 영원히 끝나지 않으면 문제가 생긴다.    
그렇기에 위 그림처럼 get메서드를 오버로드해 우리 스레드가 대기할 최대 타임아웃 시간을 설정하는 것이 좋다.

### 🎈 Future 제한
이번 예제에서는 `Future`인터페이스가 비동기 계산이 끝났는지 확인하는 isDone 메서드    
계산이 끝나길 기다리는 메서드, 결과 회수 메서드 등을 제공함을 보여준다.

하지만 이 메서드 만으로는  여러 `Future`의 결과가 있을 때 이들의 의존성을 표현하기 어렵다.

즉, 오래 걸리는 A라는 계산이 끝나면 그 결과를 다른 오래 걸리는 계산 B로 전달하시오.  
그리고 B의 결과가 나오면 다른 질의의 결과와 B의 결과를 조합하시오 같은 요구사항을 구현하기 어렵다.

`Future`로 이러한 동작을 구현하는 것은 쉽지 않다. 다음과 같은 선언형 기능이 있다면 유용할 것이다.

- 두 개의 비동기 계산 결과를 하나로 합친다.
  - 두 가지 계산 결과는 서로 독립적일 수 있다.
  - 두 번째 결과가 첫 번째 결과에 의존하는 상황일 수 있다.
- `Future`집합이 실행하는 모든 태스크의 완료를 기다린다.
- `Future`집합에서 가장 빨리 완료되는 태스크를 기다렸다가 결과를 얻는다
  - 예를 들어 여러 태스크가 다양한 방식으로 같은 결과를 구하는 상황
- 프로그램적으로 `Future`를 완료시킨다.
- `Future`완료 동작에 반응한다
  - 즉 결과를 기다리며 블록되지 않고 결과가 준비되었다는 알림을 받은 후
  - `Future`의 결과로 원하는 추가 동작을 수행할 수 있음

지금까지 설명한 기능을 선언형으로 이용할 수 있도록 자바 8에서 새로 제공하는 `CompletableFuture`클래스를 살펴보자.

stream과 비슷한 패턴 즉, 람다 표현식과 파이프라이닝을 활용하기에    
`Future`와 `CompletableFuture`의 관계를 `Collection`과 `Stream`의 관계에 비유할 수 있다.

### 🎈 CompletableFuture로 비동기 애플리케이션 만들기
이 애플리케이션을 만드는 동안 다음과 같은 기술을 배울 수 있다.

- 고객에게 비동기 API를 제공하는 방법을 배운다.
- 동기 API를 사용해야 할 때 코드를 비블록으로 만드는 방법을 배운다.
  - 두 개의 비동기 동작을 파이프라인으로 만드는 방법
  - 두 개의 동작 결과를 하나의 비동기 계산으로 합치는 방법을 살펴본다.
- 비동기 동작의 완료에 대응하는 방법을 배운다.
  - 모든 상점에서 가격 정보를 얻을 때까지 기다리는 것이 아닌 각 상점에 가격 정보를 얻을 때마다
  - 즉시 최저 가격을 찾는 애플리케이션을 갱신하는 방법을 설명한다.
  - 그렇지 않으면 서버 다운등 문제가 생김

#### 📌 동기 API와 비동기 API
전통적인 **동기 API**에서는 메서드를 호출한 다음 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면    
호출자는 반환된 값으로 계속 다른 동작을 수행한다.

이처럼 동기 API를 사용하는 상황을 블록 호출(blocking call)이라고 한다.

반면 **비동기 API**에서는 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을     
호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당한다.

비와 같은 비동기 API를 사용하는 상황을 비블록 호출(non-blocking call)이라고 한다.

다른 스레드에 할당된 나머지 계싼 결과는 콜백 메서드를 호출해서 전달하거나    
호출자가 '계산 결과가 끝날 때 까지 기다림' 메서드를 추가로 호출하면서 전달된다.

주로 I/O 시스템 프로그래밍에서 이와 같은 방식으로 동작을 수행한다.    
그리고 더 이상 수행할 동작이 없으면 디스크 블록이 메모리로 로딩될 때까지 기다린다.

## 📚 LESSION 2 : 비동기 API 구현
최저가격 검색 애플리케이션을 구현하기 위해 먼저 각각의 상점에서 제공해야하는 API부터 정의하겠다.

- 제품 가격을 반환하는 메서드 정의 코드
```java
public class Shop{
  public double getPrice(String product) {
    // 구현
  }
}

public static void delay()  {
  try{
    Thread.sleep(1000L);
  }catch(InterruptedException e){
    throw new RuntimeException(e);
  }
}
```
getPrice 같은 메서드를 구현하는 것은 오래 걸리기에 delay라는 1초 지연시키는 메서드로 대체하겠다.

### 🎈 동기 메서드를 비동기 메서드로 변환
동기 메서드 getPrice를 비동기로 변환하도록 하겠다.

 Future의 결과값은 핸들일 뿐이며 계산이 완료되면 get 메서드로 결과를 얻을 수 있다.

 getPriceAsync 메서드는 즉시 반환되므로 호출자 스레드는 다른 작업을 수행할 수 있다.
- getPriceAsync 메서드 구현
```java
public Future<Double> getPriceAsynce(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>();    // 계산 결과를 포함할 CompletableFuture 생성
  new Thread( () -> {
              double price = calculatePrice(product);    // 다른 스레드에서 비동기적으로 계산을 수행한다.
              futurePrice.complete(price);              // 오래 걸리는 계산이 완료되면 Future에 값을 설정한다.
  }).start();
  return futurePrice;      // 계산 결과가 완료되길 기다리지 않고 Future를 반환한다.
}
```
