## 🌈 Chapter 12 : 새로운 날짜와 시간 API
자바 8에서는 지금까지의 날짜와 시간 문제를 개선하는 새로운 날짜와 시간 API를 제공한다.

자바 1.0에서는 `java.util.Date` 클래스 하나로 날짜와 시간 관련 기능을 제공했다.   
날짜를 의미하는 Date의 의미와는 달리 특정 시점을 날짜가 아닌 밀리초 단위로 표현했다.

다음은 자바 9의 릴리스 날짜인 2017년 9월 21일을 가리키는 Date 인스턴스를 만드는 코드다.
```java
Date date = new Date(117, 8, 21);

// 출력 결과
Thu Sep 21 00:00:00 CET 2017
```
결과가 직관적이지 않고 Date는 JVM의 기본시간대인 CET(중앙 유럽)시간대를 사용했다.

문제가 많지만 과거 버전과의 호환성을 깨트리지 않으면서 이를 해결할 방법을 찾지 못했다.   
그래서 `Calendar`라는 클래스를 제공하는 등의 방법을 시도하였지만 그 역시도 문제를 해결하지 못했다.

더 안타까운 점은 두 클래스가 등장하며 개발자들의 혼란을 야기했고,    
`DateFormat`같은 일부 기능은 Date 클래스에만 작동했다.

`DateFormat`마저도 Thread safety하지 않는 등의 문제가 있었으며,  
`Date`, `Calendar` 모두 가변 클래스로 FP의 관점에서 유지보수에 어려움이 있었다.

자 그러면 우리는 자바에서 새롭게 제공하는 날짜와 시간 API를 살펴보도록 하겠다.

## 📚 LESSEN 1 : LocalDate, LocalTime, Instant, Duration, Period 클래스
먼저 간단한 날짜와 시간 간격을 정의해보겠다.

java.time 패키지는 `LocalDate`, `LocalTime`, `Instant`, `Duration`, `Period`등의 새로운 클래스를 제공한다.

### 🎈 LocalDate와 LocalTime
새로운 날짜와 시간 API를 사용할 때 처음 접하게 되는 것이 `LocalDate`다.    
`Loca Date` 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다.

특히 `LocalDate` 객체는 어떤 시간대 정보도 포함하지 않는다.

정적 팩토리 메서드 of로 LocalDate를 만들 수 있다.   
Local Date 인스턴스는 연도, 달, 요일 등을 반환하는 메서드를 제공한다.

- LocalDate 만들고 값 읽기
```java
LocalDate date = LocalDate.of(2017, 9, 21);  // 2017-08-21
int year = date.getYear();              // 2017
Month month = date.getMonth();          // SEPTEMBER    
int day = date.getDayOfMonth();         // 21
DayOfWeek dow = date.getDayOfWeek();    // THURSDAY
int len = date.lengthOfMonth();         // 31 (3월의 일 수)
boolean leap = date.isLeapYear();       // false (윤년이 아님)
```

팩토리 메서드 `now`는 시스템시계의 정보를 이용해 현재 날짜 정보를 얻는다.
```
LocalDate today = LocalDate.now();
```

### 📕 열거자 ChronoField
`ChronoField`클래스는 `TemporalField`라는 시간 관련 인터페이스를 정의하므로 다음 코드처럼   
열거자 요소를 이용해 원하는 정보를 쉽게 얻을 수 있다.
- TemporalField를 이용해 LocalDate값 읽기
```java
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);

// 하지만 내장 메서드를 쓰는게 가독성이 좋을 것이다.
int year = date.getYear();              // 2017
Month month = date.getMonth();          // SEPTEMBER    
int day = date.getDayOfMonth();         // 21
//
```

### 📕 LocalTime 클래스
`LocalTime` 클래스로는 **13:45:20** 같은 시간을 표현할 수 있다.

오버로드 버전의 두 가지 정적 메서드 of로 `LocalTime`인스턴스를 만들 수 있다.   
즉, 시간과 분을 인수로 받는 of 메서드와 시간과 분, 초를 인수로 받는 of 메서드가 있다.

- LocalTime 만들고 값 읽기
```java
LocalTIme time = LocalTime.of(13, 45,20)  // 13:45:20
int hour = time.getHour();      // 13
int minute = time.getMinute();  // 45
int second = time.getSecond();  // 20
```

날짜와 시간 문자열로 `LocalDate`와 `LocalTime`의 인스턴스를 만드는 방법도 있다.

- parse 정적 메서드를 이용한 방법
```java
LocalDate date = LocalDate.parse("2017-09-21");

LocalTime time = LocalTime.parse("13:45:20);
```

parse 메서드에 `DateTimeFormatter`를 전달할 수도 있다.

`DateTimeFormatter`는 이전에 말한 `DateFormat`을 대체하는 클래스로    
문자열을 `LocalDate`나 `LocalTime`으로 파싱할 수 없을 때 parse메서드는 `DateTimeParseException`을 일으킨다.

### 🎈 날짜와 시간 조합
`LocalDateTime`은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다.

이름 그대로 날짜와 시간 모두 표현할 수 있다.

- LocalDateTime을 만드는 법과 날짜와 시간을 조합하는 법
```java
// 2017-09-21T13:45:20
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

### 🎈 Instant 클래스 : 기계의 날짜와 시간
보통 사람은 주, 날짜, 시간, 분으로 날짜와 시간을 계산하지만 기계는 다르다.

새로운 `java.time.Instant`클래스에서는 이와 같은 기계적인 관점에서 시간을 표현한다.   
즉, `Instant`클래스는 유닉스 에포크 시간을 기준으로 특정 지점까지의 시간을 초로 표현한다.
  > 유닉스 에포크 시간 : 1970년 1월 1일 0시 0분 0초 UTC

팩토리 메서드 ofEpochSecond에 초를 넘겨 `Instant` 클래스 인스턴스를 만든다.

`Instant`클래스는 나노초(10억분의 1초)의 정밀도를 제공한다.

- ofEpochSecond 호출 코드
```java
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(3, 0);
Instant.ofEpochSecond(2, 1_000_000_000);  // 2초 이후의 1억 나노초(1초)
Instant.ofEpochSecond(4, -1_000_000_000); // 4초 이전의 1억 나노초(1초)

