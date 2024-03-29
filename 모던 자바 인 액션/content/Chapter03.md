## 🌈 Chapter 3 : 람다 표현식
<details><summary>정리</summary>

  
- 람다 표현식은 익명 함수의 일종이다. 이름은 없지만, 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다.
- 람다 표현식으로 간결한 코드를 구현할 수 있다.
- 함수형 인터페이스는 하나의 추상 메서드만을 정의하는 인터페이스다.
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다.
- 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공할 수 있으며 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.
- java.util.function 패키지는 Predicate<T>, Function<T, R>, Supplier<T>, Consumer<T>, BinaryOperator<T> 등을 포함해서 자주 사용하는 다양한 함수형 인터페이스를 제공한다.
- Java 8은 Predicate<T>와 Function<T, R> 같은 제네릭 함수형 인터페이스와 관련한 박싱 동작을 피할 수 있는 IntPredicate, IntToLongFunction 등과 같은 기본형 특화 인터페이스도 제공한다.
- 실행 어라운드 패턴(예를 들어 자원 할당, 자원 정리 등 코드 중간에 실행해야 하는 메서드에 꼭 필요한 코드)을 람다와 활용하면 유연성과 재사용성을 추가로 얻을 수 있다.
- 람다 표현식의 기대형식(type expected)을 대상 형식(target type)이라고 한다.
- 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.
- Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다.
    
</details>
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
|Prediacte 계열|O|boolean|boolean test(T)|- 파라미터를 조사하여 리턴을 결정|

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
람다와 동작 파라미터화로 유연하고 간결한 코드를 구현하는 데 도움을 주는 실용적인 예제를 살펴보자.

<img width="500" alt="image" src="https://github.com/Songdoeon/Book_Study/assets/96420547/85091c16-076a-4b46-9a41-1a92f4d984c5">

- 자원 처리에 사용하는 순환패턴은 자원을 열고, 처리한 다음 자원을 닫는 순서로 이루어진다.
  - 설정과 정리과정은 비슷하다.
- 즉 실제 자원 처리 코드를 설정과 정리 두 과정이 둘러 싸는 형태를 갖는다.
- 위 그림과 같은 형식의 코드를 **실행 어라운드 패턴**이라 부른다.

다음 예제는 파일의 한 행을 읽는 코드다.
```java
public sTring processFile() throws IOException{
  try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
    return br.readLine();    // 실제 필요한 작업을 하는 행
  }
}
```

### 🎈 1단계 : 동작 파라미터화를 기억하라.
현재 코드는 파일에서 한 번에 한 줄만 읽을 수 있다.

한 번에 두 줄을 읽거나 가장 자주 사용되는 단어를 반환하려면 어떻게 해야 할까?   
기존의 설정, 정리 과정은 재사용하고, processFile 메서드만 다른 동작을 수행하도록 한다면 좋을 것이다.

자 그러면 processFile의 동작을 파라미터화 해보도록 하자.

BufferedReader를 이용해 다른 동작을 수행할 수 있도록 processFile 메서드로 동작을 전달해야 한다.

### 🎈 2단계 : 함수형 인터페이스를 이용해서 동작 전달
함수형 인터페이스 자리에 람다를 사용할 수 있다.

따라서 BufferedReader -> String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다.

이 인터페이스를 BufferedReaderProcessor라고 하겠다.

```java
@FunctinalInterface
public interface BufferedReaderProcessor{
  String process(BufferedReader b) throws IOException;
}

public String processFile(BufferedReaderProcessor p) throws IOexception{
  ...
}
```
위와 같은 식으로 정의한 인터페이스를 processFile 메서드의 인수로 전달할 수 있다.

### 🎈 3단계 : 동작 실행
이제 BufferedReaderProcessor에 정의된 process 메서드의 시그니처와 일치하는 람다를 전달할 수 있다.
  > 메서드 시그니처 : BufferedReader -> String

```java
public String processFile(BufferedReaderProcessor p) throws IOexception{
   try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
    return p.process(br)
  }
}
```
### 🎈 4단계 : 람다 전달
이제 람다를 이용해 다양한 동작을 processFile 메서드로 전달해보자.

```java
String onLine = processFile(BufferedReader br) -> br.readLine());  // 한 행을 처리하는 코드
String twoLine = processFile(BufferedReader br) -> br.readLine() + br.readLine());    // 두 행을 처리하는 코드
```

