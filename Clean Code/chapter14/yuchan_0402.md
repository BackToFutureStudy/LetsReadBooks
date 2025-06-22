# Clean Code 14장: 점진적인 개선

## 📖 개요

이 장은 **점진적인 개선**을 보여주는 사례 연구입니다. 출발은 좋았으나 확장성이 부족했던 모듈을 소개하고, 이를 개선하고 정리하는 단계를 살펴봅니다.

> **핵심 메시지**: 프로그래밍은 과학보다 **공예(craft)**에 가깝습니다. 깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 합니다.

---

## 🎯 Args 유틸리티 소개

명령행 인수 구문을 분석하는 `Args` 유틸리티를 만들어보겠습니다.

### 사용법 예시

```java
public static void main(String[] args) {
  try {
    Args arg = new Args("l,p#,d*", args);
    boolean logging = arg.getBoolean('l');
    int port = arg.getInt('p');
    String directory = arg.getString('d');
    executeApplication(logging, port, directory);
  } catch (ArgsException e) {
    System.out.print("Argument error: %s\n", e.errorMessage());
  }
}
```

### 스키마 형식
- `l` : 부울 인수 (플래그)
- `p#` : 정수 인수
- `d*` : 문자열 인수

---

## ✨ 최종 완성된 Args 구현

### Args.java (최종 버전)

```java
public class Args {
  private Map<Character, ArgumentMarshaler> marshalers;
  private Set<Character> argsFound;
  private ListIterator<String> currentArgument;
  
  public Args(String schema, String[] args) throws ArgsException { 
    marshalers = new HashMap<Character, ArgumentMarshaler>(); 
    argsFound = new HashSet<Character>();
    
    parseSchema(schema);
    parseArgumentStrings(Arrays.asList(args)); 
  }
  
  // 스키마 파싱
  private void parseSchema(String schema) throws ArgsException { 
    for (String element : schema.split(","))
      if (element.length() > 0) 
        parseSchemaElement(element.trim());
  }
  
  // 개별 스키마 요소 파싱
  private void parseSchemaElement(String element) throws ArgsException { 
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    
    if (elementTail.length() == 0)
      marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*")) 
      marshalers.put(elementId, new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
      marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##")) 
      marshalers.put(elementId, new DoubleArgumentMarshaler());
    else if (elementTail.equals("[*]"))
      marshalers.put(elementId, new StringArrayArgumentMarshaler());
    else
      throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
  }
  
  // getter 메서드들
  public boolean getBoolean(char arg) {
    return BooleanArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public String getString(char arg) {
    return StringArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  public int getInt(char arg) {
    return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
  }
  
  // ... 기타 메서드들
}
```

### ArgumentMarshaler 인터페이스

```java
public interface ArgumentMarshaler {
  void set(Iterator<String> currentArgument) throws ArgsException;
}
```

### BooleanArgumentMarshaler 구현

```java
public class BooleanArgumentMarshaler implements ArgumentMarshaler { 
  private boolean booleanValue = false;
  
  public void set(Iterator<String> currentArgument) throws ArgsException { 
    booleanValue = true;
  }
  
  public static boolean getValue(ArgumentMarshaler am) {
    if (am != null && am instanceof BooleanArgumentMarshaler)
      return ((BooleanArgumentMarshaler) am).booleanValue; 
    else
      return false; 
  }
}
```

---

## 🚨 1차 초안의 문제점

### 문제가 있던 초기 코드의 특징

```java
public class Args {
  private String schema;
  private String[] args;
  private boolean valid = true;
  private Set<Character> unexpectedArguments = new TreeSet<Character>(); 
  private Map<Character, Boolean> booleanArgs = new HashMap<Character, Boolean>();
  private Map<Character, String> stringArgs = new HashMap<Character, String>(); 
  private Map<Character, Integer> intArgs = new HashMap<Character, Integer>(); 
  // ... 더 많은 인스턴스 변수들
```

### 주요 문제점들

1. **코드 중복**: 각 인수 타입마다 비슷한 로직 반복
2. **확장성 부족**: 새로운 인수 타입 추가 시 여러 곳 수정 필요
3. **책임 분산**: Args 클래스가 너무 많은 책임을 가짐
4. **예외 처리 혼재**: 예외 처리 코드가 Args 클래스에 섞여있음

> **💡 교훈**: 처음부터 지저분한 코드를 짜려는 생각은 없었지만, String과 Integer 인수 유형을 추가하면서부터 재앙이 시작되었습니다.

---

## 🔄 점진적 개선 과정

