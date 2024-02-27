## 🌈 Chapter 11 : Null 대신 Optional 클래스

<details><summary>정리</summary>
  
```
- 역사적으로 프로그래밍 언어에서는 null 참조로 값이 없는 상황을 표현해왔다.
- 자바 8에서는 값의 null 여부를 `java.util.Optional<T>`클래스 제공으로 해결했다.
- 팩토리 메서드 `empty`, `of`, `ofNullable`등을 이용해 Optional 객체를 만들 수 있다.
- Optional 클래스는 스트림과 비슷한 연산 `map`, `flatMap`, `filter`등 의 메소드를 지원한다.
- Optional로 값이 없는 상황을 적절하게 처리하도록 할 수 있다.
  - 즉, 예상치 못한 null 예외를 방지한다.
- Optional을 활용해 더 좋은 API를 설계할 수 있다.
  - 사용자는 메서드의 시그니처만 보고도 Optional값이 사용되거나 반환되는지 예측할 수 있다.
```

</details>

우리는 실력에 상관없이 자바를 다루다 보면 NullPointerException이라는 예외와 마주치게 된다.

하지만 어쩌면 이 예외는 Null값을 쓰면서 치러야할 당연한 대가이다.

Null값을 쓰면서 이 에러를 만나지 않을 방법에 대해 설명하도록 하겠다.

## 📚 LESSEN 1 : 값이 없는 상황
예제를 들며 설명하겠다.

- 자동차와 보험을 가지고 있는 사람 객체를 중첩 구조로 구현한 코드
```java
public class Person{
  private Car car;
  public Car getCar() { return car; }
}

public class Car{
  private Insurance insurance;
  public Insurance getInsurance() { return insurance; }
}

public class Insurance {
  private String name;
  public String getName() { return name; }
}

// 문제가 일어날 수 있는 코드
public String getCarInsuranceName(Person person){
  return person.getCar().getInsurance().getName();
}
```
자 위 코드에서 문제점이 무엇일까?
바로 객체가 할당되어 있지 않은 경우 일것이다.

정보를 받으려 객체를 참조하지만 객체가 Null일 경우 바로 NPE가 발생한다.

### 🎈 보수적으로 NPE 줄이기
보수적으로 NPE를 예방하려면 어떻게 해야할까?
바로 객체를 참조하기 전마다 null 체크를 하는 것이다.

하지만 그런 코드는 다음과 같은 문제들이 있다.

- 깊은 의심(들여 쓰기)
```java
public String getCarInsuranceName(Person person){
  if(person != null){
    Car car = person.getCar();
    if(car != null){
      Insurance insurance = car.getInsurance();
      if(insurance != null){
        return insurance.getName();
      }
    }
  }
  return "Unknown";  // 사람이 없는지 차가 없는지 보험이 없는지 등을 모름
}
- 너무 많은 출구
```java
public String getCarInsuranceName(Person person){
  if(person == null){
    return "Unknown"
  }
  Car = person.getCar();
  if(car == null){
    return "Unknown";
  }
  Insurance insurance = car.getInsurance();
  if(insurance == null){
    return "Unknown";
  }
  return insurance.getName();
}
```
### 🎈 Null로 야기되는 문제
- 에러의 근원이다
  - NPE는 자바에서 가장 흔히 발생하는 예외다.
- 코드를 어지럽힌다.
  - 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코드 가독성이 떨어진다.
- 아무 의미가 없다.
  - null은 아무 의미도 표현하지 않는다.
  - 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.
- 자바 철학에 위배된다.
  - 자바는 개발자로붙 모든 포인터를 숨겼다.
  - 하지만 예외가 바로 null 포인터다.
- 형식 시스템에 구멍을 만든다.
  - null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다.
  - null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을때 null의 용도를 알 수 없게 된다.

### 🎈 Null대신 무엇을?
최근 그루비 같은 언어에서는 안전 내비게이션 연산자(?.)를 도입해 null 문제를 해결했다.
```
def carInsuranceName = person?.car?.insurance?.name
```

자 이제 자바에서는 어떻게 하는지 소개하겠다.

## 📚 LESSEN 2 : Optional 클래스
자바 8은 하스켈, 스칼라의 영향을 받아 `java.util.Optional<T>`라는 클래스를 제공한다.

`Optional`은 선택형값을 캡슐화하는 클래스다.

예를들어 어떤 사람이 차를 소유하고 있지 않다면 car 변수를 null이 아닌 Optional로 감싼 null을 가지는 것이다.
![image](https://github.com/Songdoeon/Book_Study/assets/96420547/e8b8b187-f26e-49f1-a944-46ffb4dcd74b)

그럼 null 참조와 Optional.empty()는 서로 무엇이 다른지 궁금할것이다.

Null을 참조하면 NPE가 발생하지만, Optional.empty()는 객체이므로 이를 다양한 방식으로 활용할 수 있다.

## 📚 LESSEN 3 : Optional 적용 패턴
Optional를 이용해 우리는 도메인 모델의 의미를 더 명확하게 만들 수 있고,
Null 참조 대신 값이 없는 상황을 표현할 수 있음을 확인했다.

이를 실제로 어떻게 활용할 수 있을까?

### 🎈 Optional 객체 만들기
Optional을 사용하려면? Optional 객체를 만들어야 한다.   
다양한 방법으로 Optional 객체를 만들 수 있다.

- 다양한 Optional 객체 만들기
```java
// 빈 Optional
Optional<Car optCar = Optional.empty();