다음 절에서는 다양한 람다를 전달하는 데 재활용할 수 있도록 자바8에 추가된 새로운 인터페이스를 살펴보겠다.

## 📚 LESSEN 4 : 함수형 인터페이스 사용
함수형 인터페이스의 추상 메서드는 람다 표현식의 시그니처를 묘사하며, 함수 디스크립터 라고 한다.

다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다. 

다음 표로 대부분의 설명을 대체하도록 하겠다.

|종류|parameter|return|method|비고|
|--|--|--|--|--|
|java.lang.Runnable|X|X|void run()|Thread 용도|
|Consumer 계열|O|X|void accept(T)|- 소비자|
|Supplier 계열|X|O|T get()|- 공급자|
|Funcion 계열|O|O|T apply(R)|- 파라미터를 새로운 타입으로 변환하여 리턴|
|Operation 계열|O|O|T XXX(T)|- 파라미터를 동일한 타입으로 리턴|
|Prediacte 계열|O|boolean|boolean test(T)|- 파라미터를 조사하여 리턴을 결정|

### 🎈 기본형 특화
함수형 인터페이스에는 특화된 형식의 함수형 인터페이스도 있다.

- 자바의 데이터는 참조형과 기본형 두가지로 나뉜다.
- 하지만 제네릭 파라미터에는 참조형만 사용할 수 있다.
  - 제네릭 내부 구현 때문이다.
- 자바는 기본형과 참조형을 변환하는 박싱 언박싱 기능이 존재함.
- 하지만 변환 과정은 비용이 소모됨
- 박싱한 값는 메모리를 더 소비하며 기본형을 가져올 때도 메모리 탐색이 필요함

때문에 자바 8에서는 기본형을 입출력으로 사용하는 상황에서,  
오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.

다음은 `intPredicate` 예제다.

```java
public interface IntPredicate{
  booelan test(int t);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000);    // 참(박싱 없음)

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 == 0;
oddNumbers.test(1000);    // 거짓(박싱)
```

- 일반적으로 특정 형식을 입력으로 받는 함수형 인터페이스의 이름 앞에는 형식명이 붙는다.
  - `DoublePredicate`, `IntConsumer`, `LongBinaryOperator` 등
- Function 인터페이스는 다양한 출력 형식 파라미터를 제공한다.
  - `ToIntFunction<T>`. IntToDoubleFunction` 등

다음은 자바 API에서 제공하는 대표적 함수형 인터페이스와 함수 디스크립터다.   
하지만 이것도 일부에 불과하며, 필요하다면 우리가 직접 함수형 인터페이스를 만들 수 있다.
<img width="572" alt="image" src="https://github.com/Songdoeon/Book_Study/assets/96420547/dbd275f0-a53c-4c5d-8797-7496030a3c1d">

## 📚 LESSEN 5 : 형식 검사, 형식 추론, 제약
람다 표현식을 처음 설명할 때 람다로 함수형 인터페이스의 인스턴스를 만들 수 있다고 했다.  

람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않다.   
따라서 람다 표현식을 더 제대로 이해하려면 람다의 실제 형식을 파악해야 한다.

### 🎈 형식 검사
람다가 사용되는 콘텍스트(Context)를 이용해서 람다의 형식을 추론할 수 있다.

어떤 콘텍스트에서 기대되는 람다 표현식의 형식을 **대상 형식**이라고 부른다.

형식 과정은 다음과 같이 진행된다.
- 메서드의 선언을 확인한다.
- 메서드의 파라미터로 대상 형식을 기대한다.
- 파라미터로 들어오는 함수형 인터페이스를 파악한다.
- 인터페이스의 함수 디스크립터를 묘사한다.
- 전달된 인수는 인터페이스의 요구사항을 만족해야 한다.

### 🎈 형식 추론
자바 컴파일러는 람다 표현식이 사용된 Context(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다.

즉, 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다.   
결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.   

자바 컴파일러는 다음처럼 람다 파라미터 형식을 추론할 수 있다.

- 파라미터 apple에 형식을 명시적으로 지정하지 않음
```java
List<Apple> greenApples = filter(inventory, apple -> GREEN.equals(apple.getColor()));
```

- 여러 파라미터를 포함하는 경우 가독성 향상이 더 두드러진다.
```java
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight() - a2.getWeight(); // 형식을 추론하지 않음

