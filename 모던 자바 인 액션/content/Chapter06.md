## 🌈 Chapter 6 : 스트림으로 데이터 수집

5장에서는 스트림을 이용해 DB 같은 연산을 수행할 수 있음을 배웠다.

이번 장에서는 스트림의 최종 연산 collect의 다양한 요소 누적 방식으로 리듀싱 연산을 수행할 수 있음을 설명한다.    
지금부터 컬렉션, 컬렉터, collect를 헷갈리지 않도록 주의하자.

자 언제나처럼 우리가 배울 예제 코드 먼저 보고 시작하도록 하자.

- 통화별로 트랜잭션을 그룹화한 코드(명령어 버전)
```java
Map<Currency, List<Transaction>> transactionsByCurrencies =
                                                      new HashMap<>();    // 그룹화한 트랜잭션을 저장할 맵을 생성

for(Transaction transaction : transactions) {
  Currency currency = transaction.getCurrency();
  List<Transaction> transactionsForCurrency =
            transactionsByCurrencies.get(currency)    // 트랜잭션의 통화를 추출
  if(transactionsForCurrency == null) {              
    transactionsForCurrency = new ArrayList<>();      // 현재 통화를 그룹화하는 맵을 만든다.
    transactionsByCurrencies.put(currency, transactionsForCurrency);
  }
  transactionsForCurrency.add(transaction);      // 같은 통화를 가진 트랜잭션 리스트에 현재 탐색 중인 트랜잭션을 추가
}

```

- collect 메서드의 활용 버전
```java
Map<Currency, List<Transaction>> transactionsByCurrencies =
          transactions.stream().collect(groupingBy(Transaction::getCurrecny));
```

자 이 간결한 문장을 배워보도록 하자.

## 📚 LESSEN 1 : 컬렉터란 무엇인가?
Collector 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출할지 지정한다.

우리는 각 요소를 List로 만들어 내는 toList를 Collector 인터페이스 구현으로 주로 사용했다.

여기서는 groupingBy를 이용해 각 키 버킷 그리고 각 키 버킷에 대응하는 요소 리스트를 값으로 포함하는 Map을 만들라는 동작을 수행한다.

다수준(Multi level)으로 그룹화를 수행할 때 명령형 프로그래밍과 함수형 프로그래밍의 차이점이 더욱 두드러진다.

명령형 코드는 다중루프, 조건문 등으로 가독성과 유지보수성이 떨어지지만,   
함수형 프로그래밍에서는 필요한 컬렉터를 쉽게 추가할 수 있다.

### 🎈 고급 리듀싱 기능을 수행하는 컬렉터
collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다는 점이 컬렉터의 최대 강점이다.

collect에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리한다.

<img width="656" alt="스크린샷 2024-02-11 오후 12 34 55" src="https://github.com/Songdoeon/Book_Study/assets/96420547/e78d9f0f-f17d-4eed-b9d6-710697be58ac">

### 🎈 미리 정의된 컬렉터
이번에는 미리 정의된 컬렉터, 즉 `groupingBy` 같이 Collectors 클래스에서 제공하는 팩토리 메서드의 기능을 설명한다.

Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 구분할 수 있다.

- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

## 📚 LESSEN 2 : 리듀싱과 요약
우리는 `Stream.collect`를 이용하여 스트림의 항목을 컬렉션으로 재구성 할 수 있다.

그것들을 활용하는 방법들을 천천히 한 번 살펴보자

### 🎈 스트림의 최댓값과 최솟값
`Collectors.maxBy`, `Collectors.minBy` 두 개의 메서드를 이용해 스트림의 최댓값과 최솟값을 계산할 수 있다.

```java
Comparator<Dish> dishCaloriesComaparator =
                        Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCalorieDish =
                    menu.stream()
                        .collect(maxBy(dishCaloriesComaparator));
```

### 🎈 요약 연산
스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용된다.

이러한 연산을 **요약 연산**이라 부른다.

Collectors 클래스는 `Collectors.summingInt`라는 특별한 요약 팩토리 메서드를 제공한다.

- Collectors.summingInt
```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

- Collector.summingDouble
```java
double avgCalories = menu.stream().collect(summingDouble(Dish::getCalories));
```

그리고 다양한 연산을 한번에 지원하는 `summarizingInt` 메서드도 존재한다.
```java
IntSummaryStatistics menuStatistics =
                          menu.stream()
                              .collect(summarizingInt(Dish::getCalories));

