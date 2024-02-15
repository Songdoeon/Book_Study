## 🌈 Chapter 7 : 병렬 데이터 처리와 성능

자바 7 이전에는 데이터 컬렉션의 병렬처리가 까다로웠지만, 자바 7 부터는 병렬화를 수행하며 에러를 최소화 하는 **포크/조인 프레임워크** 기능을 제공한다.

이 장에서는 스트림으로 데이터 컬렉션 관련 동작을 얼마나 쉽게 병렬로 실행하는지,   
스트림을 이용한 순차 스트림의 병렬 스트림으로의 자연스러운 변환 등을 알아본다.

우선 여러 청크를 병렬로 처리하기 전에 병렬 스트림이 요소를 여러 청크로 분할하는 방법을 설명할 것이다.

이 원리를 이해하지 못하면 의도치 않은, 설명하기 어려운 결과가 발생할 수 있다.

## 📚 LESSON 1 : 병렬 스트림
4장에서 스트림을 이용해 아주 간단하게 요소를 병렬 처리가 가능하다 했다.

컬렉션에 `parallelStream`을 호출하면 **병렬 스트림**이 생성된다.
  > 병렬 스트림 : 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림

- 1 ~ N 까지의 수의 합을 반환하는 메서드
```java
public long sequentialSum(long n){
  return stream.iterate(1L, i -> i + 1)    // 무한 자연수 스트림 생성
                .limit(n)
                .retuce(0L, Long::sum);
}
```
n이 커진다면 이 연산, 병렬로 처리하는 것이 좋을 것이다.

자 그럼 병렬스트림에 대해 알아보도록 하자.

### 🎈 병렬 스트림으로 변환하기

- parallel 메서드
```java
public long sequentialSum(long n){
  return stream.iterate(1L, i -> i + 1)  
                .limit(n)
                .parallel()    // 스트림을 병렬 스트림으로 변환
                .retuce(0L, Long::sum);
}
```
이전 코드와 다른 점은 스트림이 여러 청크로 분할되어 있다는 것이다.

<img width="600" src="https://github.com/Songdoeon/Book_Study/assets/96420547/82710d87-af61-46c6-b6b0-81505e9e7d9e">

리듀싱 연산을 여러 청크에 병렬로 수행한 후 마지막 리듀싱 연산으로 생성된 부분 결과를    
다시 리듀싱 연산으로 함쳐 전체 스트림의 리듀실 결과를 도출한다.

사실 순차 스트림에 `parallel`을 호출하면 스트림이 변하는 것이 아닌 내부적으로 병렬 수행 Flag가 설정되는것이다.

반대로 `sequential`로 병렬 스트림을 순차 스트림으로 바꿀 수 있기도 하다.

두 메서드 중 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미친다.

### 📌 병렬 스트림에서 사용하는 스레드 풀 설정
병렬 스트림은 내부적으로 `ForkJoinPool`을 사용한다

기본적으로 `ForkJoinPool`은 프로세서 수, 즉 Runtime.getRuntime().availableProcessors() 가 반환하는 값에 상응하는 스레드를 갖는다.

- System.setProperty("java.util.concureent.ForkJoinPool.common.parallelism", "12")
  - 이 예제는 전역 설정 코드이므로 이후 모든 병렬 스트림 연산에 영향을 준다.
  - 현재는 하나의 병렬 스트림에 사용할 수 있는 특정한 값을 지정할 수 없다.
  - 일반적으로 기기의 프로세서 수와 같으므로 특별한 이유가 없다면 기본값을 사용하자.

### 🎈 스트림 성능 측정
병렬화를 이용하면 순차, 반복 형식에 비해 성능이 좋아질 것이다.

그러나 성능 최적화의 세 가지 규칙이 있다. 

바로 첫째도 측정, 둘째도 측정, 셋째도 측정!!

그러므로 우리는 JMH라는 자바 마이크로 벤치마크를 이용해 간단하고, 어노테이션 기반 방식으로 구현할 수 있다.