### 1단계: 중단 결정
- 새로운 인수 유형을 추가할 때마다 코드가 더 복잡해짐을 인식
- **기능 추가를 중단하고 리팩터링 시작**

### 2단계: TDD 적용
- **테스트 주도 개발(Test-Driven Development)** 기법 사용
- 언제든 실행 가능한 자동화된 테스트 슈트 준비
- 변경 후에도 시스템이 똑같이 동작함을 보장

### 3단계: ArgumentMarshaler 개념 도입

```java
private class ArgumentMarshaler { 
  private boolean booleanValue = false;

  public void setBoolean(boolean value) { 
    booleanValue = value;
  }
  
  public boolean getBoolean() {
    return booleanValue;
  } 
}
```

### 4단계: 점진적 변경
- 각 인수 타입별로 하나씩 ArgumentMarshaler로 이전
- 매번 테스트를 통해 시스템 정상 동작 확인

---

## 🎯 개선의 핵심 원칙

### 1. 관심사의 분리 (Separation of Concerns)
```java
// Before: Args 클래스가 오류 메시지까지 담당
public String errorMessage() {
  // Args 클래스 내부에서 오류 메시지 생성
}

// After: ArgsException이 오류 메시지 담당
public class ArgsException extends Exception {
  public String errorMessage() {
    // 전용 예외 클래스에서 메시지 처리
  }
}
```

### 2. 단일 책임 원칙 (Single Responsibility Principle)
- **Args**: 인수 파싱과 값 추출만 담당
- **ArgsException**: 예외 처리와 오류 메시지만 담당
- **ArgumentMarshaler**: 특정 타입의 인수 처리만 담당

### 3. 개방-폐쇄 원칙 (Open-Closed Principle)
```java
// 새로운 인수 타입 추가가 쉬워짐
if (elementTail.equals("&"))  // 새로운 타입 추가
  marshalers.put(elementId, new BooleanArrayArgumentMarshaler());
```

---

## 📊 Before vs After 비교

| 구분 | Before (1차 초안) | After (최종 버전) |
|------|------------------|------------------|
| **클래스 개수** | 1개 (Args) | 5개+ (Args, ArgsException, ArgumentMarshaler 인터페이스 + 구현체들) |
| **코드 라인** | ~200라인 | ~150라인 (Args만) |
| **확장성** | 어려움 (3곳 수정 필요) | 쉬움 (1곳만 수정) |
| **가독성** | 복잡함 | 명확함 |
| **테스트** | 어려움 | 쉬움 |

---

## 🏆 얻은 교훈들

### 1. 프로그래밍은 공예다
> "깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다"

### 2. 경험 있는 프로그래머의 자세
- **돌아가는 코드**에 만족하지 않음
- 지속적인 개선과 리팩터링 수행
- 코드의 구조와 설계에 신경씀

### 3. TDD의 중요성
- 변경사항이 기존 기능을 깨뜨리지 않음을 보장
- 점진적 개선을 안전하게 수행 가능
- 자신감 있는 리팩터링 가능

### 4. 적절한 시점의 리팩터링
- 너무 늦으면 손대기 어려운 골칫거리가 됨
- **5분 전에 엉망으로 만든 코드는 지금 당장 정리하기 아주 쉽다**

---

## ⚠️ 결론: 나쁜 코드의 위험성

### 나쁜 코드가 프로젝트에 미치는 영향
- **나쁜 일정**: 다시 짜면 됨 ✅
- **나쁜 요구사항**: 다시 정의하면 됨 ✅  
- **나쁜 팀 역학**: 복구하면 됨 ✅
- **나쁜 코드**: 썩어 문드러짐 ❌

### 나쁜 코드의 누적 효과
1. 점점 무게가 늘어나 팀의 발목을 잡음
2. 속도가 점점 느려짐
3. 모듈들이 서로 얽히고설킴
4. 숨겨진 의존성이 수도 없이 생김
5. 개선 비용이 엄청나게 많이 듦

### 최종 권고사항
> **코드는 언제나 최대한 깔끔하고 단순하게 정리하자. 절대로 썩어가게 방치하면 안 된다.**

---

## 📝 핵심 원칙 요약

1. **점진적 개선**: 한 번에 완벽한 코드를 만들려 하지 말고 단계적으로 개선
2. **테스트 우선**: 변경 전에 테스트를 준비하고, 매번 테스트로 검증
3. **관심사 분리**: 각 클래스와 메서드는 하나의 책임만 가져야 함
4. **지속적 정리**: 작은 변경사항이라도 즉시 정리하는 습관
5. **구조적 사고**: 확장성과 유지보수성을 고려한 설계