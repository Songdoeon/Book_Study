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

[그림 6-1]


### 🎈 미리 정의된 컬렉터
이번에는 미리 정의된 컬렉터, 즉 groupingBy 같이 Collectors 클래스에서 제공하는 팩토리 메서드의 기능을 설명한다.

Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 구분할 수 있다.

- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

## 📚 LESSEN 2 : 리듀싱과 요약