사실 JVM에서 벤치마크 작업은 쉽지 않다.   
바이트코드 최적화, 가비지 컬렉터로 인한 오버헤드 등과 같은 요소를 고려해야 하기 때문이다.

Maven등 의존성 관리 도구를 사용한다면, 의존성을 추가해 프로젝트에서 JMH를 사용할 수 있다.

- n개의 숫자를 더하는 함수의 성능 측정 예제 코드
```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Fork(2, jvmArgs={"-Xms46", "-Xmx4G"})
public class ParallelStreamBenchmark {
  private static final long N = 10_000_000L

  @Benchmark    // 벤치마크 대상 메서드
  public long sequentialSum() {
    return Stream.iterate(1L, i -> i + 1).limit(N)
                  .reduce(0L, Long::sum);
  }

  @TearDown(Level.Invocation)    // 매 번 벤치마크를 실행한 다음에는 가비지 컬렉터 동작 시도
  public void tearDown(){
    System.gc();
  }
}
```

위 코드를 실행할 때 JMH 명령은 핫스팟이 코드를 최적화 할 수 있도록 20번을 실행하며 벤치 마크를 준비한 다음   
20번을 더 실행해 최종 결과를 계산한다.

즉, JMH는 기본적으로 20 + 20회 프로그램을 반복 실행한다.  
이는 특정 어노테이션이나 `-w`, `-i` 플래그를 명령행에 추가해 횟수를 조절할 수 있다.

for문을 이용한다면 stream보다 약 4배정도 빠르다.

자 그러면 이제 병렬 스트림의 예제를 살펴보자
```java
@Benchmark
public long iterativeSum() {
  long result = 0;
  for(long i = 1L <= N; i++) {
    result += i;
  }
  return result;
}
```

자 그러면 결과도 한번 확인해보자.
```java

Benchmark                                Mode    Cnt    Score        Error   Units
ParallelStreamBenchmark.sequentialSum    avgt     40    121.843  +-  3.062   ms/op


Benchmark                                Mode    Cnt    Score        Error   Units
ParallelStreamBenchmark.parallelSum      avgt     40    604.059  +-  55.288  ms/op
```
병렬 버전이 쿼드코어 CPU를 활용하지 못하고 순차 버전에 비해 다섯 배나 느린 실망스러운 결과가 나왔다.

이 결과를 어떻게 설명할까? 다음과 같은 문제가 있다.

- 반복 결과로 박싱된 객체가 만들어지므로 숫자를 더하려면 언박싱을 해야 한다.
- 반복 작업은 병렬로 수행할 수 있는 독립 단위로 나누기가 어렵다.

두 번째 문제는 예사롭게 넘길 수 없다. 

우리에겐 병렬 스트림 모델이 필요하기 때문이다.  
특히 이전 연산의 결과에 따라 다음 함수의 입력이 달라지기 때문에 iterate 연산을 청크로 분할하기가 어렵다.

이처럼 병렬 프로그래밍은 까다롭고 때로는 이해하기 어려운 함정이 숨어 있다.   
심지어 병렬 프로그래밍을 오용하면 오히려 전체 프로그램의 성능이 더 나빠질 수도 있다.

따라서 마법 같은 parallel 메서드를 호출했을 때 내부적으로 어떤 일이 일어나는지 꼭 이해해야 한다.

### 📕 특화된 메서드 사용
멀티코어 프로세서를 활용해 효과적으로 합계 연산을 병렬로 실행하려면 어떻게 해야 할까?

5장에서 `LongStream.rangeClosed`라는 메서드를 소개했다.   
이 메서드는 iterate에 비해 다음과 같은 장점을 제공한다.

- 기본형 long을 직접 사용하므로 박싱과 언박싱 오버헤드가 사라진다.
- 쉽게 청크로 분할할 수 있는 숫자 범위를 생산한다.

그렇다면 언박싱 관련 오버헤드가 얼마나 될까? 한 번 알아보도록 하자.

