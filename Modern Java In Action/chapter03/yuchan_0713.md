# Chapter 03. 람다 표현식

## ✅ 이 장에서 다루는 내용

- 람다 표현식의 개념과 사용 방식  
- 실행 어라운드 패턴  
- 함수형 인터페이스와 함수 디스크립터  
- 형식 추론과 메서드 참조  
- 람다 표현식 조합 방법  

---

## 3.1 람다란 무엇인가?

람다 표현식은 **이름 없는 함수(익명 함수)**이다. 메서드처럼 파라미터, 바디, 반환형,  
예외 리스트를 가질 수 있으며, **메서드 인수로 전달하거나 변수에 저장**할 수 있다.

### 🔹 특징

- **익명**: 이름 없이 정의되어 코드 전달에 집중 가능  
- **함수**: 클래스에 종속되지 않음  
- **전달 가능**: 메서드 인수 또는 변수로 사용 가능  
- **간결함**: 익명 클래스보다 코드가 짧고 명확  

### 🔹 문법

- **표현식 스타일**  
  `(params) -> expression`

- **블록 스타일**  
  `(params) -> { statements; }`

---

## 3.2 어디에, 어떻게 사용할까?
### 3.2.1 함수형 인터페이스

**하나의 추상 메서드만 포함하는 인터페이스**를 말하며, 람다 표현식은 해당 인터페이스의 인스턴스로 간주됨.

```java
Runnable r1 = () -> System.out.println("Hello");

Runnable r2 = new Runnable() {
  public void run() {
    System.out.println("Hello");
  }
};
```

### 3.2.2 함수 디스크립터
람다의 시그니처를 정의하는 함수형 인터페이스의 추상 메서드 시그니처를 말함.

process(() -> System.out.println("Hello"));
중괄호 없이도 한 줄의 void 메서드는 표현식 형태로 작성 가능.

## 3.3 실행 어라운드 패턴
자원 처리 코드를 설정과 정리 작업으로 감싸는 패턴.

```java
public String processFile() throws IOException {
  try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    return br.readLine();
  }
}
```
람다로 읽기 작업만 전달하고, 리소스 처리 로직은 재사용 가능

## 3.4 주요 함수형 인터페이스
### 3.4.1 Predicate<T>

```java
@FunctionalInterface
public interface Predicate<T> {
  boolean test(T t);
}

Predicate<String> nonEmpty = s -> !s.isEmpty();
```

### 3.4.2 Consumer<T>
```java
@FunctionalInterface
public interface Consumer<T> {
  void accept(T t);
}

Consumer<Integer> print = i -> System.out.println(i);
```

### 3.4.3 Function<T, R>
```java
@FunctionalInterface
public interface Function<T, R> {
  R apply(T t);
}

Function<String, Integer> lengthFunc = s -> s.length();
```

🔹 기본형 특화
참조형만 제네릭에 사용 가능  
오토박싱/언박싱 자동 처리

IntPredicate, DoubleFunction 등 기본형 특화 인터페이스 제공

## 3.5 형식 검사, 추론, 제약
### 3.5.1 대상 형식
람다가 사용되는 문맥이 기대하는 함수형 인터페이스 타입

### 3.5.2 같은 람다, 다른 인터페이스
람다 시그니처가 호환되면 다양한 함수형 인터페이스에 사용 가능

### 3.5.3 형식 추론
컴파일러가 람다의 대상 형식 기반으로 타입 추론

### 3.5.4 지역 변수 캡처
람다는 자유 변수(외부 지역 변수)를 캡처할 수 있음. 단, 변수는 final 또는 effectively final이어야 함.

지역 변수는 스택에 저장되므로 복사본만 사용 가능 → 값 변경 불가

## 3.6 메서드 참조
기존 메서드를 람다처럼 재사용 가능.

```java
(Apple a) -> a.getWeight()
==> Apple::getWeight

(String s) -> System.out.println(s)
==> System.out::println

() -> Thread.currentThread().dumpStack()
==> Thread.currentThread()::dumpStack
```

### 3.6.1 참조 유형
정적 메서드 – ClassName::staticMethod

임의 객체의 인스턴스 메서드 – ClassName::instanceMethod

특정 객체의 인스턴스 메서드 – instance::method

생성자 참조 – ClassName::new

### 3.7 람다 사용 단계별 비교

#### ✅ 1단계: 코드 전달

```java
sortApplesByWeight(apples);
```

---

#### ✅ 2단계: 익명 클래스

```java
inventory.sort(new Comparator<Apple>() {
  public int compare(Apple a1, Apple a2) {
    return a1.getWeight().compareTo(a2.getWeight());
  }
});
```

---

#### ✅ 3단계: 람다 표현식

```java
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

---

#### ✅ 4단계: 메서드 참조

```java
inventory.sort(comparing(Apple::getWeight));
```

---

### 3.8 람다 조합 메서드

#### 🔸 3.8.1 Comparator 조합

- `.reversed()`  
- `.thenComparing()`

#### 🔸 3.8.2 Predicate 조합

- `.negate()` : 조건 반전  
- `.and()`, `.or()` : 조건 결합

#### 🔸 3.8.3 Function 조합

- `.compose()` : 먼저 실행 후 본 함수 적용  
- `.andThen()` : 본 함수 적용 후 다음 함수 실행

---

### 3.9 수학적 개념 연결

- 람다 표현식은 수학의 함수 개념과 유사  
- 여러 함수를 조합하여 새로운 함수를 만들 수 있음

---

### 3.10 마치며 (요약)

- 람다는 **익명 함수**로, 메서드처럼 사용할 수 있음  
- 함수형 인터페이스는 **추상 메서드 1개**를 가진 인터페이스  
- `java.util.function` 패키지에 다양한 기본 인터페이스 제공  
- 람다는 함수형 인터페이스의 **인스턴스로 간주**됨  
- **실행 어라운드 패턴**에 유용하게 활용 가능  
- 람다 표현식은 **대상 형식**을 통해 타입 추론됨  
- **메서드 참조**로 기존 메서드를 직접 전달 가능  
- Comparator, Predicate, Function은 **조합 메서드** 제공
