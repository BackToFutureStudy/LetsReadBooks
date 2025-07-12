# Chapter 03. 람다 표현식

[TOC]

**이 장의 내용**

- 람다란 무엇인가?
- 어디에, 어떻게 람다를 사용하는가?
- 실행 어라운드 패턴
- 함수형 인터페이스, 형식 추론
- 메서드 참조
- 람다 만들기

### 3.1 람다란 무엇인가?

**람다 표현식**은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다.

- **익명**

  보통의 메서드와딜리 이름이 없으므로 **익명**이라 표현한다. 구현해야 할 코드에 대한 걱정거리가 줄어든다.

- **함수**

  람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.

- **전달**

  람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.

- 간결성

  익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.

람다를 이용해서 간결한 방식으로 코드를 전달할 수 있다. 코드가 간결하고 유연해진다.

- 표현식 스타일

  (parameters) -> expression

- 블록 스타일

  (parameters) -> { statements; }

### 3.2 어디에, 어떻게 람다를 사용할까?

#### 3.2.1 함수형 인터페이스

**함수형 인터페이스**는 정확히 하나의 추상 메서드를 지정하는 인터페이스다.

```java
public interface Comparator<T> {
	int compare(T o1, T o2);
}

public interface Runnable {
  void run();
}

public interface ActionListener extends EventListener {
	void actionPerformed(ActionEvent e);
}

public interface Callable<V> {
  V call() throws Exception();
}

public interface PrivilegedAction<T> {
  T run();
}
```

> 인터페이스는 **디폴트 메서드**(인터페이스의 메서드를 구현하지 않은 클래스를 고려해서 기본 구현을 제공하는 바디를 포함하는 메서드)를 포함할 수 있다. 많은 디폴트 메서드가 있더라도 **추상 메서드가 오직 하나면** 함수형 인터페이스다.

함수형 인터페이스는 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**(기술적으로 따지면 함수형 인터페이스를 구현한 클래스의 인스턴스)할 수 있다. 함수형 인터페이스보다는 덜 깔끔하지만 익명 내부 클래스로도 같은 기능을 구현할 수 있다.

```java
// 람다 사용
Runnable r1 = () -> System.out.println("Hello World! 1");

// 익명 클래스 사용
Runnable r2 = new Runnable() {
  public void run() {
    System.out.println("Hello World! 2");
  }
}

public static void process(Runnable r) {
  r.run();
}

process(r1);	// 'Hello World! 1' 출력
process(r2);	// 'Hello World! 2' 출력
process(() -> System.out.println("Hello World! 3"));	// 직접 전달된 람다 표현식으로 'Hello World! 3' 출력
```

#### 3.2.2 함수 디스크립터

함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다. 람다 표현식의 시그나처를 서술하는 메서드를 **함수 디스크립터**라고 부른다.

> ##### 람다와 메소드 호출
>
> ```java
> process(() -> System.out.println("This is awesome"));
> ```
>
> 위 코드에선 중괄호를 사용하지 않았고, System.out.println은 void를 반환하므로 완벽한 표현식이 아닌 것처럼 보이지만 중괄호는 필요 없다. 한 개의 void 메서드 호출은 중괄호로 감쌀 필요가 없다.

### 3.3 람다 활용 : 실행 어라운드 패턴

실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 같는다. 이런 형식의 코드를 **실행 어라운드 패턴**이라고 부른다. 이를 사용하면 자원을 명시적으로 닫을 필요가 없으므로 간결한 코드를 구현하는데 도움을 준다.

```java
public String processFile() throws IOException {
  try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    // 실제 필요한 작업을 하는 행
    return br.readLine();
  }
}
```

#### 3.3.1 1단계: 동작 파라미터화를 기억하라

#### 3.3.2 2단계: 함수형 인터페이스를 이용해서 동작 전달

#### 3.3.3 3단계: 동작 실행

#### 3.3.4 4단계: 람다 전달

### 3.4 함수형 인터페이스 사용

함수형 인터페이스의 추상 메서드 시그니처를 **함수 디스크립터**라고 한다.

#### 3.4.1 Predicate

java.util.function.Predicate<T> 인터페이스는 test라는 추상 메서드를 정의하며 test는 제네릭 형식 T의 객체를 인수로 받아 불리언을 반환한다. 따로 정의할 필요 없이 바로 사용가능하다는 점이 특징이다.

```java
@FunctionalInterface
public interface Predicate<T> {
  boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
  List<T> results = new ArrayList<>();
  for(T t : list) {
    if(p.test(t)) {
      results.add(t);
    }
  }
  return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```

#### 3.4.2 Consumer

제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다. T 형식의 객체를 인수로 받아서 각 항목에 어떤 동작을 수행하는 forEach 메서드를 정의할 때 Consumer를 활용할 수 있다.

```java
@FunctionalInterface
public interface Consumer<T> {
  void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
  for(T t : list) {
    c.accept(t);
  }
}

forEach (
  Arrays.asList(1, 2, 3, 4, 5);
  (Integer i) -> System.out.println();
);
```

#### 3.4.3 Function