- 벤치마크 할 메서드
```java
@Benchmark
public long rangedSum() {
  return LongStream.rangeClosed(1, N)
                    .reduce(0L, Long::sum);
}

@Benchmark
public long parallelRangedSum() {
  return LongStream.rangeClosed(1, N)
                    .parallel()
                    .reduce(0L, Long::sum);
}
```
- 결과
```java

Benchmark                                    Mode    Cnt    Score        Error   Units
ParallelStreamBenchmark.rangedSum            avgt     40     5.315  +-   0.285   ms/op


Benchmark                                    Mode    Cnt    Score        Error   Units
ParallelStreamBenchmark.parallelrangedSum    avgt     40     2.677  +-   0.214   ms/op
```

자 드디어 병렬 리듀싱을 만들었다.  

결국 함수형 프로그래밍을 올바르게 사용하면 반복적인 코드에 비해    
최신 멀티코어 CPU가 제공하는 병렬 실행의 힘을 단순하게 직접적을 얻을 수 있다.

하지만 완전한 공짜는 아닌, 스트림을 재귀적으로 분할하고, 각 서브스트림을 서로 다른 스레드의 리듀싱 연산을 할당    
그리고 이들의 결과를 하나의 값으로 합쳐야 한다.

멀티코어 간의 데이터 이동은 우리 생각보다 비싸다.   
따라서 코어 간 데이터 전송 시간보다 훨씬 오래 걸리는 작업만 병렬로 다른 코어에서 수행하는것이 바람직하다.

병렬스트림을 사용하고싶다면 항상 병렬화를 올바르게 사용하고 있는지 확인해야 한다.

### 🎈 병렬 스트림의 올바른 사용법

병렬 스트림을 사용할 때는 공유 변수를 주의해야한다고 계속 말했다.

자 그러면 다음 예제를 살펴보자.

- n까지의 자연수를 누적자로 바꾸는 코드
```java
public long sideEffectParallelSum(long n) {
  Accumulator accmulator = new Accumulator();
  LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
  return accmuator.total;
}

public class Accumulator {
  public long total = 0;
  public void add(long value) { total += value }
}

System.out.println("SideEffect sum done in : " +
                    measurePerf(ParallelStreas::sideEffectParallelSum, 10_000_000L) + " msecs");
```

딱히 특별할것 없는 코드같다. 

하지만 parallel이 붙어있는 다중 스레드 환경에서 total에 동시에 접근한다면 문제가 생길 것이다. 

코드를 한번 실행 해보자.

``` java
Result : 59599989000692
Result : 61247832652839
Result : 62447834783265
Result : 73753834783265
Result : 92571025837293
Result : 63425372456743
Result : 69525835710258
Result : 62478347832652
Result : 97449934634472
Result : 82542346136085
    SideEffect parallel sum done in : 49 msecs
```

메서드의 성능은 둘때고, 올바른 결과값(50000005000000)이 나오지 않는다.

상태 공유 변수에 따른 부작용이다.

### 🎈 병렬 스트림 효과적으로 사용하기
천 개 이상의 요소가 있을 때만 병렬 스트림을 사용하라 처럼 양을 기준으로 병렬 스트림 사용을 결정하는 것은 적절하지 않다.

정해진 기기에서 정해진 연산을 수행할 때는 이러한 기준을 사용할 수 있지만 상황이 달라지면 기준이 제 역할을 못할 것이다.

그래도 어느정도는 수량적 힌트를 정하는 것이 도움이 될 때도 있다.

병렬 스트림을 고려할때 생각해볼 점은 다음과 같다.

- 직접 측정하라.
  - 병렬 스트림의 과정은 투명하지 않을 때가 많다.
  - 잘 모르겠다면 적절한 벤치마크를 이용해 직접 측정하자.
- 박싱을 주의하라.
  - 자동 박싱, 언박싱은 성능을 크게 저하시킬 수 있는 요소다.
  - 박싱을 피할 수 있도록 기본형 특화 스트림을 사용하자.
