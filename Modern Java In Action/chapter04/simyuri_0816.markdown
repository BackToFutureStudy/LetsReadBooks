# Chapter04. 스트림 소개

**이 장의 내용**

- 스트림이란 무엇인가?
- 컬렉션과 스트림
- 내부 반복과 외부 반복
- 중간 연산과 최종 연산

### 4.1 스트림이란 무엇인가?

- 자바 8 API에 새로 추가된 기능
- 선언형(즉, 데이터를 처리하는 임시 구현 코드 대신 질의로 표현할 수 있다.)으로 컬렉션 데이터 처리 가능
- 데이터를 투명하게 병렬로 처리 가능

~~~java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;

List<String> lowCaloricDishesName = 
  	menu.stream()
  			.filter(d -> d.getCalories() < 400)
  			.sorted(comparing(Dish::getCalories))
  			.map(Dish::getName)
  			.collect(toList());
~~~

stream()을 parallelStream()으로 바꾸면 멀티코어 아키텍처에서 병렬로 실행할 수 있다.

~~~java
List<String> lowCaloricDishesName = 
  	menu.parallelStream()
  			.filter(d -> d.getCalories() < 400)
 				.sorted(comparing(Dishes::getCalories))
  			.map(Dish::getName)
  			.collect(toList())
~~~

filter(or sorted, map, collect) 같은 연산은 고수준 빌딩 블록으로 이루어져 있으므로 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황이든 사용할 수 있다.

> **기타 라이브러리: 구아바, 아파치, 람다제이**
>
> 자바 프로그래머가 컬렉션을 제어하는 데 도움이 되는 다양한 라이브러리

자바 8의 스트림 API의 특징

- 선언형: 더 간결, 가독성 좋음
- 조립 가능: 유연성 좋음
- 병렬화: 성능 좋음

### 4.2 스트림 시작하기

스트림이란

- 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소
  - 연속된 요소: 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스 제공, 컬렉션의 주제는 데이터고, 스트림의 주제는 계산
  - 소스: 컬렉션, 배열 I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비. 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지
  - 데이터 처리 연산: 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산 지원. 스트림 연산은 순차적으로 또는 병렬로 실행 가능

스트림 중요 특징

- 파이프라이닝: 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환. -> 게으름, 쇼트서킷 같은 최적화 얻을 수 있음
- 내부 반복

- filter: 람다를 인수로 받아 스트림에서 특정 요소 제외
- map: 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출
- limit: 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소 truncate

- collect: 스트림을 다른 형식으로 변환

~~~java
import static java.util.stream.Collectors.toList;
List<String> threeHighCaloricDishNames = 
  menu.stream()
  		.filter(dish -> dish.getCalorie() > 300)
  		.map(Dish::getName)
  		.limit(3)
  		.collect(toList());
~~~

### 4.3 스트림과 컬렉션

데이터를 언제 계산하는가

- 컬렉션: 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 구조 -> 미리 계산되어야 한다
- 스트림: 이론적으로 요청할 때만 요소를 게산하는 고정된 자료구조

#### 4.3.1 딱 한 번만 탐색할 수 있다

반복자와 마찬가니로 스트림도 한 번만 탐색 가능 -> 탐색된 스트림의 요소는 소비된다.

#### 4.3.2 외부 반복과 내부 반복

컬렉션: 사용자가 직접 요소를 반복해야 한다 -> **외부 반복**

스트림: 함수에 어떤 작업을 수행할지만 지정하면 모든 것이 알아서 처리된다 -> **내부 반복**

내부 반복: 작업을 투명하게 병렬처리하거나 더 최적화된 다양한 순서로 처리 가능

외부 반복: 병렬성을 스스로 관리해야 함

 ##### 퀴즈

코드 리팩터링

~~~java
List<String> highCaloricDishes = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext()) {
  Dish dish = iterator.next();
  if(dish.getCalories() > 300) {
    highCaloricDishes.add(d.getName());
  }
}
~~~

~~~java
List<String> highCaloricDish = menu.stream()
  .filter(dish -> dish.getCalories > 300)
  .map(Dish::getName())
  .collect(toList());
~~~

### 4.4 스트림 연산

**중간 연산**: 연산할 수 있는 스트림 연산

**최종 연산**: 스트림을 닫는 연산

#### 4.4.1 중간 연산

filter, sorted 같은 중간 연산은 다른 스트림을 반환한다. 

-> 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다.

-> 게으르다.(중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리)

#### 4.4.2 최종 연산

스트림 파이프라인에서 결과를 도출

보통 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환된다.

##### 퀴즈

다음 스트림 파이프라인에서 중간 연산과 최종 연산 구별

~~~java
long count = menu.stream()
  .filter(d -> d.getCalories() > 300)
  .distinct()
  .limit(3)
  .count();