제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 추상 메서드 apply를 정의한다. 입력을 출력으로 매핑하는 람다를 정의할 때 Function 인터페이스를 활용할 수 있다.

```java
@FunctionalInterface
public interface Function<T, R> {
  R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
  List<R> result = new ArrayList<>();
  for(T t : list) {
    result.add(f.apply(t));
  }
  return result;
}

// [7, 2, 6]
List<Integer> l = map(
  Arrays.asList("lambdas", "in", "action"),
  (String s) -> s.length()
);
```

##### 기본형 특화

자바의 모든 형식은 참조형 아니면 기본형에 해당한다. 하지만 제네릭 파라미터에는 참조형만 사용할 수 있다.

자바에서는 기본형을 참조형으로 변환하는 기능을 제공한다. 이 기능을 **박싱**이라고 한다. 반대의 동작은 **언박싱**이라고 한다.

프로그래머가 편리하게 코드를 구현할 수 있도록 박싱과 언박싱이 자동으로 이루어지는 **오토박싱**이라는 기능도 제공한다.

> **예외, 람다, 함수형 인터페이스의 관계**
>
> 함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 직접 함수형 인터페이스를 만들기 어려운 상황에서는 명시적으로 확인된 예외를 잡을 수 있다.

### 3.5 형식 검사, 형식 추론, 제약

#### 3.5.1 형식 검사

어떤 콘텍스트에서 기대되는 람다 표현식의 형식을 **대상 형식**이라고 부른다.

#### 3.5.2 같은 람다, 다른 함수형 인터페이스

대상 형식이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.

#### 3.5.3 형식 추론

자바 컴파일러는 람다 표현식이 사용됱 콘텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다.

#### 3.5.4 지역 변수 사용

모든 람다 표현식은 인수를 자신의 바디 안에서만 사용했다. 하지만 람다 표현식에서는 익명 함수가 하는 것처럼 **자유 변수**를 활용할 수 있다. 이와 같은 동작을 **람다 캡쳐링**이라고 부른다.

##### 지역 변수의 제약

지역 변수는 스택에 위치한다. 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다. 따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것이다.

### 3.6 메서드 참조

메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다. 때로는 람다 표현식보다 더 가독성이 좋으며 자연스러울 수 있다.

~~~java
inventory.sort(comparing(Apple::getWeight));
~~~

#### 3.6.1 요약

메서드명 앞에 구분자(::)를 붙이는 방식으로 메서드 참조를 활용할 수 있다.

~~~java
(Apple apple) -> apple.getWeight()
=>
Apple::getWeight
 
() ->
=>
Thread.currentThread()::dumpStack

(str, i) -> str.substring()
=>
String::substring
  
(String s) -> System.out.println(s) (String s)
=>
System.out::println
  
(String s) -> System.out.println(s) (String s) -> this.isValidName(s)
=> 
this::isValidName
~~~

##### 메서드 참조를 만드는 방법

1. 정적 메서드 참조
2. 다양한 형식의 인스턴스 메서드 참조
3. 기존 객체의 인스턴스 메서드 참조

#### 3.6.2 생성자 참조

ClassName::new 처럼 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다.

### 3.7 람다, 메서드 참조 활용하기

#### 3.7.1 1단계: 코드 전달

#### 3.7.2 2단계: 익명 클래스 사용

#### 3.7.3 3단계: 람다 표현식 사용

람다 표현식이라는 경량화된 문법을 이용해서 코드를 전달할 수 있다. 

~~~java
import static java.util.Comparator.comparing;
inventory.sort(Comparing(apple -> apple.getWeight()));
~~~

#### 3.7.4 4단계: 메서드 참조 사용

~~~java
inventory.sort(comparing(Apple::getWeight));
~~~

### 3.8 람다 표현식을 조합할 수 있는 유용한 메서드

#### 3.8.1 Comparator 조합

- 역정렬
- Comperator 연결

#### 3.8.2 Predicate 조합

negate, and, or 세 가지 메서드를 제공한다.

negate: 기존 프레디케이트 객체의 결과를 반전시킨 객체를 만든다.

#### 3.8.3 Function 조합

andThen, compose 두 가지 디폴트 메서드를 제공한다.  

compose: andThen 의 반대 순서 적용

### 3.9 비슷한 수학적 개념

#### 3.9.1 적분

#### 3.9.2 자바 8람다로 연결

### 3.10 마치며

- 람다 표현식은 익명 함수의 일종이다.
- 람다 표현식으로 간결한 코드를 구현할 수 있다. 
- 함수형 인터페이스는 하나의 추상 메서드만을 정의하는 인터페이스다.
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다.
- 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공할 수 있으며 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.
- java.util.function 패키지는 자주 사용하는 다양한 함수형 인터페이스를 제공한다.
- 자바 8은 제네릭 함수형 인터페이스와 관련한 박싱 동작을 피할 수 있는 기본형 특화 인터페이스도 제공한다.
- 실행 어라운드 패턴을 람다와 활용하면 유연성과 재사용성을 추가로 얻을 수 있다.
- 람다 표현식의 기대 형식을 대상 형식이라고 한다.
- 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.
- Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다.
