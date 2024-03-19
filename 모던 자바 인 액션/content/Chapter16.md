## 🌈 Chapter 16 : CompletableFuture : 안정적 비동기 프로그래밍
<details><summary>정리</summary>

```
- 한 개 이상의 원격 외부 서비스를 사용하는 긴 동작을 실행할 때는 비동기 방식으로 애플리케이션의 성능과 반응성을 향상시킬 수 있다.
- 우리 고객에서 비동기 API를 제공하는 것을 고려해야 한다. `CompletableFuture`의 기능을 이용하면 쉽게 비동기 API를 구현할 수 있다.
- `CompletableFuture`를 이용할 때 비동기 태스크에서 발생한 에러를 관리하고 전달할 수 있다.
- 동기 API를 `CompletableFuture`로 감싸서 비동기적으로 소비할 수 있다.
- 서로 독립적인 비동기 동작이든 아니면 하나의 비동기 동작이 다른 비동기 동작의 결과에 의존하는 상황이든 여러 비동기 동작을 조립하고 조합할 수 있다.
- `CompletableFuture`에 콜백을 등록해서 `Future`가 동작을 끝내고 결과를 생산했을 때 어떤 코드를 실행하도록 지정할 수 있다.
- `CompletableFuture` 리스트의 모든 값이 완료될 때까지 기다릴지 아니면 첫 값만 완료되길 기다릴지 선택할 수 있다.
- Java 9에선 `orTimeout`, `completeOnTimeout` 메서드로 `CompletableFuture`에 비동기 타임아웃 기능을 추가했다.
```

</details>

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

이 에 무슨 문제가 있을까?

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
동기 메서드를 `CompletableFuture`를 통해 비동기 메서드로 변활할 수 있다.   
비동기 계산과 완료 결과를 포함하는 `CompletableFuture`인스턴스를 만들고 완료 결과를 `complete`메서드로 전달하여 종료할 수 있다.


### 🎈 에러 처리 방법
비동기 작업도중 발생한 예외에는 해당 스레드에만 영향을 미친다.   
즉, 에러가 발생해도 전체적인 일은 계속 진행되며 순서가 꼬이게 된다.

그러므로 클라이언트는 타임아웃 값을 받는 get메서드를 구현함으로 이 문제를 해결할 수 있다.
```java
public Future<Integer> getPriceAsync(String product) {
	  CompletableFuture<Integer> futurePrice = new CompletableFuture<>();
	  new Thread(() -> {
        try {
            int price = calculatePrice(product);
		  	    futurePrice.complete(price);    // 정삭적으로 종료될시 Future에 정보를 저장한채 Future를 종료한다.
        } catch (Exception ex) {
            futurePrice.completeExceptionally(ex); // 도중에 문제가 발생하면 발생한 에러를 포함시켜 Future를 종료한다.
        }
	  }).start();
	  return futurePrice;
} 
```
## 📚 LESSION 3 : 비블럭 코드 만들기
병렬스트림과 `CompletableFuture`를 사용하면 비블록 코드를 만들 수 있다.

두 가지 버전 모두 내부적으로 `Runtime.getRuntime().availableProcessors()`가 반환하는 스레드 수를 사용하면서   
비슷한 결과가 나온다.

결과적으로는 비슷하지믄 `CompletableFuture`는 병렬 스트림 버전에 비해 작업에 이용할 수 있는 다양한   
`Executor`를 지정할 수 있다는 장점이 있다.

따라서 `Executor`로 스레드 풀의 크기를 조절하는 등 애플리케이션에 맞는 최적화된 설정을 만들 수 있다.

### 📌 커스텀 Executor 사용하기
자 그럼 `Executor`로 우리 애플리케이션이 실제로 필요한 작업량을 고려한 스레드 풀을 관리해보자.

스레드 풀은 너무 크면 CPU와 메모리 자원을 서로 경쟁하느라 시간을 낭비할 수 있다.    
반면 스레드 풀이 너무 작으면 CPU의 일부 코어는 활용되지 않을 수 있다. 

게츠는 다음 공식으로 대략적인 CPU 활용 비율을 계산할 수 있다고 제안한다.

<img width="616" alt="스크린샷 2024-03-19 오후 9 17 33" src="https://github.com/Songdoeon/Book_Study/assets/96420547/36bf1d52-b0a2-417a-93b3-1aa4f81961dd">

## 📚 LESSION 4 : 비동기 작업 파이프라인 만들기
Stream API의 `map` 메서드와 `CompletableFuture`의 메서드들을 이용하여 비동기 작업 파이프라인을 만들 수 있다.

- `supplyAsync`: 전달받은 람다 표현식을 비동기적으로 수행한다.
- `thenApply`: `CompletableFuture`가 동작을 완전히 완료한 다음에 `thenApply`로 전달된 람다 표현식을 적용한다.
- `thenCompose`: 첫 번째 연산의 결과를 두 번째 연산으로 전달한다.
- `thenCombine`: 독립적으로 실행된 두 개의 `CompletableFuture` 결과를 이용하여 연산한다. 두 결과를 어떻게 합칠지 정의된 `BiFunction`을 두 번째 인수로 받는다.
- `thenCombineAsync`: 두 개의 `CompletableFuture` 결과를 반환하는 새로운 `Future`를 반환한다.

```java
Future<Double> futurePriceInUSD = 
    CompletableFuture.supplyAsync(() -> shop.getPrice(product))
                    .thenCombine(
                        CompletableFuture.supplyAsync(
                          () -> exchangeService.getRate(Money.EUR, Money.USD)),
                          (price, rate) -> price * rate
                    ));
```

Java 9에선 다음 메서드들이 추가되었다.

- `orTimeout`: 지정된 시간이 지난 후 `CompletableFuture`를 `TimeoutException`으로 완료하게 한다.
- `completeOnTimeout`: 지정된 시간이 지난 후 지정한 기본 값을 이용해 연산을 이어가게한다.

## CompletableFuture의 종료에 대응하는 방법
실제 원격 서비스들은 얼마나 지연될지 예측하기 어렵다.

- `thenAccept`: `CompletableFuture`가 생성한 결과를 어떻게 소비할 지 미리 지정한다.
- `allOf`: 전달받은 `CompletableFuture` 배열이 모두 완료될 때 `CompletableFuture`를 반환한다.
- `anyOf`: 전달받은 `CompletableFuture` 배열 중 하나라도 작업이 끝났을 때 완료한 `CompletableFuture`를 반환한다.