// null이 아닌 Optional 만들기
Optional<Car> optCar = Optional.of(car);

// null값으로 Optional 만들기
Optional<car> optCar = Optional.ofNullable(car);  // car이 null이라면 빈 Optional 객체가 반환된다.
```

### 🎈 맵으로 Optional의 값을 추출하고 변환하기
Optional에선 map이라는 메서드를 지원한다.

- map 메서드 예제
```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);

Optional<String> name = optInsurance.map(Insurance::getName);
```

Optional의 map과 스트림의 map 메서드는 개념적으로 비슷하다.

스트림의 map은 스트림의 각 요소에 제공된 함수를 적용하는 연산,   
Optional 객체를 최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각할 수 있다.

Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾼다.

Optional이 비어있으면 아무 일도 일어나지 않는다.
![image](https://github.com/Songdoeon/Book_Study/assets/96420547/ae8e3630-a4c2-4fcc-926f-8203d62ab46f)

### 🎈 flatMap으로 Optional 객체 연결
map을 사용하는 방법을 배웠으므로, 다음처럼 flatMap을 이용해 코드를 재구현할 수 있다.

스트림에서 2차원 배열을 flatMap을 이용해 함수를 인수로 받아 다른 스트림으로 반환하듯
Optional 에서도 flatMap을 이용하여 2차원 Optional을 1차원 Optional로 평준화 해줘야한다.
```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson.flatMap(Person::getCar)
                                  .flatMap(Car::getInsurance)
                                  .map(Insurance::getName)
                                  .orElse("Unkown");    // Optional이 비어있다면 기본값 사용
```

### 📌 도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유
우리는 앞선 예제에서 Optional을 이용해 도메인 모델에서 값이 null인지 체크를 구체적으로 표현하였다.

하지만 놀랍게도 Optional 클래스의 설계자는 이와 다른 용도로만 Optional을 사용할 것을 가정했다.   
Optional의 용도가 선택형 반환값을 지원하는 것이라고 명확하게 못을 박았다.

Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않아 Serializable을 구현하지 않았다.
따라서 우리 모델에 Optional을 사용한다면 직렬화 모델을 사용하는 도구, 프레임워크에서 문제가 생길 수 있다.

이런 단점에도 여전히 Optional을 사용해 도메인 모델을 구성하는 것이 바람직하다고 생각한다.  // 책 저자의 생각인듯

특히 객체 그래프에서 일부 또는 전체 객체가 null일 수 있는 상황이면 더욱 그렇다.

직렬화 모델이 필요하다면 다음 예제에서 보여주는 것 처럼 Optional로 반환받을 수 있는 메서드를 추가하는 방식을 권장한다.

```java
public class Person{
  private Car car;
  public Optional<Car> getCarAsOptional() {
    return Optional.ofNullable(car);
  }
}
```

### 🎈 Optional 스트림 조작
자바 9에서는 Optinal을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다.

- 사람 목록을 이용해 가입한 보험 회사 이름 찾기
```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
  return persons.stream()
                .map(Person::getCar)    // 사람 목록을 각 사람이 보유한 Optional<Car> 스트림으로 변환
                .map(optCar -> optCar.flatMap(car::getInsurance))    // FlatMap 연산으로 Optional<Insurance>으로 변환
                .map(optIns -> optIns.map(Insurance::getName))    // FlatMap 연산으로 Optional<String>으로 매핑
                .flatMap(Optional::stream)    //Stream<Optional<String>>을 현재 이름을 포함하는 Stream<String> 으로 변환
                .collect(toSet());

Stream<Optional<String>> stream = getCarInsuranceNames(persons);

Set<String> result = stream.filter(Optional::isPresent)  // null이 아니라면
                            .collect(toSet());
```

### 🎈 디폴트 액션과 Optional 언랩
Optional클래스의 포함된값을 읽는 다양한 방법을 소개하겠다.

- `get()` 메서드는 가장 간단하면서 가장 안전하지 않은 메서드다.
  - 래핑된 값이 없다면 `NoSuchElementException`을 발생시킴
  - 중첩된 null 확인코드를 넣는것과 다를게 없음
- `orElse(T other)` 메서드는 Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있음
- `orElseGet(Supplier<? extends T> other)` 메서드는 Optional 값이 null일 경우 supplier를 실행한다.
  - 디폴트 메서드를 만드는데 시간이 걸리거나(효율성) Optional이 비어있을 때 기본값을 생성한다.
- `orElseThrow (Supplier<? extends X> exceptionSupplier)는 null일 경우 예외를 발생시킨다.
  - 하지만 발생시킬 예외를 커스텀할 수 있다.
