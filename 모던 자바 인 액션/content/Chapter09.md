## 🌈 Chapter 9 : 리팩터링, 테스팅, 디버깅
<details><summary>정리</summary>
  
```

```

</details>

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