~~~

- 중간 연산: filter, distinct, limit
- 최종연산: count

#### 4.4.3 스트림 이용하기

스트림 이용 과정

- 질의를 수행할(컬렉션 같은) 데이터 소스
- 스트림 파이프라인을 구성할 중간 연산 연결
- 스트림 파이프라인을 실행하고 결괄르 만들 최종 연산

| 연산                 | 형식      | 반환 형식      | 연산의 인수                 | 함수 디스크립터     |
| -------------------- | --------- | -------------- | --------------------------- | ------------------- |
| `filter`             | 중간 연산 | `Stream<T>`    | `Predicate<T>`              | `T -> boolean`      |
| `map`                | 중간 연산 | `Stream<R>`    | `Function<T, R>`            | `T -> R`            |
| `mapToInt`           | 중간 연산 | `IntStream`    | `ToIntFunction<T>`          | `T -> int`          |
| `mapToLong`          | 중간 연산 | `LongStream`   | `ToLongFunction<T>`         | `T -> long`         |
| `mapToDouble`        | 중간 연산 | `DoubleStream` | `ToDoubleFunction<T>`       | `T -> double`       |
| `flatMap`            | 중간 연산 | `Stream<R>`    | `Function<T, Stream<R>>`    | `T -> Stream<R>`    |
| `flatMapToInt`       | 중간 연산 | `IntStream`    | `Function<T, IntStream>`    | `T -> IntStream`    |
| `flatMapToLong`      | 중간 연산 | `LongStream`   | `Function<T, LongStream>`   | `T -> LongStream`   |
| `flatMapToDouble`    | 중간 연산 | `DoubleStream` | `Function<T, DoubleStream>` | `T -> DoubleStream` |
| `distinct`           | 중간 연산 | `Stream<T>`    | 없음                        | -                   |
| `sorted()`           | 중간 연산 | `Stream<T>`    | 없음                        | -                   |
| `sorted(Comparator)` | 중간 연산 | `Stream<T>`    | `Comparator<T>`             | `(T, T) -> int`     |
| `peek`               | 중간 연산 | `Stream<T>`    | `Consumer<T>`               | `T -> void`         |
| `limit`              | 중간 연산 | `Stream<T>`    | `long n`                    | -                   |
| `skip`               | 중간 연산 | `Stream<T>`    | `long n`                    | -                   |

| 연산                            | 형식      | 반환 형식          | 연산의 인수               | 함수 디스크립터 |
| ------------------------------- | --------- | ------------------ | ------------------------- | --------------- |
| `forEach`                       | 최종 연산 | `void`             | `Consumer<T>`             | `T -> void`     |
| `forEachOrdered`                | 최종 연산 | `void`             | `Consumer<T>`             | `T -> void`     |
| `toArray`                       | 최종 연산 | `Object[]` / `T[]` | `IntFunction<T[]>` (옵션) | `int -> T[]`    |
| `reduce(identity, accumulator)` | 최종 연산 | `T`                | `BinaryOperator<T>`       | `(T, T) -> T`   |
| `reduce(accumulator)`           | 최종 연산 | `Optional<T>`      | `BinaryOperator<T>`       | `(T, T) -> T`   |
| `collect`                       | 최종 연산 | `R`                | `Collector<T, A, R>`      | -               |
| `min`                           | 최종 연산 | `Optional<T>`      | `Comparator<T>`           | `(T, T) -> int` |
| `max`                           | 최종 연산 | `Optional<T>`      | `Comparator<T>`           | `(T, T) -> int` |
| `count`                         | 최종 연산 | `long`             | 없음                      | -               |
| `anyMatch`                      | 최종 연산 | `boolean`          | `Predicate<T>`            | `T -> boolean`  |
| `allMatch`                      | 최종 연산 | `boolean`          | `Predicate<T>`            | `T -> boolean`  |
| `noneMatch`                     | 최종 연산 | `boolean`          | `Predicate<T>`            | `T -> boolean`  |
| `findFirst`                     | 최종 연산 | `Optional<T>`      | 없음                      | -               |
| `findAny`                       | 최종 연산 | `Optional<T>`      | 없음                      | -               |

### 4.5 로드맵

### 4.6 마치며

- 스트림은 소스에서 추출된 연속 요소로, 데이터 처리 연산을 지원한다.
- 스트림은 내부 반복을 지원한다. 내부 반복은 filter, map, sorted 등의 연산으로 반복을 추상화한다.
- 스트림에는 중간 연산과 최종 연산이 있다.
- 중간 연산은 스트림을 반환하면서 다른 연산과 연결되는 연산이다. 파이프라인을 구성할 순 있지만 어떤 결과도 생성할 수 없다.
- 최종 연산은 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환하는 연산이다.
- 스트림의 요소는 요청할 때 게으르게 계산된다.