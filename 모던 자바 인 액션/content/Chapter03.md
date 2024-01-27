## 🌈 Chapter 3 : 람다 표현식
우리는 동작 파라미터화를 이용해 유연하고 재사용성 좋은 코드를 만들 수 있게 되었다.

하지만 익명 클래스의 사용으로도 만족할 만큼 깔끔한 코드는 나오지 않았고,   
그 점을 람다를 이용한 방식으로 극복했다.

이번 장에서는 람다 표현식을 어떻게 만드는지, 어떻게 사용하는지, 어떻게 코드를 간결하게 만드는지에 대해 설명하겠다.

## 📚 LESSEN 1 : 람다란 무엇인가?
람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화 한것이라고 할 수 있다. 

### 🎈 람다 표현식
람다의 특징은 다음과 같다.

- 익명
  - 보통의 메서드와 달리 이름이 없다.
  - 구현해야 할 코드에 대한 걱정거리가 줄어든다.
- 함수
  - 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다.
  - 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 예외 리스트는 가정할 수 있다.
- 전달
  - 람다는 메서드를 인수로 전달하거나 변수로 저장할 수 있다.
- 간결성
  - 익명 클래스처럼 자질구레한 코드를 구현할 필요가 없다.

람다 표현식은 다음 세 부분으로 이루어진다.

<img width="500" alt="image" src="https://github.com/Songdoeon/Book_Study/assets/96420547/b27f328a-20e1-41b2-8ac4-fea0eee27be6">

- 파라미터 리스트
  - Comparator의 comapre 메서드 파라미터(사과 두 개)
- 화살표
  - -> 표시는 람다의 파라미터 리스트와 바디를 구분한다.
- 람다 바디
  - 두 사과의 무게를 비교한다. 람다의 반환값에 해당하는 표현식이다.



#### 📕 람다 표현식 기본 예제

- 표현식 스타일 람다라고 알려진 람다의 기본 문법
```
(parameters) -> expression
```
- 블록 스타일
```
(parameters) -> {statements;}
```

다음은 자바8에서 지원하는 다섯가지 람다 예제다.

```java
(String s) -> s.length()             // Function

(Apple a) -> a.getWeight() > 150     // Prediacte

(int x, int y) -> {                  // Consumer
  System.out.println("Result : ");
  System.out.println(x + y); 
}

() -> 42                            // Supplier

(Apple a1, Apple a2) -> a1.getWeight() - a2.getWeight() // Function
```

주석에 있는 단어에 대해서는 밑에서 다시 한번 다루도록 하겠다.




## 📚 LESSEN 2 : 어디에, 어떻게 람다를 사용할까?
람다의 사용법에 대해 알아보기 전에 우리는 함수형 인터페이스에 대한 이해가 먼저 필요하다.

### 🎈 함수형 인터페이스
함수형 인터페이스란 오직 하나의 추상 메서드를 가지고 있는 인터페이스를 말한다.

지금까지 살펴본 함수형 인터페이스는 대표적으로 Comparator, Runnable 등이 있었다.

#### 📕 함수형 인터페이스 정의
자바의 Functional interface 에는 다음과 같은 표준이 있다.

|종류|parameter|return|method|비고|
|--|--|--|--|--|
|java.lang.Runnable|X|X|void run()|Thread 용도|
|Consumer 계열|O|X|void accept(T)|- 소비자|
|Supplier 계열|X|O|T get()|- 공급자|
|Funcion 계열|O|O|T apply(R)|- 파라미터를 새로운 타입으로 변환하여 리턴|
|Operation 계열|O|O|T XXX(T)|- 파라미터를 동일한 타입으로 리턴|
|Prediacte 계열|O|boolean|boolean test(T)|파라미터를 조사하여 리턴을 결정|

그렇다면 이 함수형 인터페이스로 우리는 무엇을 할 수 있을까?

람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로,   
전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.

다음 예제를 살펴보며 이해를 해보자.
Runnable 은 run이라는 메서드 하나만을 정의하므로 함수형 인터페이스의 올바른 코드다.

```java
public static void process(Runnable r){
  r.run();
}

Runnable r1 = () -> System.out.println("Hello World 1");

Runnable r2 = new Runnable(){
  public void run(){
    System.out.println("Hello World 12);
  }
};

process(r1);   //Hello World 1 출력
process(r2);   //Hello World 2 출력
process(() -> System.out.println("Hello World 3");  //Hello World 3 출력
```

### 🎈 함수 디스크립터(Function Descriptor)
함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다.

람다 표현식의 시그니처를 서술하는 메서드를 함수 디스크립터라고 부른다.  

예를 들어 Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로,   
Runnable 인터페이스는 `() -> void`로 표현하며 인수와 반환값이 없는 시그니처로 생각할 수 있다.

더 다양한 종류의 함수 디스크립터는 뒤에서 소개하도록 하겠다.

하지만 왜 함수형 인터페이스를 인수로 받는 메서드만 람다를 표현식으로 사용할 수 있는지 의문을 가질 수 있다.

이는 언어 설계자들이 자바에 함수 형식을 추가하는 방법을 고려했지만,   
언어를 더 복잡하게 만들지 않는 현재 방법을 선택했고, 또한 대부분의 자바 프로그래머가 하나의 추상메서드를 갖는   
인터페이스에 이미 익숙하다는 점도 고려했다.

#### 📌 람다와 메소드 호출
```
process(() -> System.out.println("This is awesome"));
```
위 코드가 어때 보이는가?

우리는 표현식이 아닌 방식은 중괄호가 필요하다고 했다.

하지만 자바 언어 명세에서 void 반환 메소드 호출에 특별한 규칙을 정하고 있어   
하나의 void 메소드는 중괄호로 감쌀 필요가 없다.

## 📚 LESSEN 3 : 람다 활용 : 실행 어라운드 패턴
