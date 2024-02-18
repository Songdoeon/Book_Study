## 🌈 Chapter 8 : 컬렉션 API 개선
<details><summary>정리</summary>
  
```
- 자바 9는 적은 원소를 포함하며 바꿀 수 없는 리스트, 집합 맵을 쉽게 만들도록 지원한다.
- 이들 컬렉션 팩토리가 반환한 객체는 만들어진 다음 바꿀 수 없다.
- List 인터페이스는 `removeIf`, replaceAll`, `sort` 세 가지 디폴트 메서드를 지원한다.
- Set 인터페이스는 `removeIf` 디폴트 메서드를 지원한다.
- Map 인터페이스는 주로 쓰는 패턴과 버그 방지를 위해 다양한 디폴트 메서드를 지원한다.
- `ConcurrentHashMap`은 Map 에서 상속받은 새 디폴트 메서드를 지원하고 thread safety하다.
```

</details>

지금까지 컬렉션과 스트림 API를 이용해 데이터 처리 쿼리를 어떻게 효율적으로 처리할 수 있는지 살펴봤다.

하지만 여전히 컬렉션 API에는 성가시며, 에러를 유발하는 여러 단점이 존재한다.

이번 장에서는 자바 8, 9에서 추가된 컬렉션 API의 기능을 배운다.

## 📚 LESSEN 1 : 컬렉션 팩토리
자바 9에서는 작은 컬렉션 객체를 쉽게 만들 수 있는 몇 가지 방법을 제공한다.

우선 왜 이 기능이 필요한지 살펴본 다음, 새 팩토리 메서드를 사용하는 방법을 설명한다.

자바에서 적은 요소를 포함하는 리스트를 보통 어떻게 만드는가?

- 휴가를 보내려는 친구 리스트 만들기
```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```
- Arrays.asList() 팩토리 메서드 사용
```java
List<String> friends
                = Arrays.asList("Raphael", "Olivia", "Thibaut");
```
한 줄로 간단하게 만들었지만 고정크기의 리스트로 만들었으므로 요소 갱신은 가능하지만, 요소를 추가하거나 삭제할 수 없다.

요소를 추가하려 하면 `Unsupported OperationException`이 발생할 것이다.

### 📌 Unsupported OperationException 예외 발생
내부적으로 고정된 크기의 변활할 수 없는 배열로 구현되었기 때문에 이와 같은 일이 일어난다.

그럼 집합은 어떨까? 안타깝지만 `Arrays.asSet()` 이라는 팩토리 메서드는 없으므로 다른 방법을 찾아보자.

그럼 맵은 어떨까? 작은 맵을 만들 수 있는 멋진 방법은 따로 없지만   
자바 9에서는 작은 리스트, 집합, 맵을 쉽게 만들 수 있도록 팩토리 메서드를 제공한다.

### 🎈 리스트 팩토리
`List.of` 팩토리 메서드를 이용해 간단히 리스트를 만들 수 있다.

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");

System.out.println(friends);    // ["Raphael", "Olivia", "Thibaut"]
```

하지만 사실 이렇게 생성해도 요소 추가시 에러가 발생한다.   
set() 메서드도 마찬가지다.

책에서는 이런방식이 컬렉션이 의도치 않게 변하는것을 막기위해 리스트를 바꿔야 한다면 직접 리스트를 만들면 된다고 한다.  
그리고 null요소를 금지하여 버그를 방지하고 간결한 내부구현을 구현했다.

하지만 요소 자체의 변화를 막지는 못하고 추가와 삭제만 막았다는 점이 개인적으로 썩 그렇게 마음에 들진 않는다.

어떤 상황에서 새로운 컬렉션 팩토리 메서드 대신 스트림 API를 사용해 리스트를 만들어야 하는지 궁금할 수도 있다.

7장에서 살펴본것처럼 `Collectors.toList()`로 스트림을 리스트로 변환할 수 있다.   
하지만 데이터 처리 형식을 설정하거나 변환할 필요가 없다면 사용하기 편한 팩토리 메서드를 이용할 것을 권장한다.

### 📌 오버로딩 vs 가변 인수

List 인터페이스 내부는 다음과 같은 식으로 구현되어있다.

- `List.of`의 내부 구현
```java
// 실제 구현
static <E> List<E> of(E e1, E e2, E e3, E e4);
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5);

// 다른 방식을 생각했을 경우의 코드
static <E> List<E> of(E... elements);
```

이 코드를 보며 왜 다음과 같이 구현하지 않을까 생각한 사람이 있을 수 있다.

내부적으로 가변 인수 버전은 추가 배열을 할당하여 리스트로 감싼다.   
따라서 배열을 할당하고 초기화 하며 나중에 GC 비용까지 지불해야 한다.

하지만 10개 이상의 요소로 List.of로 열 개 이상의 요소를 가진 리스트를 만들 경우에는 가변 인수를 이용하는 메서드가 사용된다.

