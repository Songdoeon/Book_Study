## 🌈 Chapter 13 : 디폴트 메서드
전통적인 자바에서 인터페이스와 관련 메서드는 한 몸처럼 구성된다.

인터페이스를 구현하는 클래스는 인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 슈퍼클래스의 구현을 상속받아야 한다.   

평소에는 이 규칙이 문제없지만 라이브러리 설계자 입장에서 인터페이스에 새로운 메서드를 추가하거나,   
인터페이스를 바꾸고 싶을 때는 문제가 많아진다.

하지만 자바 8에서부터는 이 문제를 해결하는 새로운 기능을 제공한다.

기본 구현을 포함하는 인터페이스 정의하는 방법은 두 가지다.

- 인터페이스 내부 정적메서드 사용
- 인터페이스의 기본 구현을 제공하도록 디폴트 메서드 기능을 사용

디폴트 메서드를 이용하면 자바 API의 호환성을 유지하며 라이브러리를 바꿀 수 있다.

<img width="555" alt="스크린샷 2024-03-02 오후 1 01 46" src="https://github.com/Songdoeon/Book_Study/assets/96420547/355e7667-8187-4ba9-b648-cf19762ba49d">

디폴트 메서드가 없던 시절에는 인터페이스에 메서드를 추가하려면 인터페이스의 구현체 모두에 해당 메서드를 구현하도록 해야 했다.

보인이 직접 구현체들을 관리한다면 이 문제를 해결할 수 있지만, 인터페이스를 대중에 공개했다면 상황이 다르다.

라이브러리나 프레임워크(?) 개발을 하게된다면 해당 기능을 잘 아는게 중요하다.

#### 📌 정적 메서드와 인터페이스
보통 자바에서는 인터페이스 그리고 인터페이스의 인스턴스를 활용할 수 있는 다양한 정적메서드를 정의하는 유틸리티 클래스를 활용한다.

예를 들어 Collections는 Collection 객체를 활용할 수 있는 유틸리티 클래스다.

자바 8에서는 인터페이스에 직접 정적 메서드를 선언할 수 있으므로 유틸리티 클래스를 없애고   
직접 인터페이스 내부에 정적 메서드를 구현할 수 있다.

그럼에도 과거 버전과 호환성을 유지할 수 있도록 자바 API에는 유틸리티 클래스가 남아있다.

이번 장에서는 디폴트 메서드를 살펴볼 것이다.

## 📚 LESSION 1 : 변화하는 API
API를 바꾸는 것이 왜 어려운지 예제를 통해 알아보자.

우리는 인기 있는 자바 그리기 라이브러리 설계자가 되었다.

- 라이브러리에는 SetHeight, setWidth, getHeight, getWidth 등 메서드를 정의하는 `Resizable` 인터페이스가 있다.
- 뿐만 아니라 Rectangle, Square 처럼 `Resizable`을 구현하는 클래스도 제공한다.
- 라이브러리가 인기를 얻으며 일부 사용자는 직접 `Resizable`인터페이스를 구현하는 `Ellipse`라는 클래스를 구현하기도 했다.

이러한 상황에서 우리는 `Resizable` 인터페이스에 인수로 크기를 조절하는 `setRelativeSize`를 추가하며 구현체도 고쳤다.

모든 문제가 해결되었을까?

정답은 아니다 기존 사용자가 만든 클래스를 우리가 어떻게 할 수는 없다.    
그렇다면 어떤 방식으로 고칠 수 있을까?

### 🎈 API 버전 1
- Resizable 인터페이스 초기 버전
```java
public interface Resizable extends Drawble{
  int getWidth();
  int getHeight();
  void setWidth(int width);
  void setHeight(int height);
  void setAbsoluteSize(int width, int height);
```

- 사용자의 구현체
```java
public class Ellipse implements Resizable{
  ...
}

// 다양한 Resizable 모양을 처리하는 게임
public class Game{
  public static void main(String[] args){
    List<Resizable> resizableShapes =
      Arrays.asList(new Square(), new Rectangle(), new Ellipse());
    Utils.paint(resizableShape);
  }
}

public calss Utils{
  public static void paint(List<Resizable> l){
    l.forEach(r -> {
      r.setAbsoluteSize(42, 42);
      r.draw();
    });
  }
}
```
### 🎈 API 버전 2
몇 개월 후 사용자들은 `Resizable`를 구현하는 `Square`와 `Rectangle`을 개선해달라 요청했다.

그리하여 다음과 같은 구조로 API를 개선했다.