Comparator<Apple> c = (a1, a2) -> a1.getWeight() - a2.getWeight(); // 형식을 추론함
```

### 🎈 지역 변수 사용
람다 표현식에선 익명 함수가 하는 것 처럼 자유 변수를 활용할 수 있다.   
이같은 동작을 **람다 캡처링** 이라고 부른다.

- 람다는 인스턴스 변수와 정적 변수를 자유롭게 참조할 수 있다.
- 하지만 그러려면 지역 변수는 명시적으로 final이거나 실질적으로 final 변수처럼 사용되어야한다.
- 즉 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 참조할 수 있다.
  - 인스턴스 변수 참조는 final 지역 변수 this를 참조하는 것과 마찬가지다.

#### 📌 지역 변수의 제약
지역 변수의 이런 제약이 이해하기 힘들 수 있다.

자바는 내부적으로 인스턴스 변수는 힙에 지역변수는 스택에 저장한다.

람다에서 지역 변수에 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면   
변수를 할당한 스레드가 사라져 변수 할당이 해제되었지만 람다 실행스레드에서는 해당 변수에 접근하려 할 것이다.

따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다.  
그래서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당하는 제약이 생겼다.

그리고 지역 변수의 제약 때문에 외부 변수를 변화시키는 일반적인 Functional 프로그래밍 패턴에 제동을 걸 수 있다.

## 📚 LESSEN 6 : 메서드 참조
메서드 참조를 이용하면 기존 메서드 정의를 재활용해 람다처럼 전달할 수 있다.

때로는 람다보다 메서드 참조가 더 가독성 좋고 자연스러울 수 있다.

예제부터 살펴보자.
```java
inventory.sort((a1, a2) -> a1.getWeight() - a2.getWeight())

inventory.sort(comapring(Apple::getWeight);    // 메서드 참조
```

### 🎈 요약
메서드 참조는 특정 메서드만 호출하는 람다의 축약형이라 생각할 수 있다.

메서드 참조는 세 가지 유형으로 구분할 수 있다.
- 정적 메서드 참조
  - Integer::parseInt
- 다양한 형식의 인스턴스 메서드 참조
  - String::length
- 기존 객체의 인스턴스 메서드 참조
  - expensiveTransaction::getValue

### 🎈 생성자 참조
ClassName::new 처럼 클래스명과 new 키워드를 이용해 기존 생성자의 참조를 만들 수 있다.

이것은 정적 메서드의 참조를 만드는 방법과 비슷하다.

```java
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();

Function<Integer, Apple> c2 = Apple::new;
Apple a2 = c2.apply(110);
```

## 📚 LESSEN 7 : 람다 표현식을 조합할 수 있는 유용한 메서드
함수형 인터페이스에서는 다양한 유틸리티 메서드를 지원한다.   
Comparator, Function, Predicate같은 함수형 인터페이스는 람다 표현식을 조합할 수 있도록 유틸리티 메서드를 제공한다.

하지만 함수형 인터페이스는 하나의 추상메서드를 가진다고 했는데 어떻게 유틸리티 메서드를 제공할 수 있을까?

여기서 등장하는 것이 바로 **디폴트 메서드**이다.  
디폴트 메서드는 나중에 자세히 설명하겠다.

### 🎈 Comparator 조합
- Comparing : 비교에 사용할 키를 추출하는 Function 기반의 Comparator를 반환
- reversed : 역정렬
- thenComparing : 두 번째 비교자를 만든다.

```java
inventory.sort(comparing(Apple::getWeight)
         .reversed()    // 무게를 내림차순으로 정렬
         .thenComparing(Apple::getCountry));    // 사과의 무게가 같으면 국가별로 정렬
```

### 🎈 Predicate 조합
- negate : 특정 프레디케이트를 반전시킨다.
- and : && 연산
- or : || 연산
``` java
Predicate<Apple> notRedApple = redApple.negate();

Predicate<Apple> redAndHeavyAppleOrGreen =
    redApple.and(apple -> apple.getWeight() > 150)
            .or(apple -> GREEN.equals(a.getColor()));   // Predicate 메서드를 연결시켜 더 복잡한 객체를 만든다.
```

### 🎈 Function 조합
- andThen : 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달
- compose : 인수로 주어진 함수를 먼저 실행한 다음 그 결과를 외부 함수의 입력으로 전달


```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h1 = f.andThen(g);
int result1 = h1.apply(1);    // 4를 반환

Function<Integer, Integer> h2 = f.compose(g);
int result2 = h2.apply(1);    // 3을 반환
```