이는 `Set.of`, `Map.of` 에서 같은 패턴이 등장한다

### 🎈 집합, 맵 팩토리

- `Set.of` 메서드
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");

System.out.println(friends);    // ["Raphael", "Olivia", "Thibaut"]
```

중복된 요소를 제공하면 `IllegalArgumentsException`이 발생한다.

- `Map.of` 메서드
```java
Map<String, Integer> ageOfFriends =
                  Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);

System.out.println(ageOfFriends);    // {"Raphael=30", "Olivia=25", "Thibaut=26"}
```
열 개 이하의 키와 값 쌍을 가진 작은 맵을 만들 때는 이 메소드가 유용하다.

그 이상의 맵은 다음과 같이 구현하도록 하다.

- `Map.ofEntries` 메서드
```java
import static java.util.Map.entry;

Map<String, Integer> ageOfFriends =
                  Map.ofEnties(entry("Raphael", 30),
                               entry("Olivia", 25),
                               entry("Thibaut", 26);

System.out.println(ageOfFriends);    // {"Raphael=30", "Olivia=25", "Thibaut=26"}
```

## 📚 LESSEN 2 : 리스트와 집합 처리
자바 8에서는 List, Set 인터페이스에 당므과 같은 메서드를 추가했다.

- `removeIf`
  - Predicate를 만족하는 요소를 제거한다.
  - List, Set을 구현하거나 상속받는 모든 클래스에서 이용 가능하다.
- `replaceAll`
  - List 에서 사용할 수 있는 기능으로 `UnaryOperator`함수를 이용해 요소를 바꾼다
- `sort`
  - List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다.
 
위 메서드는 호출한 컬렉션 자체를 바꾼다.  
컬렉션을 바꾸는 동작은 에러를 유발하지만 왜 이런 메서드가 추가되었을까?

### 🎈 removeIf 메서드
다음은 숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드다.

```java
for(Transaction transaction : transactions) {
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))){
    transactions.remove(transaction);
  }
}
```

하지만 위 코드는 `ConcurrentModificationException`을 일으킨다.

이유는 내부적으로 for-each 루프는 `Iterator`객체를 사용하므로 위 코드는 다음과 같다.

```java
Iterator<Transaction> iterator =  transactions.iterator();
while(iterator.hasNext()) {
  Transaction transaction = iterator.next();
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))){
    transactions.remove(transaction);
  }
}
```

두 개의 개별 객체가 컬렉션을 관리한다는 사실을 주목하자.
- `Iterator`
  - `next()`, `hasNext()`를 이용해 소스를 질의한다.
- `Collection`
  - `remove()`를 호출해 요소를 삭제한다.
 
결과적으로 반복자의 상태는 컬렉션의 상태와 서로 동기화 되지 않는다.

Iterator객체를 명시적으로 사용하고 그 객체에 `remove()`메서드를 호출함으로 이 문제를 해결할 수 있다.
```java
Iterator<Transaction> iterator =  transactions.iterator();
while(iterator.hasNext()) {
  Transaction transaction = iterator.next();
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))){
    iterator.remove();
  }
}
```
코드가 조금 복잡해졌는데 이 코드 패턴을 자바8의 `removeIf`메서드를 이용해 바꿀 수 있다.

그러면 코드가 단순해지며, 버그도 예방할 수 있다.

- `removeIf`를 사용한 코드
```java
transactions.removeIf(transaction ->
                  Character.isDigit(transaction.getReferenceCode().charAt(0)));
```
하지만 때로는 요소를 제거하는게 아닌 바꿔야 할 때가 있다.

그럴때 `replaceAll`를 사용한다.

### 🎈 removeIf 메서드
`replaceAll` 메서드의 개발 이유를 살펴보자.

- 스트림 API 사용
```java
referenceCodes.stream()
              .map(code -> Chracter.toUpperCase(code.charAt(0)) + code.substring(1))
              .collect(Collectors.toList())
              .forEach(System.out::println);
```

하지만 위 코드는 새 문자열 컬렉션을 만든다. 우리가 원하는 것은 기존 컬렉션을 바꾸는 것이다.

- ListIterator 객체 사용
```java
Iterator<String> iterator =  referenceCodes.iterator();
while(iterator.hasNext()) {
  String code = iterator.next();
  iterator.set(Chracter.toUpperCase(code.charAt(0)) + code.substring(1));
}
```

- `replaceAll` 사용
```java
referenceCodes.replaceAll(code -> Chracter.toUpperCase(code.charAt(0)) + code.substring(1)));
```

## 📚 LESSEN 3 : 맵 처리
자바 8에서는 Map 인터페이스에 몇 가지 디폴트 메서드를 추가했다.

자주 사용되는 패턴을 개발자가 직접 구현할 필요가 없도록 이들 메서드를 추가했다.

### 🎈 forEach 메서드
```java
for(Map.Entry<String, Integer> entry : ageOfFriends.entrySet()) {
  System.out.println(entry.getKey + " : " + entry.getValue());
}

