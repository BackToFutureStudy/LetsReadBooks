# Chapter02. 동작 파라미터화 코드 전달하기

**이 장의 내용**

- 변화하는 요구사항에 대응
- 동작 파라미터화
- 익명 클래스
- 람다 표현식 미리보기
- 실전 예제 : Comparator, Runnable, GUI

 우리가 어떤 상황에서 일을 하든 소비자 요구사항은 항상 바뀐다. 변화하는 요구사항은 소프트웨어 엔지니어링에서 피할 수 없는 문제이다. 이렇게 시시각각 변하는 사용자 요구사항에 어떻게 대응해야 할까? 특히 우리의 엔지니어링적인 비용이 가장 최소화될 수 있으면 좋을 것이다. 그뿐 아니라 새로 추가한 기능은 쉽게 구현할 수 있어야 하며 장기적인 관점에서 유지보수가 쉬워야 한다.

 **동작 파라미터화**를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다. 동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다. 이 코드 블록은 나중에 프로그램에서 호출한다.

 예를 들어, 컬렉션을 처리할 때 다음고하 같은 메서드를 구현한다고 가정하자

- 리스트의 모든 요소에 대해서 어떤 동작을 수행할 수 있음
- 리스트 관련 작업을 끝낸 다음에 어떤 다른 동작을 수행할 수 있음
- 에러가 발생하면 정해진 어떤 다른 동작을 수행할 수 있음

### 2.1 변화하는 요구사항에 대응하기

 일단 하나의 예제를 선정한 다음 예제 코드를 점차 개선하면서 유연한 코드를 만들어보자. 기존의 농장 재고목록 애플리케이션에 리스트에서 녹색 새과만 필터링하는 기능을 추가한다고 가정하자.

#### 2.1.1 첫 번째 시도 : 녹색 사과 필터링

사과 색을 정의하는 다음과 같은 Color num이 존재한다고 가정하자

~~~java
enum Color{RED, GREEN}
~~~

첫 번째 시도 결과 코드

~~~java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    if(GREEN.equals(apple.getColor())) {
      result.add(apple);
    }
  }
  return result;
}
~~~

 만약 갑자기 녹색 사과 말고 빨관 사과도 필터링 하고 싶어졌다면 어떻게 고쳐야 할까?

#### 2.1.2 두 번째 시도 : 색을 파라미터화

~~~java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    if(apple.getColor().equals(color)) {
      result.add(apple);
    }
  }
  return result;
}
~~~

 만약 색 이외에도 가벼운 사과와 무거운 사과로 구분할 수 있다면 좋을 것 같다. 다양한 요구사항을 듣다보면 색과 마찬가지로 앞으로 바뀔 수 있는 다양한 무게에 대응할 수 있도록 무게 정보 파라미터도 추가했다.

~~~java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    if(apple.getWeight() > weight) {
      result.add(apple);
    }
  }
  return result;
}
~~~

 구현 코드를 자세히 보면 목록을 검색하고, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복된다. 이는 소프트웨어 공학의 DRY(*don't repeat yourself*)(같은 것을 반복하지 말 것) 원칙을 어기는 것이다.

#### 2.1.3 세 번째 시도 : 가능한 모든 속성으로 필터링

 모든 속성을 메서드 파라미터로 추가하면 앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수도 없다. 

### 2.2 동작 파라미터화

 사과의 어떤 속성에 기초해서 불리언값을 반환하는 방법이 있다.  참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 한다. **선택 조건을 결정하는 인터페이스**를 정의하자.

~~~java
public interface ApplePredicate {
  boolean test(Apple apple);
}
~~~

 아래 코드처럼 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있다.

~~~java
public class AppleHeavyWeightPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return apple.getWeight() > 150;
  }
}

public class AppleGreenColorPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return GREEN.equals(apple.getColor());
  }
}
~~~

 위 조건에 따라 filter 메서드가 다르게 동작할 것이라고 예상할 수 있다. 아를 전략 디자인 패턴(*strategy design pattern*)이라고 부른다. 각 알고리즘(전략)을 캡슐화해서 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다. 

알고리즘 패밀리 -> ApplePredicate

전략 -> AppleHeavyWeightPredicate, AppleGreenColorPredicate

 이제 filterApples 메서드가 ApplePredicate 객체를 인수로 받도록 고치자.

