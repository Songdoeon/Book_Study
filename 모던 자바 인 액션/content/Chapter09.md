## 🌈 Chapter 9 : 리팩터링, 테스팅, 디버깅

우리는 람다와 스트림 API를 배웠지만, 모든 개발환경이 람다와 스트림으로 구성되어 있는것은 아니다.   
그리고 새 프로젝트라 하더라도 보통 예전 코드들을 기반으로 시작한다.

이 장에서는 기존 코드를 가독성과 유연성을 높이는 방식으로 리팩터링하는 방법을 설명할것이다.

그리고 람다 표현식으로 짜는 다양향 객제치쟝 디자인 패턴을 간소화 하는 방법도 살펴본다.   
마지막으로는 람다 표현식과 스트림 API를 사용하는 코드를 테스트하고 디버깅하는 방법을 설명한다.

## 📚 LESSEN 1 : 가독성과 유연성을 개선하는 리팩터링

이번 절에서는 지금까지 배운 람다, 메서드 참조, 스트림 등의 기능을 이용해서    
더 가독성이 좋고 유연한 코드로 **리팩터링**하는 방법을 설명한다.

### 🎈 코드 가독성 개선
코드 가독성이란 무엇일까?

코드 가독성이 좋다는 것은 추상적이고 개인차가 있는 의견이라고 생각한다.   
하지만 보통은 다른사람도 쉽게 이해할 수 있는 코드를 보통 가독성이 좋다라고 표현한다.

코드 가독성을 높이기 위해선 코드의 문서화를 잘하고, 표준 코딩 규칙이나 개발 무리간의 컨벤션을 잘 지키는 것에 노력해야한다.

자바 8의 새 기능들은 코드 가독성을 높일 수 있다.   
이것들을 이용해 리팩토링 예제를 소개하도록 하겠다.

- 익명 클래스를 람다로 리팩터링하기
- 람다 표현식을 메서드 참조로 리팩터링하기
- 명령형 데이터 처리를 스트림으로 리팩터링하기

### 🎈 익명 클래스를 람다로 리팩터링하기
함수형 인터페이스를 구현하는 익명 클래스는 람다 표현식으로 리팩터링 가능하다.

하지만 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다.

- 익명 클래스에서 사용한 `this`, `super`는 람다 표현식에서 다른 의미를 갖는다.
  - 익명 클래스에서 `this`는 익명 클래스 자신을 가리킨다.
  - 하지만 람다에서 `this`는 람다를 감싸는 클래스를 가리킨다.
- 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다.(Shadow variable)
  - 하지만 다음 예제 처럼 람다  표현식으로는 변수를 가릴 수 없다.
- 변수를 가리는 코드
```java
int a = 10;
Runnable r1 = () -> {
  int a = 2;      // 컴파일 에러
  System.out.println(a);
};

Runnable r2 = new Runnable(){
  public void run() {
    int a = 2;      // 잘 작동한다.
    System.out.println(a);
  }
};
```

- 람다로 변환시 Context Overloading에 따른 모호함이 초래될 수 있다.
  - 익명 클래스는 인스턴스화할 때 명시적으로 형식이 정해진다.
  - 하지만 람다의 형식은 Context에 따라 달라진다.
- 예제 코드
```java
interface Task {
  pulbic void execute();
}

public static void doSomething(Runnable r) { r.run(); }
public static void doSomething(Task t) { r.execute(); }

// 태스크를 구현하는 익명클래스
doSomething(new Task()) {
  public void execute() {
    System.out.println("Danger danger!!");
  }
});

// doSomething 메서드
doSomething(() -> System.out.println("Danger danger!!"));
```

위 상황에서 익명클래스를 람다 표현식으로 바꾸면 메서드 호출 시    
`Runnable`과 `Task` 모두 대상 형식이 될 수 있으므로 문제가 생긴다.

즉 `doSomething`의 대상이 모호해진다는 것이다.

그래서 명시적 형변환을 통해 모호함을 제거할 수 있다.
- 명시적 형변환
```java
doSomething((Task)() -> System.out.println("Danger danger!!"));
```

보통의 IDE에서 리팩터링 기능을 사용하면 이러한 문제는 자동으로 해결된다.