- 순차 스트림보다 병렬 스트림이 성능이 떨어지는 연산이 있다.
  - `limit`, `findFirst` 처럼 요소의 순서에 의존하는 연산은 병렬 스트림에 적절하지 않다.
  - 정렬된 스트림에 `unordered`를 호출하면 비정렬된 스트림을 얻을 수 있다.
  - 스트림에 N개 요소가 있을 때 요소의 순서가 상관없다면, 비정렬된 스트림에 `limit` 호출이 더 효율적이다.
- 스트림 전체 연산 비용을 고려하라.
  - 처리할 요소 수가 N 이고 하나의 요소 처리비용을 Q라고 하면 전체 파이프 라인 처리비용은 N*Q쯤 일것이다.
  - Q가 높아진다는 것은 벙렬 스트림으로 성능을 개선할 수 있는 가능성이 있음을 의미한다.
- 소량의 데이터는 병령 스트림이 도움 되지 않는다.
  - 소량의 데이터 처리는 병렬화 과정의 부가 비용을 상쇄할 수 있을 만큼 이득을 얻지 못할 것이다.
- 컬렉션의 자료구조가 적절한지 확인하라.
  - 분할의 경우 `LinkedList`보다 `ArrayList`가 효율적이다.
  - `LinkedList`는 분할 시 모든 요소를 탐색해야 하지만 `ArrayList`는 그렇지 않다.
  - 또한 range 메서드로 만든 기본형 스트림도 쉽게 분해 가능하다.
- 스트림 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라진다.
  - SIZED 스트림은 정확히 같은 크기의 두 스트림을 분할할 수 있으므로 효과적인 병렬 처리가 가능하다.
  - 반면 필터 연산이 있으면 스트림 길이를 예측할 수 없어 효과적으로 처리할 수 없다.
- 최종 연산의 병합 과정 비용을 살펴보자.
  - 병합 과정의 비용이 비싸다면 병렬 스트림으로 얻는 이익이 서브스트림의 부분 결과를 합치는 과정에 상쇄될 수 있다.
  - Ex) `Collector.combiner`

|소스|분해성|
|--|--|
|ArrayList|훌륭함|
|LinkedList|나쁨|
|IntStream.range|훌륭함|
|Stream.iterate|나쁨|
|HashSet|좋음|
|TreeSet|좋음|

## 📚 LESSON 2 : 포크/조인 프레임워크
포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음   
서브태스크 각각의 결과를 합쳐 전체 결과를 만들도록 설계되었다.

포크/조인 프레임워크에서는 서브태스크를 스레드풀(ForkJoinPool)의 작업자 스레드에 분산 할당하는   
`ExecutorService`인터페이스를 구현한다.

### 🎈 RecursiveTask 활용
스레드 풀을 이용하려면 `RecursiveTask <R>`의 서브클래스를 만들어야 한다.

여기서 R은 병렬화된 태스크가 생성하는 결과 형식 또는 결과가 없을 때는 RecursiveAction 형식이다.   
RecursiveTask를 정의하려면 추상 메서드 compute를 구현해야 한다.
```java
protected abstract R compute();
```

compute 메서드는 태스크를 서브태스크로 분할하는 로직과 더 이상 분할할 수 없을 때   
개별 태서브태스크의 결과를 생산할 알고리즘을 정의한다.

- 대부분의 compute 메서드의 의사코드 형식
```java
if ( 태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
  순차적으로 태스크 계산
} else {
  태스크를 두 서브 태스크로 분할
  태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출
  모든 서브태스크의 연산이 완료될 때까지 기다림
  각 서브태스크의 결과를 합침
}
```
![image](https://github.com/Songdoeon/Book_Study/assets/96420547/4d9c6130-17f0-4823-bec5-46d65c0f6735)

이 알고리즘은 분할 정복(devide and conquer) 알고리즘의 병렬화 버전이다.

포크/조인 프레임워크를 이용해 범위의 숫자를 더하는 문제를 구현하며 사용법을 확인하자.