#### 2.2.1 네 번째 시도 : 추상적 조건으로 필터링

ApplePredicate를 이용한 필터 메서드

~~~java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    // 프레디케이트 객체로 사과 검색 조건을 캡슐화
    if(p.test(apple)) {
      result.add(apple);
    }
  }
  return result;
}
~~~

##### 코드/동작 전달하기

 이제 필요한 대로 다양한 ApplePredicate를 만들어서 filterApples메서드로 전달할 수 있다. 즉, filterApples메서드의 동작을 파라미터화한 것이다. filterApples메서드의 새로운 동작을 정의하는 것이 test메서드이다. 안타깝게도 메서드는 객체만 인수로 받으므로 test메서드를 ApplePredicate 객체로 감싸서 전달해야 한다. test 메서드를 구현하는 객체를 이용해서 불리언 표현식 등을 전달할 수 있으므로 이는 코드를 전달할 수 있는 것이나 다름 없다. 

##### 한 개의 파라미터, 다양한 동작

 컬랙션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 동작 파라미터화의 강점이다. 따라서, 한 메서드가 다른 동작을 수행하도록 재활용할 수 있다. 유연한 API를 만들 때 동작 파라미터화가 중요한 역할을 한다.

> **퀴즈 2-1 유연한 prettyPrintApple 메서드 구현하기**
>
> 사과 리스트를 인수로 받아 다양한 방법으로 문자열을 생성할 수 있도록 파라미터화된 prettyPrintApple 메서드를 구현하여라.
>
> ~~~java
> public static void prettyPrintApple(List<Apple> inventory, AppleFormatter formatter) {
>   for(Apple apple : inventory) {
>     String output = formatter.accept(apple);
>     System.out.println(output);
>   }
> }
> ~~~

 여러 클래스를 구현해서 인스턴스화하는 과정이 조금은 거추장스럽게 느껴질 수 있다. 이 부분을 개선해보자.

### 2.3 복잡한 과정 간소화

 자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 익명 클래스라는 기법을 제공한다. 이를 이용하면 코드의 양을 줄일 수 있다. 

#### 2.3.1 익명 클래스

**익명 클래스**는 자바의 지역 클래스와 비슷한 개념이다. 

#### 2.3.2 다섯 번째 시도 : 익명 클래스 시용

익명 클래스를 이용해서 ApplePredicate를 구현하는 객체를 만드는 방법으로 필터링 예제를 다시 구현하였다.

~~~java
List<Apple> redApples = filterApples(inventory, new ApplePredicate()) {
  // filterApples 메서드의 동작을 직접 파라미터화함
  public boolean test(Apple apple) {
    return RED.equals(apple.getColor());
  }
}
~~~

 GUI 애플리케이션에서 이벤트 핸들러 객체를 구현할 때는 익명 클래스를 종종 사용한다. 

 익명 클래스로도 아직 부족한 점이 있다.

1. 익명 클래스는 여전히 많은 공간을 차지한다.
2. 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다.

~~~java
public class MeaningOfThis {
  public final int value = 4;
  publi void doIt() {
    int value = 6;
    Runnable r = new Runnable() {
      public final int value = 5;
      public void run() {
        int value = 10;
        System.out.println(this.value);
      }
    };
    r.run();
  }
  
  public static void main(String...args) {
    MeaningOfThis m = new MeaningOfThis();
    m.doIt();
  }
}
~~~

m.doIt();의 출력 결과는?

코드에서 this는 MeaningOfThis가 아니라  Runnable을 참조하므로 5가 정답이다. 

 코드의 장황함은 나쁜 특성이다. 장황한 코드는 구현하고 유지보수하는 데 시간이 오래 걸릴 뿐 아니라, 읽는 즐거움을 빼앗는 요소로, 개발자로부터 외면받는다. 람다 표현식을 이용해서 어떻게 코드를 간결하게 정리할 수 있는지 간단히 살펴보자.

#### 2.3.3 여섯 번째 시도 : 람다 표현식 사용

자바 8의 람다 표현식을 이용해서 위 예제 코드를 다음처럼 간단하게 재구현할 수 있다.

~~~java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
~~~

 이전 코드보다 훨씬 간결해지면서 문제를 더 잘 설명하는 코드가 되었다.