### 🎈 람다 표현식을 메서드 참조로 리팩터링하기
람다 표현식은 쉽게 전달할 수 있는 짧은 코드다.

하지만 람다식 대신 메서드 참조를 이용하면 가독성을 높일 수 있다.   
메서드 명으로 코드의 의도를 명확하게 알릴 수 있기 때문이다.

- 메서드 명을 활용하는 방법
```java
inventory.sort(a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));  // 비교 구현에 신경써야한다.
inventory.sort(comapring(Apple::getWeight));    // 코드 자체가 문제를 설명한다.
```

`sum`, `maximum` 등 자주사용하는 리듀싱 연산은 메서드 참조와 함께 사용할 수 있는 내장 헬퍼 메서드를 제공한다.

다음과 같은 예제들도 알아두자.

- 내장 컬렉터를 이용하는 코드
```java
int totalCalories = menu.stream().map(Dish::getCalories)
                                 .reduce(0, (c1, c2) -> c1 + c2);

// 내장 컬렉터를 이용한 코드
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

### 🎈 명령형 데이터 처리를 스트림으로 리팩터링 하기
이론적으로는 반복자를 이용한 기존의 모든 컬렉션 처리 코드를 스트림 API로 바꿔야 한다.

왜나하면 스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다.

스트림은 쇼트서킷, Lazy라는 강력한 최적화와 멀티코어 아키텍처를 활용할 수 있는 좋은 방법이다.    

예를 들어 다음 코드는 두 가지 패턴(필터링과 추출)으로 엉킨 코드다.   
이 코드를 접한 개발자는 전체 구현을 살펴본 이후에야 전체 코드의 의도를 이해할 수 있다. 게다가 이 코드를 병렬처리하는 것은 매우 어렵다.

- 필터링과 추출을 이용하는 코드
```java
// 엉킨 코드
List<String dishNames = new ArrayList<>();

for(Dish dish : menu){
  if(dish.getCalories() > 300) {
    dishNames.add(dish.getName());
  }
}

// 스트림으로 간결화되고 병렬처리하는 코드
menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Diush::getName)
    .collect(toList());
```

명령형 코드의 `break`, `continue`, `return` 등의 제어 흐름문을 모두 분석해서 같은 기능을 수행하는 스트림 연산으로 유추해야 하므로,   
명령형 코드를 스트림 API로 바꾸는 것은 쉬운 일이 아니다.

다행히도 명령형 코드를 스트림 API로 바꾸도록 도움을 주는 몇 가지 도구들도 있긴하다.

### 🎈 코드 유연성 개선성
람다를 이용해 다양한 동작을 표현할 수 있고, 변화하는 요구사항에 유연하게 대응하는 코드를 구현할 수 있게되었다.

#### 📕 함수형 인터페이스 적용
람다를 이용하려면 함수형 인터페이스가 필요하다.

이번에는 조건부 연기 실행(conditional deferred execution)과 실행 어라운드(execute around)   
즉 두 가지 자주 사용하는 패턴으로 람다 리팩토링을 살펴본다.

#### 📕 조건부 연기 실행
실제 작업을 처리하는 코드 내부에 제어 흐름문이 복잡하게 얽힌 코드를 흔히 볼 수 있다.

흔히 보안 검사나 로깅 관련 코드가 이처럼 사용된다.

- 자바 Logger 클래스
```java
// 기존 클래스
if(logger.isLoggable(Log.FINER)) {
  logger.finer("Problem: " + generateDiagnostic());
}

//바람직한 코드
logger.log(Level.FINER, "Problem: " + generateDiagnostic());
```

위 코드의 설명은 다음과 같다.

- logger의 상태가 isLoggable이라는 메서드에 의해 클라이언트 코드로 노출된다.
- 메시지를 로깅할 때마다 logger 객체의 상태를 매번 확인해야 할까?
  - 이들은 코드를 어지럽힌다.
- 메시지 로깅하기 전에 logger 객체가 적절한 수준으로 설정되었는지 내부적으로 확인하는 log 클래스를 사용하도록 하자.
  - 불필요한 if문을 제거할 수 있음
  - logger의 상태를 노출할 필요가 없음

하지만 아직 모든 문제가 해결된 것은 아니다.

인수로 전달된 메시지 수준에서 logger가 활성화되어 있지 않더라도 항상 로깅 메시지를 평가하게 된다.    
이는 람다를 이용해 쉽게 해결할 수 있다.

특정 조건에서만 메시지가 생성될 수 있도록 생성 과정을 연기할 수 있어야 한다.

자바 8 부터는 Supllier를 인수로 갖는 오버로르된 Log 메서드를 제공한다.

- 새로 추가된 log 메서드 시그니처
```java
public void log(Level levelm Supplier<String> msgSupplier)

