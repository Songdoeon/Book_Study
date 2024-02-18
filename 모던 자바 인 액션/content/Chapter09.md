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

### 🎈 명령형 데이터 처리를 스트림으로 리팩터링 하기