- `ifPresent(Consumer<? super T> consumer)를 이용하면 값이 존재할 때 인수로 넘겨준 동작을 실행한다.
  - 값이 없다면 아무 일도 일어나지 않는다.
- `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`
  - 자바 9에서 추가된 인스턴스 메서드
  - Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다.
 

### 🎈 두 Optional 합치기
Person과 Car 정보를 이용해 저렴한 보험료를 제공하는 보험회사를 찾는 서비스를 Optional로 구현해보자.

두 Optional을 인수로 받아 Optional<Insurance>를 반환하는 메소드다.
```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
  if(person.isPresent() && car.isPresent()){
    return Optional.of(findCheapestInsurance(person.get(), car.get()));
  }
  else{
    return Optional.empty();
  }
}
```
음 하지만 null 체크하는 코드는 딱히 다를게 없어보인다.

좀더 멋있게 리팩토링 해보자

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
  return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```
첫 Optional에 flatMap을 호출하여 null이라면 전달한 람다식이 실행되지 않고 그대로 빈 Optional을 반환한다.

person 값이 있다면 flatMap 메서드에 필요한 Optional<Insurance>를 반환하는 Function의 입력으로 person을 사용한다.    
이 함수의 바디에서는 두 번째 Optional에 map을 호출하고 null을 체크한다.

결국 person과 car가 모두 존재해야 map메서드로 전달한 람다식이 findCheapestInsurance를 안전하게 호출할 수 있다.


### 🎈 Optional의 Filter
Optional에서 filter를 사용할때에도 null체크와 predicate로 값을 비교해야할 것이다.

- Optional 객체에서의 filter 메서드
```java
Optional<Insurance> optInsurance = ...;

optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"));
```

## 📚 LESSEN 4 : Optional을 사용한 실용 예제
Optional 클래스를 효과적으로 이용하려면 잠재적으로 존재하지 않는 값의 처리 방법을 바꿔야 한다.   
즉, 코드 구현만 바꾸는 것이 아니라 네이티브 자바 API와 상호작용하는 방식 도 바꿔야 한다.

사실 저자는 Optional 클래스 지원 여부가 더 일찍 결정되었다면    
많은 API 구현 코드가 달라졌을거라 생각한다고한다.

호환성 유지때문에 기존 자바 API는 Optional을 적절하게 활용하고 있지 못한다.    
그렇다고 기존 API에 Optional을 쓰지 못하는것은 아니다.

Optional기능을 활용 가능하도록 우리 코드에 작은 유틸리티 메서드를 추가하는 방식을 이용해 보도록 하겠다.

### 🎈 잠재적 null 대상을 Optional로 감싸기
get메서드의 시그니처를 우리가 고칠 수는 없지만 get메서드의 반환값은 Optional로 감쌀 수 있다.

Map<String, Object> 형식의 맵에서 다음처럼 key로 값에 접근한다.
```java
Object value = map.get("key");    // NPE 유발

Optional<Object> value = Optional.ofNullable(map.get("key"));
```

### 🎈 예외와 Optional
자바 API는 어떤 이유에서 값을 제공할 수 없을 때 null 대신 예외를 발생시킬 때도 있다.

예를 들어 `Integer.parseInt`는 문자열을 정수로 바꾸지 못하면 `NumberFormatException`을 발생시킨다.   
즉, 문자열이 숫자가 아니라는 사실을 예외로 알리는 것이다.

이처럼 정수로 반환 불가능한 문자열 문제를 빈 Optional로 해결할 수 있다.

- 문자열을 정수 Optional로 변환
```java
public static Optional<Integer> stringToInt(String s){
  try{
    return Optional.of(Integer.parseInt(s));
  } catch (NumberFormatException e) {
    return Optional.empty();
  }
}
```

### 🎈 기본형 Optional을 피해야 하는 이유
스트림처럼 Optional도 기본형 특화 클래스 `OptionalInt`, `OptionalLong`, `OptionalDouble`가 있다.   
예를 들어 `Optional<Integer>` 대신 `OptionalInt`를 사용할 수 있다.

5장에서 스트림이 많은 요소를 가질 때는 기본형 특화 스트림을 이용해 성능을 향상시킬 수 있다고 했다.
  > 박싱, 언박싱 비용을 줄인다

하지만 Optional의 최대 요소 수는 한 개 이므로 성능을 개선할 수는 없다.  
그리고 Optional의 유용한 메서드 `map`, `flatMap`, `filter`등을 기본형 특화 Optional 에서는 지원하지 않는다.

게다가 스트림과 마찬가지로 특화 Optional로 생성한 결과는 다른 일반 Optional과 혼용할 수 없다.

예를 들어 OptionalInt를 반환한다면 다른 Optional의 flatMap에 메서드 참조로 전달할 수 없다.