// 호출법
logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic());

// log 메서드 내부 구현 코드
public void log(Level levem Supplier<String> msgSupplier) {
  if(logger.isLoggable(level)){
    log(level, msgSupplier.get());    // 람다 실행
  }
}
```
이 기법으로 어떤 문제를 해결할 수 있을까?

만일 클라이언트 코드에서 객체 상태를 자주 확인하거나, 객체의 일부 메서드를 호출하는 상황이라면    
내부적으로 객체의 상태를 확인한 다음 메서드를 호출하도록 새로운 메서드를 구현하는 것이 좋다.

그러면 가독성도 좋아지고 캡슐화도 강화된다.

#### 📕 실행 어라운드
매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드가 있다면 이를 람다로 변환할 수 있다.

준비, 종료 과정을 처리하는 로직을 재사용함으로써 코드 중복을 줄일 수 있다는 것이다.

- 람다를 이용한 다양한 방식으로의 파일 처리
```java
String oneLine =
            processFile((BufferedReader b) -> b.readLine());    // 람다 전달

String twoLines =
            processFile((BufferedReader b) -> b.readLine() + b.readLine());    // 다른 람다 전달

prublic static String processFile(BufferedReaderProcessor p) throws IOException {
  try(BufferedREader br = new BufferedReader(new FileReader("ModernJavaInAction/chap9/data.txt"))){
    return p.process(br);    // 인수로 전달된 BufferedReaderProcessor 실행
  }
}