// UnsupportedTemporalTypeException을 일으키는 코드
int day = Instant.now().get(ChronoField.DAY_OF_MONTH);  // 기계 전용이라 사람이 읽을 시간 정보를 제공하지 않는다.
```

### 🎈 Duration과 Period 정의
지금까지 살펴본 모든 클래스는 `Temporal` 인터페이스를 구현하는데,   
특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.

이번에는 두 시간 객체 사이의 지속시간 `Duration`을 만들어 보겠다.

- 두 개의 객체로 만드는 Duration
```java
Duration d1 = Duration.between(time1, time2);
Duration d2 = Duration.between(dateTime1, dateTime2);
Duration d3 = Duration.between(instant1, instant2);
```
`Duration`클래스는 초와 나노초로 시간 단위를 표현하므로 between 메서드에 `LocalDate`를 전달할 수 없다.

년, 월, 일로 시간을 표현할 때는 `Period`클래스를 사용한다.

- LocalDate의 기간
```java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11), LocalDate.of(2017, 9, 21));
```

- Duration과 Period 만들기
```java
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);

Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
Period.twoYearsSixMonthsOneDay = Period.of(2, 6, 1);
```

![image](https://github.com/Songdoeon/Book_Study/assets/96420547/c3d132e3-425c-4655-9354-0fffe7b7f391)

지금까지 살펴본 모든 클래스는 불변이다.

불변 클래스는 FP 그리고 thread Safe, 도메인 모델 일관성 유지하는 데 좋은 특징이다.

하지만 새로운 날짜와 시간 API에서는 변경된 객체 버전을 만들 수 있는 메서드를 제공한다.   
예를 들어 기존 `LocalDate` 인스턴스에 3일을 더해야 하는 상황이 발생할 수 있다.

다음 절에서 이방법에 대해 설명하겠다.

## 📚 LESSEN 2 : 날짜 조정, 파싱, 포매팅
`withAttribute` 메서드로 기존의 LocalDate를 바꾼 버전을 직접 간단하게 만들 수 있다.

모든 메서드는 기존 객체를 바꾸지 않고 새로운 객체를 할당하는 방식이다.

- 절대적인 방식으로 LocalDate 속성 바꾸기
```java
LocalDate date1 = LocalDate.of(2017, 9, 21);    // 2017-09-21
LocalDate date2 = date1.withYear(2011);         // 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25);     // 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2);   // 2011-02-25
```

- 상대적인 방식으로 LocalDate 속성 바꾸기
```java
LocalDate date1 = LocalDate.of(2017, 9, 21);    // 2017-09-21
LocalDate date2 = date1.plusWeeks(1);           // 2017-09-28
LocalDate date3 = date2.minuteYears(6)          // 2011-09-28
LocalDate date4 = date3.with(6, ChronoUnit.MONTHS);   // 2012-03-28
```

### 🎈 TemporalAdjusters 사용하기
이번에는 다음주 일요일, 돌아오는 평일 어떤 달의 마지막 날 등 좀 더 복잡한 날짜 조정을 해보자.

- 미리 정의된 TemporalAdjusters
```java
import static java.time.temporal.TemporalAdjusters.*;
LocalDate date1 = LocalDate.of(2014, 3, 18);      // 2014-03-18
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));  // 2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth());   // 2014-03-31
```
![image](https://github.com/Songdoeon/Book_Study/assets/96420547/f53cd31d-7349-4b5d-8540-dfb8e81c289c)

커스텀 TemporalAdjuster 만들기는 생략하도록 하겠다.


### 🎈 날짜와 시간 객체 출력과 파싱
날짜와 시간 관련 작업에서 포매팅과 파싱은 서로 떨어질 수 없는 관계다.

`DateTimeFormatter`는 새로 추가된 클래스로 정적 팩토리 메서드와 상수를 이용해 쉽게 만들 수 있다.

- DateTimeFormatter 예제
```java
LocalDate date = LocalDate.of(2014, 3 ,18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE);    // 20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE);    // 2014-03-18

// 날짜나 시간을 표현하는 문자열을 파싱
LocalDate date1 = LocalDate.parse("20140318",DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date2 = LocalDate.parse("2014-03-18",DateTimeFormatter.ISO_LOCAL_DATE);

// 패턴으로 DateTimeFormatter 만들기
DateTimeFormatter formatter = DAteTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date 1 = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(formatter);
LocalDate date2 = LocalDateparse(formattedDate, formatter);

// 지역화된 DateTimeFormatter 만들기
DateTimeFormatter italianFormatter = DateTimeFormatter.ofPattern("d. MMMM yyyy", Local.ITALIAN;
LocalDate date1 = LocalDate.of(2014, 3, 18);
String formattedDate = date.format(italianFormatter);    // 18. marzo 2014
LocalDate date2 = LocalDate.parse(formattedDate, italianFormatter);

// 빌더 클래스를 이용한 세부적인 제어
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
                                        .appendText(ChronoField.DAY_OF_MONTH)
                                        .appendLiteral(". ")
                                        .appendText(ChronoField.MONTH_OF_YEAR)
                                        .appendLiteral(" ")
                                        .appendText(ChronoField.YEAR)
                                        .parseCaseInsensitive()
                                        .toFormatter(Local.ITALIAN);
```