#### 2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화

~~~java
public interface Predicate<T> {
  boolean test(T t);
}

// 형식 파라미터 T 등장
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
  List<T> result = new ArrayList<>();
  for(T e : list) {
    if(p.test(e)) {
      result.add(e);
    }
  }
  return result;
}
~~~

 이제 바나나, 오렌지, 정수, 문자열 등의 리스트에 필터 메서드를 사용할 수 있다. 다음은 람다 표현식을 사용한 예제이다.

~~~java
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
~~~

### 2.3 실전 예제

#### 2.4.1 Comparator로 정렬하기

 컬렉션 정렬은 반복되는 프로그래밍 작업이다. 개발자에게는 변화하는 요구사항에 쉽게 대응할 수 있는 다양한 정렬 동작을 수행할 수 있는 코드가 절실하다. 

 자바 8의 List에는 sort메서드가 포함되어 있따. Comparator를 구현해서 sort 메서드의 동작을 다양화할 수 있다. 

~~~java
inventory.sort(new Comparator<Apple>() {
  public int compare(Apple a1, Apple a2) {
    return a1.getWeight().compareTo(a2.getWeight());
  }
});
~~~

 실제 정렬 세부사항은 추상화되어 있으므로 신경 쓸 필요가 없다. 람다 표현식을 이용하면 다음처럼 간단하게 코드를 구현할 수 있다.

~~~java
inventory.sort(
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
~~~

#### 2.4.2 Runnable로 코드 블록 실행하기

 자바 스레드를 이용하면 병렬로 코드 블록을 실행할 수 있다. 여러 스레드가 각자 다른 코드를 실행할 수 있다나중에 실행할 수 있는 코드를 구현할 방법이 필요하다. 자바 8까지는 Thread 생성자에 객체만을 전달할 수 있었으므로 보통 겨로가를 반환하지 않는 void run 메서드를 포함하는 익명 클래스가 Runnable 인터페이스를 구현하도록 하는 것이 일반적인 방법이었다.

 자바에서는 Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있다. 

 자바 8부터는 지원하는 람다 표현식을 이용하면 다음처럼 스레드 코드를 구현할 수 있다.

~~~java
Thread t = new Thread(() -> System.out.println("Hello World!"));
~~~

#### 2.4.3 Callable을 결과로 반환하기 

 Callable 인터페이스를 이용해 결과를 반환하는 태스크를 만든다. 이 방식은 Runnable의 업그레이드 버전이라고 생각할 수 있다.

~~~java
public interface Callable<V> {
  V call();
}
~~~

 실행 서비스에 태스크를 제출해서 위 코드를 활용할 수 있다. 

~~~java
ExecutorService executorService = Executors.newCachedThreadPool();
Future<String> threadName = executorService.submit(new Callable<String>() {
  @Override
  public String call()throws Exception {
    return Thread.currentThread().getName();
  }
});
~~~

 람다를 이용하면 다음처럼 코드를 줄일 수 있다.

~~~java
Future<String> threadName = executorService.submit(() -> Thread.currentThread().getName());
~~~

#### 2.4.4 GUI 이벤트 처리하기

 일반적으로 GUI 프로그래밍은 마우스 클릭이나 문자열 위로 이동하는 등의 이벤트에 대응하는 동작을 수행하는 식으로 동작한다. GUI 프로그래밍에서도 변화에 대응할 수 있는 유연한 코드가 필요하다. 모든 동작에 반응할 수 있어야 하기 때문이다. 자바 FX에서는 setOnAction 메서드에 EventHandler를 전달함으로써 이벤트에 어떻게 반응할 지 설정할 수 있다.

~~~java
Button button = new Button("send");
button.setOnAction(new EventHandler<ActionEvent>() {
  public void handle(ActionEvent event) {
    label.setText("Sent!!");
  }
});
~~~

 즉, EventHandler는 setOnAction 메서드의 동작을 파라미터화한다. 람다 표현식으로 다음처런 구현할 수 있다.

~~~java
button.setOnAction((ActionEvent event) -> label.setText("Sent!!"));
~~~

### 2.5 마치며

- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄일 수 있다.
- 코드 전달 기법을 이용하면 동작을 메서드 인수로 전달할 수 있다.
- 자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화할 수 있다.