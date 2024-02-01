## 🌈 Chapter 2 : 동작 파라미터화 코드 전달하기

우리가 일을 하며 겪을 일 중 하나인 소비자의 요구사항은 언제든 변할 수 있다.   
그런 상황속에서 유연한 프로그래밍은 필수이며, 우리가 꼭 적용해야할 점이라 생각한다.

그런 의미에서 **동작 파라미터화**는 아직 역할이 정해지지 않은 코드 블록으로   
동작을 호출자에게 넘김으로 다양한 기능을 할 여지를 남겨둔다.

이러한 동작 파라미터화의 예제를 이용해 살펴보도록 하겠다.

### 🎈 변화하는 요구사항에 대응하기
우리는 농장 재고목록 애플리케이션을 만드는 것을 목표로 한다.

#### 📕 녹색 사과 필터링
농장 주인은 녹색 사과를 필터링하고 싶어한다.
```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
	for (Apple apple: inventory) {
		if (GREEN.equals(apple.getColor()) { // 녹색 사과만 선택
				result.add(apple);
		}
	}
	return result;
}
```

하지만 위 코드는 다른 색 사과를 필터링하고싶어진다면 위 메소드와 비슷한 메소드가 나올것이다.  
그러므로 더 다양한 색을 필터링하기위해선 이 메서드를 추상화 하는 방법이 좀 더 효과적이다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
	List<Apple> result = new ArrayList<>(); 
	for (Apple apple: inventory) {
		if (apple.getColor().equals(color)) { 
				result.add(apple);
		}
	}
	return result;
}
```
위와 같이 메서드를 구현한다면
```java
List<Apple> greenApples = filterAppleByColor(inventory, GREEN);
List<Apple> redApples = filterAppleByColor(inventory, RED);
```
이와 같은 식으로 메소드를 호출할 수 있을것이다.

#### 📕 가능한 모든 속성으로 필터링
하지만 색이 아닌 무게로 사과를 필터링하는 방법을 요구한다면 어떻게 될까?

한발 더 나아가 색과 무게 모두를 필터링한다면?

우리는 변화하는 요구사항에는 좀 더 변화에 대응하기 좋은 코드를 구현해야할 것이다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
	List<Apple> result = new ArrayList<>(); 
	for (Apple apple: inventory) {
		if ((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) { 
				result.add(apple);
		}
	}
	return result;
}
```
하지만 그렇다고 위와 같이 코드를 작성한다면 추가되는 속성들과 의미를 모를 flag까지 생기며 코드가 더러워 보이고,  
확장에도 그렇게 썩 좋아보이지 않는다.

그렇다면 우리는 어떤식으로 코드를 작성해야할까?

### 🎈 동작 파라미터화
우리는 요구사항에 대응하며 파라미터 추가하는 방법이 해결책이 아니라는것을 확인했다.

그러므로 우리는 사과의 속성에 기초해 boolean값을 반환하는   
`Predicate` 라고 하는 **선택 조건을 결정하는 인터페이스** 정의하도록 하겠다.

```java
public interface ApplePredicate{
  boolean test(Apple apple);
}
```
- 무거운 사과만 선택
```java
public class AppleHeavyWeightPredicate implements ApplePredicate{
  public boolean test(Apple apple){
    return apple.getWeight() > 150;
  }
}
```
- 녹색 사과만 선택
```java
public class AppleGreenColorPredicate implements ApplePredicate{
  public boolean test(Apple apple){
    return GREAN.equals(apple.getColor());
  }
}
```
조건에 따라 filter 메서드가 다르게 동작할 것이며, 이를 전략 디자인 패턴(Strategy Design Pattern)이라 한다.

전략 디자인 패턴은 각 알고리즘을 캡슐화 하는 알고리즘 패밀리를 정의해둔 다음, 런타임에 알고리즘을 선택하는 기법이다.

우리의 예제에선 ApplePredicate가 알고리즘 패밀리,
AppleHeavyWeightPredicate와 AppleGreenColorPredicate가 전략이다.

#### 📕 추상적 조건으로 필터링
다음은 ApplePredicate를 이용한 필터 메서드다.