IntSummaryStatistics{count = 9, sum = 4300, min = 120, average = 477.44478, max = 800} // 출력결과
```

### 🎈 문자열 연결
컬렉터의 `joining` 팩토리 메서드를 이용하면 스트림의 각 개체에 toString 메서드를 호출해 모든 문자열을 하나로 연결해 반환한다.

내부적으로 StringBuilder를 이용하기때문에 성능상 크게 떨어지지 않을것이다.

- joining 연산
```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", ")); // joining에 구분 문자를 따로 입력하지 않으면 공백으로 적용된다.

// 실행 결과 pork, beef, chicken, frech fries ...
```

### 🎈 범용 리듀싱 요약 연산
다음은 `reducing` 팩토리 메서드를 알아보자.

- 기본 reducing 메서드
```java
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i , j) -> i + j));
```

- max 값을 가진 요소 찾기
```java
Optional<Dish> mostCalorieDish =
            menu.stream().collect(reducing(
                          (d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

### 📌 reduce vs collect
위의 연산들은 reducing으로도 구현할 수 있다. 

그런데 왜 collect와 같은 기능을 하는것 처럼 보이는 연산이 두 가지나 있을까?

둘의 차이점은 다음과 같다.

- collect 메서드는 도출 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드다.
  - 반면 reduce는 두 값을 하나로 도출하는 불변형 연산이다.
- generate도 상태를 가지도록 커스텀해서 사용할 수 있지만 그 방식을 지양하듯 reduce 메서드도 마찬가지다.
  - 상태나, 누적자를 사용하게되면 멀티 스레드 환경에서 문제가 발생할 가능성이 크다.
- 위 문제를 해결하려면 매번 새로운 상태나 누적자를 가져야 하고, 객체 할당에 성능이 저하될 것이다.

그러므로 병렬성을 확보하려면 collect 메서드로 리듀싱 연산을 구현하는 것이 바람직하다.

### 🎈 같은 연산 다양한 방식
이전 reducing 컬렉터를 사용한 예제에서 `sum` 메서드 참조와 람다를 이용하면 코드를 좀 더 단순화할 수 있다.

- 메서드 참조를 이용한 연산
```java
int totalCalories = menu.stream().map(Dish::getCalories).reduce(Integer::sum).get;
```
- IntStream을 이용한 연산
```java
int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum();
```

컬렉터를 이용하는 코드는 조금 더 복잡할 수 있지만 재사용성과 커스터마이징의 가능성이 조금 더 높다.

대신, IntStream을 사용하면 언방식 비용 절약까지하여 성능과 가독성까지 챙겨갈 수 있다.

## 📚 LESSEN 3 : 그룹화
데이터 집합을 하나 이상의 특성으로 분류해 그룹화하는 DB에서 연산도 많이 수행되는 작업이다.

자바 8의 `Collectors.groupingBy`를 이용하면가독성 있는 한 줄의 코드로 그룹화를 구현할 수 있다.

- `Collectors.groupingBy` 예제
```java
Map<Dish.Type, List<Dish> dishesByType =
                            menu.stream().collect(groupingBy(Dish::getType));

// Map의 출력 결과다.
{Fish =[prawns, salmon], OTHER = [french fries, rice, season fruit, pizza], MEAT=[pork, beek, chicken]}
```
- 람다를 이용한 예제
```java
public enum CaloricLevel { DIET, NORMAL, FAT}

Map<CaloricLevel, List<Dish> dishesByCaloricLevel = menu.stream().collect(
                                groupingBy(dish -> {
                                    if(dish.getCalories() <= 400) return CaloricLevel.DIET;
                                    else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                                    else return CaloricLevel.FAT;
                                }));
```

이 함수를 기준으로 스트림이 그룹화되므로 이를 **분류 함수**라고 한다.

### 🎈 그룹화된 요소 조작
자 그러면 두 가지 기준으로 동시에 그룹화할 수 있을까?

- fitering 후 collect 하는 방식
```java
Map<Dish.Type, List<Dish> caloricDishesByType =
                            menu.stream().filter(dish -> dish.getCalories() > 500)
                                        .collect(groupingBy(Dish::getType));

// Map의 출력 결과다.
{OTHER = [french fries, pizza], MEAT=[pork, beek]}
```

fitering 후 collect 하는 방식은 아예 fish라는 종류가 맵에 담기지 않을 수 있다.

- Collectors의 filtering 메서드
```java
Map<Dish.Type, List<Dish> caloricDishesByType =
                            menu.stream()
                                .collect(groupingBy(Dish::getType,
                                          filtering(dish -> dish.getCalories() > 500, toList())));

// Map의 출력 결과다.
{OTHER = [french fries, pizza], MEAT=[pork, beek], FISH=[]}
```

- 맵핑 함수를 이용한 요소 변환
```java
Map<Dish.Type, List<String> dishNamesByType =
                            menu.stream()
                                .collect(groupingBy(Dish::getType, mapping(Dish::getName, toList())));
```

### 🎈 다수준 그룹화
`Collectors.groupingBy`는 일반적인 분류 함수와 컬렉터를 인수로 받는다.

즉, 바깥쪽 groupingBy메서드에 스트림의 항목을 분류할 두 번째 기준을    
정의하는 내부 groupingBy를 전달해 두 수준으로 스트림의 항목을 그룹화 할 수 있다.

- 다수준 그룹화 예제
```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel =
              menu.stream().collect(
                  groupingBy(Dish::getType,
                      groupingBy(dish -> {
                          if(dish.getCalories() <= 400) return CaloricLevel.DIET;
                          else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                          else return CaloricLevel.FAT;
                      })
                  )
              );

// 결과값
{MEAT = {DIET = [Chicken], NORMAL = [beef], FAT = [pork]}, FISH = {DIET = [prawns], ...
```

### 🎈 서브 그룹으로 데이터 수집
groupingBy의 컬렉터의 두 번째 인수 컬렉터의 형식은 제한이 없다.

다양한 컬렉터를 이용하는 예제를 보자.

- counting 컬렉터
```java
Map<Dish.Type, Long> typesCount = menu.stream().collect(
                                  groupingBy(Dish::getType, counting()));

// 결과
{MEAT = 3, FISH = 2, OTHER = 4}
```

- maxBy 및 collectingAndThen을 이용한 Optional 변환
```java
Map<Dish.Type, Optional<Dish>> mostCaloricByType =
                  menu.stream()
                      .collect(groupingBy(Dish::getType,  //분류 함수
                            collectingAndThen(
                                maxBy(comparingInt(Dish::getCalories)), // Optional 객체를 반환함
                            .Optional::get)));    // 변환 함수

// 결과
{FISH = salmon, OTHER = pizza, MEAT = pork}
```

## 📚 LESSEN 4 : 분할

분할은 **분할 함수** 라 불리는 Predicate를 분류 함수로 사용하는 특수한 그룹화 기능이다.

분할 함수는 Boolean을 반환하므로 맵의 키 형식은 Boolean이다.   
결과적으로 그룹화 맵은 최대 두 개의 그룹으로 분류된다.

- `partitioningBy`메서드 활용
```java
Map<Boolean, List<Dish>> partitionedMenu =
                          menu().stream().collect(partitioningBy(Dish::isVegetarian));

// 결과
{false = [pork, beef, chicken], true = [french fries, rice, season fruit]}

// 채식요리는 얻는법
List<Dish> vegetarianDishes = partitionedMenu.get(true);
```

### 🎈 분할의 장점
분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스틀 모두 유지한다는 것이 분할의 장점이다.

`partitioningBy`가 반환한 맵 구현은 true, false 두 가지 키만 포함하므로 간결하고 효과적이다.

그리고 partitioningBy를 이용하여도 다수준의 분할 결과를 만들어 낼 수 있다.

### 📌 Collectors 클래스의 정적 팩토리 메서드

<img width="620" src="https://github.com/Songdoeon/Book_Study/assets/96420547/326778dd-043f-4bac-b2de-d2d933994f5c">
<img width="630" src="https://github.com/Songdoeon/Book_Study/assets/96420547/b14fabc7-f5d0-46f3-9fd5-f13ea756c669">

## 📚 LESSEN 5 : Collector 인터페이스
Collector 인터페이스는 리듀싱 연삭(즉, 컬렉터)을 어떻게 구현할 지 제공하는 메서드 집합으로 구성된다.

다음 코드는 Collector 인터페이스의 시그니처와 다섯 개의 메서드 정의를 보여준다.

- Collector 인터페이스
```java
public interface Collector<T, A, R> {
  Supplier<A> supplier ();
  BiConsumer<A, T> accumulator();
  Function<A, R> finisher();
  BinaryOperator<A> combiner();
  Set<Characteristics> characteristics();
```
  - T : 수집될 스트림 항목의 제네릭 형식
  - A : 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
  - R : 수집 연산 결과 객체의 형식(대게 컬렉션 형식)이다.

### 🎈 Collector 인터페이스의 메서드 살펴보기

#### 📕 Supplier 메서드 : 새로운 결과 컨테이너 생성
- supplier 메서드는 빈 결과로 이뤄진 Supplier를 반환해야 한다.
- 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수

#### 📕 Accumulator 메서드 : 결과 컨테이너에 요소 추가
- accumulator 메서드는 리듀싱 연산을 수행하는 함수를 반환한다.
- 스트림에서 n 번째 요소를 탐색할 때 누적자와 n 번째 요소를 함수에 적용한다.
- 반환값은 void
  
#### 📕 Finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용
- 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하며 끝낼 때 호출할 함수를 반환한다.
- 때로는 ToListCollector에서 처럼 누적자가 이미 최종 결과인 상황도 있다.
- 그럴 땐 변환 과정이 필요없으므로 항등 함수를 반환한다.

#### 📕 combiner 메서드 : 두 결과 컨테이너 병합
- 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의한다.
- combiner 이용시 스트림의 리듀싱을 병렬로 수행할 수 있다.

#### 📕 Characteristics 메서드
- 컬렉터의 연산을 정의하는 Characteristics 형식의 불변 지밥을 반환한다.
- 스트림을 병렬로 리듀스할 것인지, 한다면 어떤 최적화를 선택할지 힌트를 제공한다.
- Characteristics는 다음 세 항목을  포함하는 열거형이다.
  - `UNORDERED`
    - 리듀싱 결과는 스트림 요소의 방문 순서가 누적 순서에 영향을 받지 않는다.
  - `CONCURRENCT`
    - 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있음
    - 컬렉터의 플래그에 `UNORDERED`를 함께 설정하지 않았다면     
      데이터가 정렬되어 있지 않은 상황에서만 병렬 리듀싱을 수행할 수 있다.
  - `IDENTITY_FINISH`
    - finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 생략 가능하다.
    - 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다.
    - 누적자 A를 결과 R로 안전하게 형변환할 수 있다.

## 📚 LESSEN 6 : 커스텀 컬렉터를 구현해 성능 개선하기
커스텀 컬렉터를 이용하여 성능 개선에 한번 힘 써보자.

나는 생략을 했지만 앞에서 isPrime() 이라는 메소드로 해당 숫자가 소수인지 아닌지 판별하는 메서드가 있었다.

구현 방식은 해당 숫자를 1부터 제곱근 까지의 수로 나눠보며 판단한다.

성능 개선은 백엔드 개발자로서 반드시 가지고 가야할 역량이 아닌가 싶다.   
성능 개선에 도움을 줄 수 있는 부분을 다른 문제에도 적용할 수 있을지도 모르니 잘 배워보도록 하자.

### 🎈 1단계 : 소수로만 나누기
우선 소수로만 나누어떨어지는지 확인하여 대상의 범위를 좁힐 수 있다.

하지만 그러려면 지금까지 발견한 소수 리스트에 접근해야 하는데,   
이 부분을 커스텀 컬렉터 클래스로 해결할 수 있다.

중간 결과 리스트가 있다면 isPrime 메서드로 중간 결과 리스트를 전달하도록 다음과 같이 코드를 구현할 수 있다.
```java
public static boolean isPrime(List<Integer> primes, int candidate){
  return primes.stream().noneMatch(i -> candidate % i == 0);
```
이제 위 코드에도 제곱근 까지만 나누도록 커스텀 해보도록 하자.

```java
public static boolean isPrime(List<Integer> primes, int candidate){
  int candidateRoot = (int) Math.sqrt((double) candidate);
  return primes.stream()
                .takeWhile(i -> i <= candidateRoot)
                .noneMatch(i -> candidate % i == 0);
```

이제 isPrime 메서드를 구현했으니 커스텀 컬렉터를 구현해보도록하자.

### 🎈 2단계 : Collector 클래스 시그니처 정의
Collector 인터페이스 정의를 참고해 클래스 시그니처를 만들자.

```java
public class PrimeNumbersCollector implements Collector<Integer,                        // 스트림 요소의 형식
                                                        Map<Boolean, List<Integer>>,    // 누적자 형식
                                                        Map<Boolean, List<Integer>>>    // 수집 연산의 결과 형식
```

### 🎈 3단계 : 리듀싱 연산 구현
이번에는 Collector 인터페이스의 다섯 메서드를 구현해보자.

- Supplier 메서드
```java
public Supplier<Map<Boolean, List<Iteger>>> supllier() {
    return () -> new HashMap<Boolean, List<Integer>>() {{
        put(true, new ArrayList<Integer>());        
        put(false, new ArrayList<Integer>());        // 소수와 비소수를 구분하여 빈 리스트에 추가한다.
    }};
}
```

- Biconsumer 메서드
  - 스트림 요소의 수집을 결정하는 메서드로 최적화의 핵심 메서드다.
```java
public Biconsumer<Map<Boolean, List<Iteger>>> accmulator() {
    return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
        acc.get(isPrime(acc.get(true), candidate))      // isPrime의 결과에 따라 리스트를 만든다.
            . add(candidate);                           // candidate를 알맞은 리스트에 추가한다.
    };
}
```

### 🎈 4단계 : 병렬 실행가능한 컬렉터 만들기(가능하다면)
이번에는 병렬 수집 과정에서 두 부분 누적자를 합칠 수 있는 메서드를 만든다.

예제에서는 단순히 두 번째 맵의 소수 리스트와 비소수 리스트의 모든 수를 첫 번째 맵에 추가하는 연산이면 충분하다.

하지만 이런 순차적인 알고리즘에서는 컬렉터를 병렬로 실행할 수 없다.

그러므로 사실 빈 구혀느로 남겨둘 수도 있지만, 학습을 목적으로 구현해보도록 하자.
- Combiner 메서드
```java
public BinaryOperator<Map<Boolean, List<Integer>> combiner(){
    return (Map<Boolean, List<Integer>> map1, Map<Boolean, List<Integer>> map2) -> {
            map1.get(true).addAll(map2.get(true));
            map1.get(false).addAll(map2.get(false));
            return map1;
    };
}
```


### 🎈 5단계 : finisher 메서드와 characteristics 메서드
나머지 두 메서드는 쉽게 구현 가능하다.

accumulator의 형식은 컬렉터 결과 형식과 같으므로 변환과정이 필요없다.

그러므로 항등 함수 identity를 반환하도록 finisher 메서드를 구현한다.

- finisher 메서드
```java
public Function<Map<Boolean, List<Integer>>,
                Map<Boolean, List<Integer>>> finisher(){
    return Function.identity();
}
```

커스텀 컬렉터는 `CONCURRENT`도 아니고 `UNORDERED`도 아니지만    
`IDENTITY_FINISH`이므로 다음과 같이 구현한다

- caracteristics 메서드
```java
public Set<Characteristics> Characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));
```
### 🎈 6단계 : 마무리 적용
이제 partitioningBy 메서드에 우리의 커스텀 컬렉터를 적용해보자.

```java
public Map<Boolean, List<Integer>>
                          partitionPrimesWithCustomCollector(int n){
    return IntStream.rangeClosed(2, n).boxed()
                    .collect(new PrimeNumbersCollector());
```

위 코드를 간단한 벤치마킹으로 체크했을때 

백만 개의 자연수 분류를 10번 반복하며 진행한 결과 커스 클래스가 약 32퍼 향상되었다.

### 📌 6.7 마치며

- collect는 스트림의 요소를 요약 및 결과로 누적하는 다양한 방법(컬렉터)을 인수로 갖는 최종 연산이다
- 스트림의 요소를 하나의 값으로 리듀스하고 요약하는 컬렉터 그리고 최솟값, 최댓값, 평균값을 계산하는 컬렉터가 미리 정의되어 있다.
- `groupingBy`라는 컬렉터로 스트림 요소를 그룹화 하거나, `partitioningBy`로 스트림 요소를 분할할 수 있다.
- 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다.
- Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발할 수 있다.
