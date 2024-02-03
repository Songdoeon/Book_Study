## 🌈 Chapter 5 : 스트림 활용

4장에서는 스트림을 이용해 외부 반복을 내부 반복으로 바꾸는 방법을 살펴봤다.   

명시적 반복 대신 filter와 collect 등 연산을 지원하는 스트림 API를 이용해 내부적으로 처리했다.

이번 장에서는 스트림 API가 지원하는 다양한 연산을 살펴보겠다.  

스트림 API가 지원하는 다양한 연산을 이용해 필터링, 슬라이싱, 매핑, 검색, 매칭, 리듀싱 등   
다양한 데이터 처리 질의를 표햔해보자.

그 다음으로는 숫자 스트림, 파일과 배열 등 다양한 소스로 스트림을 만드는 방법과, 무한 스트림 등 스트림의 특수한 경우도 살펴본다.

## 📚 LESSEN 1 : 필터링

스트림의 요소를 선택하는 방법,  
즉 **Predicate** 필터링 방법과 고유 요소만 필터링 하는 방법을 알아보자.

### 🎈 프레디케이트로 필터링
filter 메서드는 **Predicate**를 인수로 받아 일치하는 모든 요소를 포함하는 스트림을 반환한다.

예를 들어 다음과 같은 코드가 있다.

```java
List<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarian)    // 채식 요리인지 확인하는 메서드 참조
                                .collect(toList());
```

### 🎈 고유 요소 필터링
스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원한다.
  > 고유 여부는 객체의 hash, equals로 결정된다.

- 리스트의 짝수를 선택하고 중복을 필터링 한다.
```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
          .filter(i -> i % 2 == 0)
          .distinct()
          .forEach(System.out::println);
```

## 📚 LESSEN 2 : 스트림 슬라이싱

이제는 스트림의 요소를 선택하거나 스킵하는 다양한 방법을 설명하겠다.

스트림의 처음 몇 개의 요소를 무시하는 방법, 특정 크기로 스트림을 줄이는 방법 등   
다양한 방법을 이용해 효율적으로 이런 작업을 수행할 수 있다.

### 🎈 Predicate를 이용한 슬라이싱
자바 9는 스트림의 요소를 효과적으로 선택할 수 있도록 **takeWhile**, **dropWhile** 두 가지 새로운 메서드를 지원한다.

#### 📕 TAKEWHILE 활용
이미 정렬이 되어있는 데이터 컬렉션 에서 filter메서드로 모든 데이터에 대해 판단하는 것은 비효율적일 수 있다.

takeWhile 연산을 이용하면 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 슬라이스할 수 있다.

```java
List<Dish> slicedMenu1
	= specialMenu.stream()
                    .takeWhile(dish -> dish.getCalories() < 320)
                    .collect(toList());
```

#### 📕 DROPWHILE 활용
dropWhile 은 takeWhile과 정 반대의 작업을 수행한다.

predicate가 처음으로 거짓이 되는 지점까지 발견된 요소를 버린 후    
그 지점에서 작업을 중단하고 남은 요소를 반환한다.
```java
List<Dish> slicedMenu2
	= specialMenu.stream()
                    .dropWhile(dish -> dish.getCalories() < 320)
                    .collect(toList());
```

### 🎈 스트림 축소
스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit 메서드를 지원한다.

정렬이 되어있으면 최대요소 n개를 반환할 수 있고, filter와 같이 사용할 경우  
처음 판단된 limit 개수 만큼을 스트림에 담아낸다.

### 🎈 요소 건너뛰기
처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원한다.

n개 이하의 요소를 포함하는 스트림에는 빈 스트림을 반환시킨다.

그리고 skip 메서드와 limit 메서드는 상호 보완적인 연산을 수행하는데 다음과 같은 예가 있다.

- limit로 생성한 5개중 2개를 스킵
```java
IntStream.rangeClosed(1, 10).limit(5).skip(2).forEach(System.out::println);  // 3 4 5 출력
```

- skip 2개 진행 후 limit 5개 담기
```java
IntStream.rangeClosed(1, 10).skip(2).limit(5).forEach(System.out::println);  // 3 4 5 6 7 출력
```