<img width="552" alt="스크린샷 2024-03-02 오후 1 21 12" src="https://github.com/Songdoeon/Book_Study/assets/96420547/316d4178-cec3-4108-b16a-8f187ed7c909">
```java
public interface Resizable{
  int getWidth();
  int getHeight();
  void setWidth(int width);
  void setHeight(int height);
  void setAbsoluteSize(int width, int height);
  void setRelativeSize(int wFactor, int hFactor);  // 새로 추가된 메서드
```
해당 메서드를 추가하며 여러가지 문제가 발생한다.

- `Resiable`을 구현하는 모든 클래스는 `setRelatievSize`메서드를 구현해야함
- 바이너리 호환성은 유지되지만 언젠가 문제가 일어날 여지가 있다.
  - 바이너리 호환성이란 새 메서드를 호출하지만 않으면 기존 클래스 파일 구현이 잘 동작한다는 의미다.
- Ellipse를 포함하는 전체 애플리 케이션을 재빌드할 때 컴파일 에러가 발생한다.

#### 📌 자바의 호환성
자바 프로그램을 바꾸는 것과 관련된 호환성 문제는 크게 세가지로 분류된다.
- 바이너리 호환성
  - 무언가 바뀐 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황
  - 바이너리 실행에는 인증, 준비, 해석 등의 과정이 포함된다.
  - Ex) 인터페이스에 메서드를 추가했을 때 추가된 메서드를 호출하지 않는 한 문제가 일어나지 않는다.
- 소스 호환성
  - 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일 할 수 있음을 의미한다.
  - Ex) 인터페이스에 메서드를 추가하면 소스 호환성이 아니다.
- 동작 호환성
  - 코드를 바꾼 다음에도 같은 입력에 프로그램이 같은 동작을 실행한다는 의미다.
  - Ex) 인터페이스에 메서드를 추가하더라도 프로그램에서 추가된 메서드를 호출할 일이 없다면 동작 호환성은 유지된다.


이러한 문제들을 예방해주는 것이 **디폴트 메서드**다.

디폴트 메서드를 이용해 API를 바꾸면 새롭게 바뀐 인터페이스에서 자동으로 기본 구현을 제공하므로   
기존 코드를 고치지 않아도 된다.

## 📚 LESSION 2 : 디폴트 메서드란?
공개된 API에 새 메서드를 추가하면 기존 구현체에 어떤 문제가 생기는지 살펴보았다.

자 그러면 그 문제들을 해결해준다는 디폴트 메서드에 대해 좀 더 알아보도록 하자.

인터페이스 자체에서 자신을 구현하는 구현체에게 구현하지 않아도 되는 새로운 메서드 시그니처를 제공한다.    
다폴트 메서드는 인터페이스 자체에서 기본으로 제공한다.

- 디폴트 메서드 선언법
```java
public interface Sized{
  int size();
  default boolean isEmpty(){    // 디폴트 메서드
    return size() == 0;         // 다른 메서드 처럼 바디가 존재함
  }
}
```

#### 📌 추상 클래스와 자바 8의 인터페이스
자 그러면 디폴트 메서드가 추상클래스와 무엇이 다른지 궁금할것이다.

추상 클래스와 인터페이스는 이제 둘 다 바디를 포함하는 메서드를 정의할 수 있게 되었다.

하지만 클래스는 하나의 추상 클래스만 상속 받을 수 있고, 인터페이스는 여러개를 구현할 수 있다.   
그리고 추상 클래스는 일스턴스 변수로 공통 상태를 가질 수 있지만, 인터페이스는 변수를 가질 수 없다.

자바는 기본적으로 다중 상속을 지원하지 않지만 디폴트 메서드로 인해 사실상 다중 상속이 가능해졌다고 볼 수 있다.

## 📚 LESSION 3 : 디폴트 메서드 활용 패턴
디폴트 메서드를 이용해 라이브러리를 바꿔도 호환성을 유지할 수 있음을 확인했다.

디폴트 메서드를 다른 방식으로도 활용할 수 있을까?

이 절에서는 디폴트 메서드를 이용하는 두 가지 방식을 소개하겠다.

### 🎈 선택형 메서드
인터페이스를 구현하는 클래스에서 메서드의 내용이 비어있을 수 있다.

예를 들어 Iterator 인터페이스는 `hasNext`, `next` 뿐아니라 `remove`메서드도 정의한다.

하지만 사용자들이 `remove`를 잘 사용하지 않아 자바 8이전에는 `remove`기능을 무시했다.   
결과적으로 `Iterator`를 구현하는 많은 클래스에서 remove에 빈 구현을 제공했다.

디폴트 메서드를 이용하면 `remove`같은 메서드에 기본 구현을 제공할 수 있으므로,   
인터페이스를 구현하는 클래스에서 빈 구현을 제공할 필요가 없다.

기본 구현이 제공되므로 Iterator 인터페이스를 구현하는 클래스는 빈 remove를 구현할 필요가 없어지고,   
불필요한 코드를 줄일 수 있다.

