# Chapter 15. JUnit 들여다보기

## JUnit 프레임워크

JUnit은 자바 개발자라면 누구나 사용하는 유명한 테스트 프레임워크입니다. 
이 챕터에서는 JUnit 코드의 일부를 살펴보며 클린 코드의 원칙이 어떻게 적용되었는지 분석합니다.

우리가 살펴볼 모듈은 문자열 비교 오류를 파악할 때 유용한 `ComparisonCompactor` 클래스입니다. 이 코드는 잘 분리되어 있고, 표현력이 적절하며, 구조가 단순합니다. 저자들이 모듈을 아주 좋은 상태로 남겨두었지만, 보이스카우트 규칙에 따라 한번 개선해보겠습니다.

## 원본 코드

```java
package junit.framework;

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
    if (fExpected == null || fActual == null || areStringsEqual()) {
      return Assert.format(message, fExpected, fActual);
    }
    findCommonPrefix();
    findCommonSuffix();
    String expected = compactString(fExpected);
    String actual = compactString(fActual);
    return Assert.format(message, expected, actual);
  }

  private String compactString(String source) {
    String result = DELTA_START + source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;
    if (fPrefix > 0) {
      result = computeCommonPrefix() + result;
    }
    if (fSuffix > 0) {
      result = result + computeCommonSuffix();
    }
    return result;
  }

  private void findCommonPrefix() {
    fPrefix = 0;
    int end = Math.min(fExpected.length(), fActual.length());
    for (; fPrefix < end; fPrefix++) {
      if (fExpected.charAt(fPrefix) != fActual.charAt(fPrefix)) {
        break;
      }
    }
  }

  private void findCommonSuffix() {
    int expectedSuffix = fExpected.length() - 1;
    int actualSuffix = fActual.length() - 1;
    for (; actualSuffix >= fPrefix && expectedSuffix >= fPrefix; actualSuffix--, expectedSuffix--) {
      if (fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix)) {
        break;
      }
    }
    fSuffix = fExpected.length() - expectedSuffix;
  }

  private String computeCommonPrefix() {
    return (fPrefix > fContextLength ? ELLIPSIS : "") + fExpected.substring(Math.max(0, fPrefix - fContextLength), fPrefix);
  }

  private String computeCommonSuffix() {
    int end = Math.min(fExpected.length() - fSuffix + 1 + fContextLength, fExpected.length());
    return fExpected.substring(fExpected.length() - fSuffix + 1, end) + (fExpected.length() - fSuffix + 1 < fExpected.length() - fContextLength ? ELLIPSIS : "");
  }

  private boolean areStringsEqual() {
    return fExpected.equals(fActual);
  }
}
```



## 리팩토링 과정

### 1. 접두어 제거

변수의 범위를 나타내는 접두어는 현대 IDE에서는 필요하지 않으므로 제거합니다.

```java
// Before
private int fContextLength;

// After
private int contextLength;
```



### 2. 조건문 캡슐화

의도를 명확하게 표현하기 위해 조건문을 캡슐화합니다.

```java
// Before
if (expected == null || actual == null || areStringsEqual()) {
  return Assert.format(message, expected, actual);
}

// After
if (shouldNotCompact()) {
  return Assert.format(message, expected, actual);
}

private boolean shouldNotCompact() {
  return expected == null || actual == null || areStringsEqual();
}
```



### 3. 변수명을 명확하게 변경

함수에서 멤버 변수와 이름이 똑같은 변수를 사용하는 것은 혼란스러울 수 있습니다. 다른 의미라면 명확하게 이름을 구분해야 합니다.

```java
// Before
String expected = compactString(fExpected);
String actual = compactString(fActual);

// After
String compactExpected = compactString(expected);
String compactActual = compactString(actual);
```



### 4. 부정문을 긍정문으로 변경

부정문은 긍정문보다 이해하기 약간 더 어렵습니다. 조건문을 긍정으로 변경하여 가독성을 높입니다.

```java
// Before
if (shouldNotCompact()) {
  return Assert.format(message, expected, actual);
} else {
  // 처리 로직
}

private boolean shouldNotCompact() {
  return expected == null || actual == null || areStringsEqual();
}

// After
if (canBeCompacted()) {
  // 처리 로직
} else {
  return Assert.format(message, expected, actual);	
}

private boolean canBeCompacted() {
  return expected != null && actual != null && !areStringsEqual();
}
```



### 5. 적절한 함수이름 적용

`compact` 함수는 `canBeCompacted`가 false면 압축하지 않습니다. 그러므로 이름을 `compact`로 할 경우 오류 점검이라는 부가 단계가 숨겨집니다. 또한 해당 함수는 단순히 압축 문자열을 반환하는 것이 아니라 형식이 갖춰진 문자열을 반환합니다. 따라서 `formatCompactedComparison`이라는 이름이 더 적절합니다.



### 6. 함수 분리

if문 안에서는 예상 문자열과 실제 문자열을 진짜로 압축하는 로직을 별도의 함수로 분리합니다.

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



### 7. 일관적인 함수 사용방식 적용

`compactExpectedAndActual` 함수에서 마지막 두 줄은 변수를 반환하지만 첫째 줄과 둘째 줄은 반환값이 없어 함수 사용방식이 일관적이지 못합니다. 이를 개선합니다.

```java
// Before
private void compactExpectedAndActual() {
  findCommonPrefix();
  findCommonSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}

// After
private void compactExpectedAndActual() {
  prefixIndex = findCommonPrefix();
  suffixIndex = findCommonSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}
```



### 8. 숨겨진 시간적인 결합(hidden temporal coupling) 해결

`findCommonSuffix`는 `findCommonPrefix`가 `prefixIndex`를 계산한다는 사실에 의존하는 시간적 결합이 존재합니다. 함수 호출 순서가 바뀔 경우 오류를 찾아내기 힘들기 때문에, 이 의존성을 명확히 드러냅니다.

```java
private void compactExpectedAndActual() { 
  prefixIndex = findCommonPrefix(); 
  suffixIndex = findCommonSuffix(prefixIndex); 
  compactExpected = compactString(expected); 
  compactActual = compactString(actual);
}
```

그러나 이 방식은 호출 순서는 명확하지만 `prefixIndex`가 필요한 이유는 설명하지 못합니다. 의도가 분명히 드러나지 않으므로 다른 프로그래머가 되돌려 놓을 가능성이 있습니다.

따라서 `findCommonPrefixAndSuffix`로 합친 후 내부에서 올바른 순서로 호출하도록 변경합니다.

```java
private void compactExpectedAndActual() {
  findCommonPrefixAndSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}

private void findCommonPrefixAndSuffix() {
  findCommonPrefix();
  // 이후 로직
}
```



## 리팩토링 정리

* 모듈은 일련의 분석 함수와 조합 함수로 나뉘었습니다. 
* 전체 함수는 위상적으로 정렬했으므로 각 함수가 사용된 직후에 정의됩니다. 
* 분석 함수가 먼저 나오고 조합 함수가 그 뒤를 이어서 나옵니다.
* 리팩토링 과정에서 초반에 내렸던 결정을 번복한 부분도 있었습니다. 
* 코드를 리팩토링하다 보면 원래 했던 변경을 되돌리는 경우가 흔합니다. 
* 이는 코드가 어느 수준에 이를 때까지 수많은 시행착오를 반복하는 작업이기 때문입니다.



## 결론

* 우리는 보이스카우트 규칙을 지켰습니다. 모듈은 처음보다 조금 더 깨끗해졌습니다.
* **세상에 개선이 불필요한 모듈은 없습니다.**  코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있습니다.