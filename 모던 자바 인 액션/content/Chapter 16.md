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
