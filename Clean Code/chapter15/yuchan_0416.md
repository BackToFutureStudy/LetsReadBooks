# Clean Code 15장: JUnit 들여다보기

## 📖 개요

JUnit은 자바 프레임워크 중 가장 유명하며, 일반적인 프레임워크와 동일하게 **개념은 단순하고, 정의는 정밀하며, 구현은 우아합니다**.

> **역사적 사실**: JUnit은 켄트 백(Kent Beck)과 에릭 감마(Erich Gamma)가 아틀란타 행 비행기에서 만들어졌다고 합니다.

---

## 🎯 ComparisonCompactor 모듈 소개

### 역할과 기능
- **목적**: 두 문자열을 받아 차이를 반환하는 문자열 비교 오류 파악 도구
- **예시**: `ABCDE`와 `ABXDE`를 받아 `<...B[X]D...>`를 반환

### 테스트 케이스 분석

```java
public class ComparisonCompactorTest extends TestCase {
  
  // 기본 메시지 테스트
  public void testMessage() {
    String failure = new ComparisonCompactor(0, "b", "c").compact("a");
    assertTrue("a expected:<[b]> but was:<[c]>".equals(failure));
  }
  
  // 시작 부분이 같은 경우
  public void testStartSame() {
    String failure = new ComparisonCompactor(1, "ba", "bc").compact(null);
    assertEquals("expected:<b[a]> but was:<b[c]>", failure);
  }
  
  // 끝 부분이 같은 경우
  public void testEndSame() {
    String failure = new ComparisonCompactor(1, "ab", "cb").compact(null);
    assertEquals("expected:<[a]b> but was:<[c]b>", failure);
  }
  
  // 컨텍스트와 생략 기호 테스트
  public void testStartAndEndContextWithEllipses() {
    String failure = new ComparisonCompactor(1, "abcde", "abfde").compact(null);
    assertEquals("expected:<...b[c]d...> but was:<...b[f]d...>", failure);
  }
}
```

**📊 테스트 커버리지**: 100% (모든 행, if문, for문 실행)

---

## 🔍 원본 코드 분석

### ComparisonCompactor.java (원본)

```java
public class ComparisonCompactor {
  private static final String ELLIPSIS = "...";
  private static final String DELTA_END = "]";
  private static final String DELTA_START = "[";
  
  private int fContextLength;
  private String fExpected;
  private String fActual;
  private int fPrefix;
  private int fSuffix;
  
  public ComparisonCompactor(int contextLength, String expected, String actual) {
    fContextLength = contextLength;
    fExpected = expected;
    fActual = actual;
  }
  
  public String compact(String message) {
    if (fExpected == null || fActual == null || areStringsEqual())
      return Assert.format(message, fExpected, fActual);
    
    findCommonPrefix();
    findCommonSuffix();
    String expected = compactString(fExpected);
    String actual = compactString(fActual);
    return Assert.format(message, expected, actual);
  }
  
  private String compactString(String source) {
    String result = DELTA_START + 
        source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;
    if (fPrefix > 0)
      result = computeCommonPrefix() + result;
    if (fSuffix > 0)
      result = result + computeCommonSuffix();
    return result;
  }
  
  // ... 기타 메서드들
}
```

### 원본 코드의 특징
✅ **장점**:
- 코드가 잘 분리되어 있음
- 표현력이 적절함
- 구조가 단순함

⚠️ **개선 필요 사항**:
- 긴 표현식과 이상한 `+1` 등이 눈에 띔
- 헝가리안 표기법(`f` 접두어) 사용
- 일부 조건문이 캡슐화되지 않음

---

## 🛠️ 점진적 개선 과정

### 1단계: 멤버 변수 접두어 제거

**Before**:
```java
private int fContextLength;
private String fExpected;
private String fActual;
private int fPrefix;
private int fSuffix;
```

**After**:
```java
private int contextLength;
private String expected;
private String actual;
private int prefix;
private int suffix;
```

**이유**: 오늘날 개발환경에서는 변수 이름에 범위를 명시할 필요가 없으며, 접두어 `f`는 중복 정보입니다.

---

### 2단계: 조건문 캡슐화

**Before**:
```java
public String compact(String message) {
  if (expected == null || actual == null || areStringsEqual())
    return Assert.format(message, expected, actual);
  // ...
}
```

**After**:
```java
public String compact(String message) {
  if (shouldNotCompact())
    return Assert.format(message, expected, actual);
  // ...
}

private boolean shouldNotCompact() {
  return expected == null || actual == null || areStringsEqual();
}
```

**이유**: 의도를 명확하게 표현하기 위해 조건문을 메서드로 추출하고 적절한 이름을 부여합니다.

---