// 다음과 같은 방식도 가능하다.
ageOfFriends.forEach((friend, age) -> System.out.println(friends + " is " + age + " years old"));
```

### 🎈 정렬 메서드
```java
Map<String, String> favoriteMovies =
                        Map.ofEntries(entry("Raphael", "Start wars"),
                                      entry("Cristina", "Matrix"),
                                      entry("Olivia", "James Bond"));

favoriteMovies
    .entrySet()
    .stream()
    .sorted(Entry.comparingByKey())
    .forEachOrdered(System.out::println);    // 사람 이름을 알파벳순으로 정렬하여 처리
```

### 🎈 계산 패턴
맵에 키의 존재여부에 따라 결과값을 저장해야할 때가 있을 수 있다.

다음 세 가지 연산이 이런 상황에서 도움을 준다.

- `computeIfAbsent`
  - 제공된 키에 해당하는 값이 없으면(값이 없거나 null), 키를 이용해 새 값을 계싼하고 맵에 추가한다.
- `computeIfPresent`
  - 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
- `compute`
  - 제공된 키로 새 값을 계산하고 맵에 저장한다.

자 그럼 예제를 살펴보자.

- `computeIfAbsent` 예제
```java
Map<String, byte[]> dataToHash = new HashMap<>();
MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");

line.forEach(line ->
      dataToHash.computeIfAbsent(line, this::calculateDigest));    // 키가 없을시 동작을 실행

private btye[] calculatorDigest(String key){
  return messageDigest.degiest(key.getBytes(StandardCharsets.UTF_8));
}
```

### 🎈 삭제, 교체 패턴
- remove : `favoriteMovies.remove(key, value)`
- replaceAll : `favoriteMovies.replaceAll((friend, movie) -> movie.toUpperCase())`
- Replace : 키가 존재하면 맵의 값을 바꾼다.

### 🎈 합침
두개의 맵을 합친다고 하면 `putAll`을 사용할 수 있다.   
하지만 중복값이 있을 경우 문제가 생기는데 이 점을 `merge` 메서드를 이용할 수 있다.

- 맵 두개를 합치는 코드
```java
Map<String, Long> moviesToCount = new HashMap<>();
String movie Name = "JamesBond";
long count = moviesToCount.get(MovieName);

if(coint == null) moviesToCount.put(movieName, 1);
else moviesToCount.put(movieName, count + 1);

// merge를 이용한 방법
moviesTocOunt.merge(movieName, 1L, (count, increment) -> count + 1L);
```

## 📚 LESSEN 4 : 개선된 ConcurrentHashMap
`ConcurrentHashMap` 클래스는 동시성 친화적이며, 최신 기술을 반영한 HashMap 버전이다.

내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.   
따라서 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산이 월등하다.

### 🎈 리듀스와 검색
`ConcurrentHashMap`은 스트림에서 본 종류와 비슷한 연산을 지원한다.

- `forEach` : 각 (키, 값)쌍에 주어진 액션을 실행
- `reduce` : 모든 (키, 값)쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- `search` : 널이 아닌 값을 반환할 때까지 각 (키, 값)쌍에 함수를 적용

이들 연산은 상태를 잠그지 않고 연산을 수행한다.

따라서 이들 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야한다.

또 이들 연산에 병렬성 기준값을 지정해야 한다.

- 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다.   
- 기준값을 1로 지정하면 공통 스레드 풀을 이용해 병렬성을 극대화 한다.
- Long.MAX_VALUE를 기준값으로 설정하면 한 개의 스레드로 연산을 실행한다.

우리의 소프트웨어 아키텍처가 고급 수준의 자원 활용 최적화를 사용하고 있지 않다면, 기준값 규칙을 따르는 것이 좋다.

- `reduceValues` 메서드를 이용한 맵의 최댓값 찾기
```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThresold = 1;

Optional<Integer> maxValue =
                Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

int, long, double 등의 기본값에는 전용 each reduce 연산이 제공되므로    
`reduceValuesToInt`, `reduceKeysToLong` 등을 이용하면 박싱 작업 없이 효율적으로 처리 가능하다.

### 🎈 계수, 집합 뷰
`ConcurrentHashMap` 클래스는 맵의 매핑 개수를 반환하는 `mappingCount` 메서드를 제공한다.

기존의 size 와 다른점은 매핑의 개수를 int가 아닌 long을 반환한다는 것이다.

그리고 `ConcurrentHashMap` 를 집합 뷰로 반환하는 `keySet` 이라는 새 메서드를 제공한다.   
맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다.

`newKeySet`이라는 메서드를 이용해 `ConcurrentHashMap`으로 유지되는 집합을 만들 수도 있다.