public interface BufferedReaderProcessor {
  String process(BufferedReader b) throws IOException;
}
```

## 📚 LESSEN 2 : 람다로 객체지향 디자인 패턴 리팩터링하기
다양한 패턴을 유형별로 정리한 것이 디자인 패턴이다.

디자인 패턴은 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는, 검증된 청사진을 제공한다.   
재사용할 수 있는 부품으로 여러 가지 다리를 건설하는 엔지니어링에 비유할 수도 있겠다.

예를 들어 구조체와 공작하는 알고리즘을 서로 분리하고 싶을 때 방문자 디자인 패턴을 사용할 수 있다.

디자인 패턴에 람다식이 더해지면 색다른 기능을 발휘할 수 있다.   
즉, 람다를 이용해 디자인 패턴을 리팩터링 하는것이다.

우리는 대략 다음 다섯가지의 패턴을 살펴볼 것이다.   
하지만 자세한 설명은 생략하도록 하겠다.

- 전략
- 템플릿 메서드
- 옵저버
- 의무 체인
- 팩토리

### 🎈 전략 패턴
전략 패턴은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이다.

2장에서 다양한 Predicate로 목록을 필터링 하는것이 그 예제이다.

다양한 기준을 갖는 입력값을 검증하거나, 다양한 파싱 방법, 입력 형식을 설정하는 등 다양한 시나리오에 활용할 수 있다.

### 🎈 템플릿 메서드
알고리즘의 개요를 제시한 다음에 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 템플릿 메서드 디자인을 사용한다.

다시 말해, 알고리즘을 조금 수정한 버전이 필요한 상황에 적합하다는 말이다.

### 🎈 옵저버
어떤 이벤트가 발생했을 때 한 **객체(주체)** 가 다른 **객체 리스트(옵저버)** 에 자동으로 알림을 보내야 하는 상황에서 옵저버 디자인 패턴을 사용한다.

### 🎈 의무 체인
작업 처리 객체의 체인(동작 체인)을 만들 때는 의무 체인 패턴을 사용한다.

한 객체가 어떤 작업을 처리한 다음 다른 객체로 결과를 전달, 다른 객체도 해야 할 작업을 처리한 후 다른 객체로 전달하는 식이다.

### 🎈 팩토리
인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.

## 📚 LESSEN 3 : 람다 테스팅
이제 람다식으로 코드를 짜는 법은 익숙해졌으리라 믿는다.

그럼 이 코드가 제대로 작동하는지 테스팅을 진행해봐야지 않겠는가?

간단한 테스팅의 경우는 생략하도록 하겠다.

### 🎈 복잡한 람다를 개별 메서드로 분할하기
우리는 곧 많은 로직을 포함하는 복잡한 람다식을 접하게 될 것이다.

그런데 테스트 코드에서 람다식을 참조할 수 없는데, 복잡한 람다식을 어떻게 테스트를 할 것인가?

해결책은 람다 표현식을 메서드 참조로 바꿔 일반 메서드를 테스트 하듯 테스트 하는것이다.

### 🎈 고차원 함수 테스팅
함수를 인수로 받거나 다른 함수를 반환하는 메서드(고차원 함수)는 좀 더 사용하기 어렵다.

메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트할 수 있다.

- 다양한 프레디케이트로 filter 메서드 테스트 코드
```java
@Test
public void testFilter() throws Exception{
  List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
  List<Integer> even = filter(numbers, i -> i % 2 == 0);
  List<Integer> smallerThanThree = filter(numbers, i -> i < 3)
  assertEquals(Arrays.asList(2, 4), even);
  assertEquals(Arrays.asList(1, 2), smallerThanThree);
}
```

테스트해야 할 메서드가 다른 함수를 반환한다면 어떻게 해야 할까?

이때는 `Comparator`에서 살펴본 함수형 인터페이스의 인스턴스로 간주하고 함수의 동작을 테스트 할 수 있다.

## 📚 LESSEN 4 : 디버깅
문제가 발생한 코드를 디버깅할 때 개발자는 다음 두가지를 가장 먼저 확인해야 한다.

- 스택 트레이스
- 로깅

하지만 람다식과 스트림은 기존의 디버깅 기법을 무력화 한다.

우리는 람다와 스트림 디버깅 방법을 살펴보겠다.

### 🎈 스택 트레이스 확인
예외 발생등 으로 프로그램이 멈추면 그것에 대한 정보는 **스택 프레임**에서 확인할 수 있다.

우리는 **스택 프레임**에서 다음과 같은 정보를 알 수 있다.

- 프로그램에서의 메서드 호출 위치
- 호출할 때의 인수값
- 호출된 메서드의 지역 변수 등을 포함한 호출 정보

따라서 프로그램이 멈췄다면 멈춘 이유에 대한 정보를 프레임별로 보여주는 **스택 트레이스**를 얻을 수 있다.

### 🎈 람다와 스택 트레이스
람다식은 이름이 없기 때문에 복잡한 스택 트레이스가 생성된다.

- 람다의 오류 예제
```java
at Debugging.lambda$main$0(Debugging.java:6)  // $0 의 정보를 해석할 수 없음
```
- 일반적인 메서드 오류
```java
at Debugging.divideByZero(Debugging.java:6)   // 메서드 명으로 확인 가능
```

따라서 람다식과 관련한 스택 트레이스는 이해하기 어려울 수 있다.

### 🎈 정보 로깅
스트림의 파이프라인 연산을 디버깅한다고 가정하자.

다음과 같이 `peek`연산을 이용하여 확인할 수 있다.

- 스트림 디버깅 코드
```java
List<Integer> numbers = Arrays.asList(2, 3, 4, 5);

numbers.stream()
        .peek(x -> System.out.println("from stream: " + x);
        .map(x -> x + 17)
        .peek(x -> System.out.println("after map: " + x);
        .filter(x -> x % 2 == 0)
        .peek(x -> System.out.println("after filter: " + x);
        .limit(3)
        .peek(x -> System.out.println("after limit: " + x);
        .collect(toList());

// 출력
from stream: 2
after map: 19
from stream: 3
after map: 20
after filter: 20
after limit: 20
from stream: 4
after map: 21
from stream: 5
after map: 22
after filter: 22
after limit: 22
```
