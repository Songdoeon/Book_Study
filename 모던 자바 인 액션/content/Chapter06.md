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

### 📌 reduce vs collect
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
int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum();
```

컬렉터를 이용하는 코드는 조금 더 복잡할 수 있지만 재사용성과 커스터마이징의 가능성이 조금 더 높다.

대신, IntStream을 사용하면 언방식 비용 절약까지하여 성능과 가독성까지 챙겨갈 수 있다.

## 📚 LESSEN 3 : 그룹화