### 3단계: 부정문을 긍정문으로 변경

**Before**:
```java
if (shouldNotCompact()) {
  return Assert.format(message, expected, actual);
}
```

**After**:
```java
if (canBeCompacted()) {
  findCommonPrefix();
  findCommonSuffix();
  String compactExpected = compactString(expected);
  String compactActual = compactString(actual);
  return Assert.format(message, compactExpected, compactActual);
} else {
  return Assert.format(message, expected, actual);
}

private boolean canBeCompacted() {
  return expected != null && actual != null && !areStringsEqual();
}
```

**이유**: 부정문은 긍정문보다 이해하기 어렵기 때문입니다.

---

### 4단계: 함수 이름 개선

**Before**: `compact(String message)`
**After**: `formatCompactedComparison(String message)`

**이유**: 
- 단순히 압축하는 것이 아니라 형식이 갖춰진 문자열을 반환
- 오류 점검이라는 부가 단계가 숨겨져 있음
- 새 이름이 함수의 실제 역할을 더 정확히 표현

---

### 5단계: 함수 분리와 책임 분담

```java
private String compactExpected;
private String compactActual;

public String formatCompactedComparison(String message) {
  if (canBeCompacted()) {
    compactExpectedAndActual();
    return Assert.format(message, compactExpected, compactActual);
  } else {
    return Assert.format(message, expected, actual);
  }
}

private void compactExpectedAndActual() {
  findCommonPrefix();
  findCommonSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}
```

**개선 사항**:
- 압축 작업과 형식 맞추기 작업을 분리
- 각 함수가 하나의 책임만 담당

---

### 6단계: 시간적 결합(Temporal Coupling) 해결

**문제점**: `findCommonSuffix`가 `findCommonPrefix`에서 계산한 `prefixIndex`에 의존

**Before**:
```java
private void compactExpectedAndActual() {
  findCommonPrefix();    // prefixIndex 계산
  findCommonSuffix();    // prefixIndex 사용
  // ...
}
```

**After**: 
```java
private void findCommonPrefixAndSuffix() {
  findCommonPrefix();
  suffixLength = 0;
  for (; !suffixOverlapsPrefix(); suffixLength++) {
    if (charFromEnd(expected, suffixLength) != 
        charFromEnd(actual, suffixLength))
      break;
  }
}

private char charFromEnd(String s, int i) {
  return s.charAt(s.length() - i - 1);
}

private boolean suffixOverlapsPrefix() {
  return actual.length() - suffixLength <= prefixLength ||
         expected.length() - suffixLength <= prefixLength;
}
```

**개선 사항**:
- 두 함수를 하나로 합쳐 시간적 결합 제거
- 더 직관적인 로직으로 변경
- `+1` 보정 제거로 코드 간소화

---

### 7단계: 변수명 정확성 개선

**Before**: `prefixIndex`, `suffixIndex`
**After**: `prefixLength`, `suffixLength`

**이유**: 실제로는 인덱스가 아닌 길이를 나타내므로 더 정확한 이름 사용

---

### 8단계: 불필요한 조건문 제거

**Before**:
```java
private String compactString(String source) {
  String result = DELTA_START + 
      source.substring(prefixLength, source.length() - suffixLength) + DELTA_END;
  if (prefixLength > 0)
    result = computeCommonPrefix() + result;
  if (suffixLength > 0)
    result = result + computeCommonSuffix();
  return result;
}
```

**After**:
```java
private String compact(String s) {
  return new StringBuilder()
    .append(startingEllipsis())
    .append(startingContext())
    .append(DELTA_START)
    .append(delta(s))
    .append(DELTA_END)
    .append(endingContext())
    .append(endingEllipsis())
    .toString();
}
```

**개선 사항**:
- 조건부 문자열 연결을 각각의 함수로 분리
- 더 읽기 쉬운 구조로 변경

---

## 🎯 최종 완성 코드

### ComparisonCompactor.java (최종 버전)