## 📚 LESSEN 3 : 매핑
특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산이다.

스트림 API의 map과 flatMap 메서드는 특정 데이터를 선택하는 기능을 제공한다.

map에 대한 설명은 생략하도록 하겠다.

### 🎈 스트림 평면화
["Hello", "World]라는 데이터가 담긴 리스트가 있을때  
["H", "e", "l", "o", "W", "r", "d"]를 얻고싶다면 어떻게 할 수 있을까

- distinct로 중복 문자를 필터링하기
```java
words.stream()
      .map(word -> word.split(""))
      .distinct()
      .collect(toList());
```
위의 방법으로는 어림도 없다.

map 메서드의 반환 스트림 형식이 Stream<String[]> 인것이 문제다.   
우리는 Stream<String> 형식으로 받아보도록하자.

#### 📕 flatMap활용

- map과 Arrays.stream 활용
``` java
List<String> uniqueCharacters =
	words.stream()
            .map(word -> word.split(""))   // 각 단어를 개별 문자를 포함하는 배열로 변환
            .map(Arrays::stream)   // 각 배열을 별도의 스트림으로 생성
            .distinct()
            .collect(toList());
```
위와 같은 방식으로 하면 결국 List<Stream<String>>가 만들어지며 문제가 해결되지 않았다.

- flatMap 사용
```java
words.stream()
          .map(word -> word.split(""))    // 각 단어를 개별 문자열 배열로 반환
          .flatMap(Arrays::stream)    // 생성된 스트림을 하나의 스트림으로 평면화
          .distinct()
          .collect(toList());
```
flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다.


## 📚 LESSEN 4 : 검색과 매칭
특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용된다.

스트림 API는 다음과 같은 다양한 유틸리티 메서드를 제공한다.

- `anyMatch` : 적어도 한 요소와 프레디케이트가 일치하는지 확인
- `allMatch` : 모든 요소가 프레디케이트와 일치하는지 확인
- `noneMatch` : 프레디케이트와 일치하는 요소 하나도 없는지 확인

위의 세 메서드는 스트림 쇼트서킷 기법으로 자바의 &&, || 같은 연산을 활용한다.

- `findAny` : filter 메서드와 연계해 조건에 부합하는 아무 요소를 Optional형태로 반환한다.
- `findFirst` : filter 메서드와 연계해 조건에 부합하는 첫 요소를 Optional형태로 반환한다.

### 📌 findAny와 findFirst
두 메서드의 차이는 무엇일까?

바로 병렬성이다.

벙렬 실행에서는 첫 번째 요소를 찾기 어렵다.   
따라서 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.

### 🎈 Optional이란?
Optional<T> 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스다.

findAny 메서드는 아무 요소도 반환하지 않을 수 있는데 그렇다면 NPE의 유발이 심해짐으로   
자바 8에서는 Optional 클래스를 이용해 해당 문제를 해결하고자 했다.

Optional 에는 다음과 같은 기능이 있다는 것만 알아두고 일단 넘어가자

- `isPresent()`
  - Optional이 값을 포함하면 true, 아니면 false
- `isPresent (Consumer<T> block)`
  - 값이 있으면 주어진 블록을 실행한다.
- `T get()`
  - 값이 존재하면 값을 반환, 없으면 NoSuchElementException을 일으킨다.
- `T orElse(T other)`
  - 값이 존재하면 값을 반환, 값이 없으면 기본값을 반환.

## 📚 LESSEN 5 : 리듀싱

이번에는 Reduce연산을 이용해 복잡한 질의를 수행해보자.

이러한 질의를 수행하려면 Integer 같은 결과가 나올 때까지 스트림의 모든 요소를 반복적으로 처리해야하는데,  
이런 질의를 **리듀싱 연산**이라고 한다.

리듀스 연산의 파라미터는 다음과 같다.
- `total` : 초기값이자, 연산의 결과가 저장되는 값
- `연산자` : 수의 연산을 결정하는 값

- 요소들의 합과 곱을 구하는법
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
int mul = numbers.stream().reduce(1, (a, b) -> a * b);
```
- 최대값과 최소값을 구하는법
```java
int sum = numbers.stream().reduce(Integer::min);
int mul = numbers.stream().reduce((x, y) -> x < y ? x : y);  // 위와 같은 결과값을 가지는 코드
```

#### 📌 초기값 없음
reduce 연산에는 초깃값을 받지 않도록 오버로드된것도 있다
```java
int sum = numbers.stream().reduce((a, b) -> a + b);
```

이 reduce는 반환값이 Optional 객체인 이유가 무엇일까?

스트림에 아무 요소가 없을경우 합계가 없음을 가리킬 수 있도록 Optional 객체로 감싼 결과를 반환한다.

#### 📌 reduce의 이유
기존의 단계적 반복으로 합계를 구하는것과 reduce를 이용해 합계를 구하는 것은 어떤 차이가 있을까?

병렬로 반복적인 합계를 구하려면 sum 변수를 공유해야 하므로 쉽게 병렬화 하기 어렵다.    
강제적으로 동기화 시키더라도 결국 병렬화로 얻어야 할 이득이 스레드 간 경쟁에 상쇄될 것이다.

이 작업을 병렬화하려면 입력을 분할하고, 분할된 입력을 더한 값을 합쳐야 한다.

reduce를 활용하면 내부 반복이 추상화 되며 내부 구현에서 병렬로 reduce를 실행할 수 있게된다.

자세한 내용은 뒤에서 다시 다루도록 하겠다.

## 📚 LESSEN 6 : 숫자형 스트림
```java
int calories = menu.stream()
                      .map(Dish::getCalories)
                      .reduce(0, Integer::sum);
```
사실 위 코드에는 박싱 비용이 숨어있다.

내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱해야 한다.  
다음 코드처럼 직업 sum 메서드를 호출할 수 있다면 더 좋지 않을까?

```java
int calories = menu.stream()
                            .map(Dish::getCalories)
                            .sum();
```

하지만 위 코드처럼 sum 메서드를 직접 호출할 수 없다.

map 메서드가 Stream<T>를 생성하기 때문이다.   
그렇기에 스트림은 숫자 스트림을 효율적으로 처리할 수 있도록 기본형 특화 스트림을 제공한다.

### 🎈 기본형 특화 스트림
Java 8에서는 세 가지 기본형 특화 스트림을 제공한다.

Stream API는 박싱 비용을 피할 수 있도록 int 요소에 특화된 IntStream, double 요소에 특화된 DoubleStream, long 요소에 특화된 LongStream 을 제공한다.

각각의 인터페이스는 sum, max 같이 자주 사용하는 숫자 관련 리듀싱 연산 수행 메서드를 제공한다.   
또한, 필요할 때 다시 객체 스트림으로 복원하는 기능도 제공한다.

특화 스트림은 오직 박싱 과정에서 일어나는 효율성과 관련 있으며 스트림에 추가 기능을 제공하지는 않는다.

스트림을 특화 스트림으로 변환할 때는 mapToint, mapToDouble, mapToLong 세 가지 메서드를 가장 많이 사용한다.

```
int calories = menu.stream()
                   .mapToInt(Dish::getCalories)
                   .sum();  // 기본값 0

IntStream intStream = menu.stream().mapToInt(Dish::getCalories);    // 스트림을 숫자 스트림으로 변환
Stream<Integer> stream = intStream.boxed();    // 숫자 스트림을 스트림으로 변환
```

#### 📌 기본값 : OptionalInt
합계 에제에서 기본값이 없을 경우를 위해 Optinal 클래스를 이용한다고 언급했다.

하지만 기본값이 0이 실제 값일 경우와 스트림에 요소가 없는 상황을 어떻게 구분할까?

바로 OptinalInt, OptionalDouble, OptionalLong 세 가지의 기본형 특화 스트림 버전도 있다.

```java
OptionalInt maxCalories = menu.stream()
                              .mapToInt(Dish::getCalories)
                              .max();

int max = maxCalories.orElse(1);   // 값이 없을 경우 기본값을 명시적으로 설정
```

## 📚 LESSEN 7 : 스트림 만들기
스트림이 데이터 처리에 용이성을 지금까지 알아보았다.

그렇다면 컬렉션만 스트림을 이용하는것이 너무 아쉽지않은가!

우리는 다양한 데이터들을 스트림으로 다루는 방법에 대해 알아보자.

- `Stream.of()`
  - ({"Modern", "java"})이런 식으로 바로 배열을 할당하여 사용가능하다.
- `Stream.empty()`
  - 비어있는 스트림 생성
- `Stream.ofNullable()`
  - null이 될 수 있는 객체를 스트림으로 만들 수 있다.
  - 원래라면 null을 확인한 다음 비어있는 스트림을 생성해야 했다.
- `Arrays.stream()`
  - 배열을 스트림으로 만들 수 있다.
  - 기본형 int 배열을 IntStream으로 변환할 수 있다.

### 🎈 파일로 스트림 만들기
파일의 입출력을 처리하는 자바의 NIO API도 스트림 API를 활용할 수 있도록 업데이트 되었다.

```java
long uniqueWords = 0;
 //스트림은 자원을 자동으로 해제할 수 있는 AutoCloseble이라 try-finally가 필요없다.
try(Stream<String> lines =
          Files.lines(Paths.get("data.txt"), Charset.defaultCharset())){ 
uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))  // 고유 단어 수 계산
                   .distinct()  
                   .count();
}
catch(IOException e){
	// 예외처리
```
}

### 🎈 무한 스트림 만들기
스트림 API는 함수에서 스트림을 만들 수 있는 두 정적 메서드 `Stream.iterate`와 `Stream.generate`를 제공한다.

두 연산은 무한 스트림, 즉 크기가 고정되지 않은 스트림을 계속 만들기에 보통 limit와 함께 연결해서 사용한다.   
무한 스트림은 언바운드 스트림이라고 표현한다.

#### 📕 iterate 메서드
이제 iterate 예제를 살펴보자.

- 기본형
```java
Stream.iterate(0, n -> n + 2)
          .limit(10)
          .forEach(System.out::println)
```
- 피보나치 수열
```java
Stream.iterate(new int[] {0, 1},
                    t -> new int[]{t[1], t[0] + t[1])
          .limit(10)
          .map(t -> t[0])    // 배열의 첫 번째 요소만 추출
          .forEach(System.out::println)      // 0, 1, 1, 2, 3, 5, 8, 13, 21...
```
- iterate 메서드의 predicate 지원
```java
Stream.iterate(0, n -> n < 100, n -> n + 4)    // 해당 부분을 takeWhile로 도 구현할 수 있음
          .forEach(System.out::println)    
```

#### 📕 generate 메서드
기본적으로는 iterate와 비슷하지만 generate는 생산된 각 값을 연속적으로 계산하지 않는다.

무슨말인지 모르겠다면 다음 예제를 살펴보자.

```java
Stream.generate(Math::random)
          .limit(5)
          .forEach(System.out::println);
```

위 코드의 실행 결과는 다음과 같다.
0.871249812751
0.151982471982
0.093209330593
0.730182958242
0.125892138915

#### 📌 병렬 코드에서의 발행자
두 메서드의 차이가 잘 와닿지 않을 수 있다.

둘의 차이는 파라미터 인터페이스의 차이가 있다.

iterate 의 경우 Function<T, R>, generate 의 경우 Supplier<T>를 인자로 받는다.    
Supplier<T> 의 경우 상태가 없는 메서드이다.

supplier의 메서드를 커스텀하여 상태 필드를 정의 할 수 있긴하다.  
하지만 그럴 경우 상태 필드를 계속해서 업데이트해야 하기에 객체를 가변 상태로 만든다.

우리는 병렬 코드에서는 공유 변수를 다룰때의 주의점을 이야기 해왔다.  
스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 불변 상태를 유지해야 하는것이 좋을 것이다.

더 자세한 이야기는 뒤에서...