### 🎈 동작 다중 상속
디폴트 메서드를 이용하면 기존 불가능했던 다중 상속 기능도 구현할 수 있다.

<img width="553" alt="스크린샷 2024-03-02 오후 1 51 16" src="https://github.com/Songdoeon/Book_Study/assets/96420547/7a5fc2f5-87d5-45f5-bd46-56401ef73eb7">

- ArrayList 클래스
```java
public class ArrayList<E> extends AbstractList<E>  // 한개의 클래스를 상속받는다.
                          implements List<E>, RandomAccess, Cloneable, Serializable{}  // 4개의 인터페이스를 구현한다.
```

`ArrayList`는 한 개의 클래스를 상속받고 여섯 개의 인터페이스를 구현한다.

결과적으로 `ArrayList`는 AbstractList, List, RandomAccess, Cloneable, Serializable,   
Iterable, Collection의 **서브 형식**이 된다.

따라서 디폴트 메서드를 사용하지 않아도 다중 상속을 활용할 수 있다.

자바 8에서는 인터페이스가 구현을 포함할 수 있으므로, 클래스는 여러 인터페이스에서 동작을 상속받을 수 있다.

다중 동작 상속이 어떤 장점을 제공하는지 예제로 살펴보자.

#### 📕 기능이 중복되지 않는 최소의 인터페이스
우리는 최대한 기존 코드를 재사용하여 새로운 기능을 구현할 것 이다.

- Rotatable 인터페이스
```java
public interface Rotatable {
	void setRotationAngle(int angleInDegrees);
	int getRotationAngle();
	default void rotateBy(int angleInDegrees) {
		setRotationAngle((getRotationAngle() + angleInDegrees) % 360);
	}
}
```
기존 구현되어있는 메서드를 이용해 디폴트 메서드를 정의함으로서 별 다른 구현 없이 새로운 기능을 사용할 수 있게 된다.

이러한 방식으로 인터페이스를 리팩토링 해보자.
- Moveable 인터페이스
```java
public interface Moveable {
	int getX();
	int getY();
	void setX(int x);
	void setY(int y);

	default void moveHorizontally(int distance) {
		setX(getX() + distance);
	}

	default void moveVertically(int distance) {
		setY(getY() + distance);
	}
}
```
- Resizable 인터페이스
```java
public interface Resizable {
	int getWidth();
	int getHeight();
	void setWidth();
	void setHeight(int height);
	void setAbsoluteSize(int width, int height);
	
	default void setRelativeSize(int wFactor, int hFactor) {
		setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
	}
}
```

#### 📕 인터페이스 조합
이제 이 인터페이스를 조합해 다양한 클래스를 구현할 수도 있다.
```java
public class Monster implements Rotatable, Moveable, Resizable {
	...
}

이러한 방식으로 원하는 기능을 담은 클래스를 인터페이스를 빠르게 만들 수 있게 되었다.
```
#### 📌 옳지 못한 상속
상속으로 코드 재사용 문제를 모두 해결할 수 있는 것은 아니다.    
예를 들어 한 개의 메서드를 재사용 하려고 100개의 메서드와 필드가 정의되어 있는 클래스를 상속받는 것은 좋은 생각이 아니다.

이럴 때는 **델리게이션(delegation)**, 즉 멤버 변수를 이용해 클래스에서 필요한 메서드를    
직접 호출하는 메서드를 작성하는 것이 좋다.

종종 `final`로 선언된 클래스를 볼 수 있다.

다른 클래스가 이 클래스를 상속받지 못하게 함으로써 원래 동작이 바뀌지 않길 원하는 것이다.

우리의 디폴트 메서드에도 이 규칙을 적용하여, 필요한 기능만 포함하도록 인터페이스를    
최소한으로 유지한다면 필요 기능만 선택해 쉽게 기능을 조절할 수 있을 것이다.

## 📚 LESSION 4 : 해석 규칙
같은 디폴트 메서드 시그니처를 포함하는 두 인터페이스를 구현하는 상황이라면 어떻게 될까?

실제로 다중 상속을 지원하는 C++의 다이아몬드 문제가 있다.

이럴 때는 어떤 메서드가 사용될까? 자바 8은 이러한 문제의 해결 규칙을 제공한다.


### 🎈 알아야 할 세 가지 해결 규칙
다른 클래스나 인터페이스로부터 같은 시그니처를 갖는 메서드를 상속받을 때는 세 가지 규칙을 따라야 한다.

- 클래스가 항상 이긴다.
  - 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.
- 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다.
  - 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다.
  - 즉 B가 A를 상속받는다면 B가 A를 이긴다.
- 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면?
  - 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.
  