```java
public class ComparisonCompactor {
  private static final String ELLIPSIS = "...";
  private static final String DELTA_END = "]";
  private static final String DELTA_START = "[";
  
  private int contextLength;
  private String expected;
  private String actual;
  private int prefixLength;
  private int suffixLength;
  
  public ComparisonCompactor(int contextLength, String expected, String actual) {
    this.contextLength = contextLength;
    this.expected = expected;
    this.actual = actual;
  }
  
  public String formatCompactedComparison(String message) {
    String compactExpected = expected;
    String compactActual = actual;
    if (shouldBeCompacted()) {
      findCommonPrefixAndSuffix();
      compactExpected = compact(expected);
      compactActual = compact(actual);
    }
    return Assert.format(message, compactExpected, compactActual);
  }
  
  private boolean shouldBeCompacted() {
    return !shouldNotBeCompacted();
  }
  
  private boolean shouldNotBeCompacted() {
    return expected == null || actual == null || expected.equals(actual);
  }
  
  private void findCommonPrefixAndSuffix() {
    findCommonPrefix();
    suffixLength = 0;
    for (; !suffixOverlapsPrefix(); suffixLength++) {
      if (charFromEnd(expected, suffixLength) != 
          charFromEnd(actual, suffixLength))
        break;
    }
  }
  
  private void findCommonPrefix() {
    prefixLength = 0;
    int end = Math.min(expected.length(), actual.length());
    for (; prefixLength < end; prefixLength++)
      if (expected.charAt(prefixLength) != actual.charAt(prefixLength))
        break;
  }
  
  private char charFromEnd(String s, int i) {
    return s.charAt(s.length() - i - 1);
  }
  
  private boolean suffixOverlapsPrefix() {
    return actual.length() - suffixLength <= prefixLength ||
           expected.length() - suffixLength <= prefixLength;
  }
  
  private String compact(String s) {
    return new StringBuilder()
      .append(startingEllipsis())
      .append(startingContext())
      .append(DELTA_START)
      .append(delta(s))
      .append(DELTA_END)
      .append(endingContext())
      .append(endingEllipsis())
      .toString();
  }
  
  private String startingEllipsis() {
    return prefixLength > contextLength ? ELLIPSIS : "";
  }
  
  private String startingContext() {
    int contextStart = Math.max(0, prefixLength - contextLength);
    int contextEnd = prefixLength;
    return expected.substring(contextStart, contextEnd);
  }
  
  private String delta(String s) {
    int deltaStart = prefixLength;
    int deltaEnd = s.length() - suffixLength;
    return s.substring(deltaStart, deltaEnd);
  }
  
  private String endingContext() {
    int contextStart = expected.length() - suffixLength;
    int contextEnd = Math.min(contextStart + contextLength, expected.length());
    return expected.substring(contextStart, contextEnd);
  }
  
  private String endingEllipsis() {
    return (suffixLength > contextLength ? ELLIPSIS : "");
  }
}
```

---

## 📊 최종 코드 구조 분석

### 함수 분류
1. **분석 함수들**: `findCommonPrefixAndSuffix`, `findCommonPrefix`, `shouldBeCompacted` 등
2. **조합 함수들**: `compact`, `startingEllipsis`, `startingContext`, `delta` 등

### 특징
- **위상적 정렬**: 각 함수가 사용된 직후에 정의됨
- **단일 책임**: 각 함수가 하나의 명확한 역할만 수행
- **가독성**: 함수 이름만으로 역할을 쉽게 파악 가능

---

## 🎓 리팩터링에서 얻은 교훈

### 1. 시행착오의 정상성
> 💡 **코드를 리팩터링하다 보면, 원래 했던 변경을 되돌리는 경우가 흔하다**

- `shouldNotBeCompacted` 조건을 원래대로 되돌림
- 일부 메서드를 다시 `formatCompactedComparison`에 집어넣음
- **리팩터링은 코드가 어느 수준에 이를 때까지 수많은 시행착오를 반복하는 작업**

### 2. 보이스카우트 규칙 준수
- 처음 왔을 때보다 더 깨끗하게 만들기
- 세상에 개선이 불필요한 모듈은 없음
- 지속적인 개선의 중요성

### 3. 점진적 개선의 가치
- 한 번에 모든 것을 완벽하게 만들려 하지 않음
- 작은 단위로 나누어 안전하게 개선
- 각 단계에서 테스트를 통한 검증

---

## 🏆 핵심 원칙 요약

### 1. 명확한 의도 표현
- 조건문을 메서드로 추출하여 의도 명확화
- 함수 이름을 실제 역할에 맞게 변경
- 변수명을 정확한 의미로 수정

### 2. 단일 책임 원칙
- 각 함수가 하나의 책임만 담당
- 관련 있는 기능들을 함께 묶기
- 시간적 결합 제거

### 3. 가독성 우선
- 부정문보다 긍정문 사용
- 복잡한 표현식을 작은 함수로 분리
- 직관적인 함수 구조 설계

### 4. 지속적 개선
- 작은 변경사항도 즉시 정리
- 테스트를 통한 안전한 리팩터링
- 원래보다 더 깨끗한 코드 만들기

---

## 📝 결론

**보이스카우트 규칙을 지켜 모듈이 처음보다 더욱 깨끗해졌습니다.** 이는 코드를 처음보다 조금 더 깨끗하게 만드는 것이 우리 모두의 책임임을 보여줍니다.

> **"세상에 개선이 불필요한 모듈은 없다. 코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있다."**