우리는 다음과 같이 filterApples 메서드의 동작을 파라미터화 한것이다.

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
	List<Apple> result = new ArrayList<>();
	for(Apple apple : inventory) {
		if(p.test(apple)) { // Predicate 객체로 사과 검사 조건을 캡슐화했다.
			result.add(apple);
		}
	}
	return result;
}
```

```java
List<Apple> redAndHeavyApples = filterApples(inventory, new AppleReadAndHeavyPredicate());
```

```java
public class AppleReadAndHeavyPredicate implements ApplePredicate{
  public boolean test(Apple apple){
    return RED.equals(apple.getColor()) && apple.getWeight() > 150;
  }
}
```

하지만 위 코드 또한 여러 클래스 구현으로 인스턴스화 과정에서 너무 많은 코드를 작성해야 한다.

### 🎈 복잡한 과정 간소화
자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 **익명클래스** 라는 기법을 제공한다.

익명클래스는 자바의 지역 클래스와 비슷한 개념으로, 사용한다면 불필요하거나 중복되는 코드의 양을 줄일 수 있다.

#### 📕 익명 클래스의 사용
자바의 익명클래스를 사용하다면 코드가 다음과 같이 변한다.
```java
List<Apple> redAndHeavyApples = filterApples(inventory, new ApplePredicate(){
  public boolean test(Apple apple){ // filterApples의 동작을 직접 파라미터화 하였다.
    return READ.equals.(apple.getColor()); 
  }
});
```

하지만 익명클래스도 아직까지는 많은 코드를 작성해야하고, 많은 프로그래머가 익명 클래스 사용에 익숙하지 않다.

코드의 장황함은 나쁜 특성이다.  
장황함은 코드를 구현 및 유지보수에 소요되는 시간을 증가시키고 개발자의 재미를 감소시킨다.

동작 파라미터화와 익명클래스의 사용에도 아직까지 코드가 썩 이쁘지않다.

그래서 우리는 자바8 설계자가 도입한 람다 표현식을 이용하여 더 간단한 코드 전달 기법을 이용하겠다.


#### 📕 람다 표현식 사용

일단 먼저 람다 표현식의 예제를 보여주겠다. 

람다 표현식을 이용하여 우리는 빨간 사과를 다음과 같이 구할 수 있디.
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

이전보다 코드가 훨씬 간단해지며 직관적으로 바뀐것으로 느껴지지않는가?

우선 람다의 예제들을 살펴보고 다음장에서 람다에 대해 구체적으로 살펴보도록 하겠다.

### 🎈 람다 실전 예제

지금까지 동작 파라미터화의 유연성과 이 메서드를 익명클래스, 그리고 람다로 발전시켰다.

다음은 람다를 이용한 활용 에제를 조금 살펴보겠다.

#### 📕 Comparator로 정렬하기
```java
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```
위 인터페이스를 이용하여 컬렉션 정렬을 다음과 같이 구현할 수 있다.

```java
inventory.sort((a1, a2) -> a1.getWeight() - a2.getWeight());
```

#### 📕 Runnable로 코드블록 실행하기

자바 스레드는 병렬 고드블록을 실행할 수 있도록 해준다.
우리는 여기에도 람다를 적용해볼 수 있다.
```java
public interface Runnable {
	void run();
}
```

```java
Thread t = new Thread(() -> System.out.println("Hello world"));
```

#### 📕 Callable을 결과로 반환하기
자바 5부터 지원하는 ExecutorService 추상화 개념이 있다.   
ExecutorService 인터페이스는 태스크 제출과 실행 과정의 연관성을 끊어준다.

ExecutorService를 이용하면 태스크를 스레드 풀로 보내고 결과를 Future로 저장한다는 점이   
thread와 Runnable을 이용하는 방식과 다르다.

이 개념은 뒷부분에서 병렬 실행을 설명할 때 더 설명하도록 하겠다.

당장은 Callable 인터페이스를 이용해 결과를 반환하는 태스크를 만든다는 사실만 알아두자.

```java
public interface Callable<V> {
  V call();
}
```

```java
Future<String> threadName = ExecutorService.submit(
  () -> Thread.currentThread().getName()
);
```

#### 📕 GUI 이벤트 처리하기
일반적으로 GUI 프로그래밍은 마우스 클릭이나 문자열 위로 이동하는 등의 이벤트에 대응하여   
동작을 수행하는 방식이다.

자바FX에서는 setOnAction 메서드에 EventHandler를 전달함으로써 이벤트 대응 방법을 설정한다.

```java
button.setOnAction((ActionEvent event) -> lavel.setText("Sent!!"));
```
