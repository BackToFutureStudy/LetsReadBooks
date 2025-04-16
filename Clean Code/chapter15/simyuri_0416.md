## Chapter 15. JUnit 들여다보기

JUnit은 자바 프레임워크 중 가장 유명하다.

**JUnit 프레임워크**

JUnit은 저자가 많다 하지만 시작은 켄트 벡과 에릭 감마, 두 사람이다.

우리가 살펴볼 모듈은 문자열 비교 오류를 파악할 때 유용한 코드다. ComparisonCompactor라는 모듈로 영리하게 짜인 코드다. ComparisonCompactor는 두 문자열을 받아 차이를 반환한다.

부정문은 긍정문보다 이해하기 약간 더 어렵다. 그러므로 if를 긍정문으로 만든다.

_ComparisonCompactor.java_

```java
    package junit.framework;

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
        }

        for(; !suffixOverlapsPrefix(); suffixLength++) {
            if(charFromEnd(expected, suffixLength) !=   charFromEnd(actual, sufixLength)
            )
            break;
        }
    }

    private char charFromEnd(String s, int i) {
        return s.charAt(s.lengthj() - i - 1);
    }

    private boolean suffixOverlapsPrefix() {
        return actual.length() - suffixLength <= prefixLength || expected.length() - suffixLength <= prefixLength;
    }

    private void findCommonPrefix() {
        prefixLength = 0;
        int end = Math.min(expected.length(), actual.length());
        for(; prefixLength < end; prefixLength++)
            if(expected.charAt(prefixLength) != actual.charAt(prefixLength))
                break;
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

    private String startingEllipsis() [
        return prefixLength > contextLength ? ELLIPSIS : "";
    ]

    private String startingContext() {
        int contextStart = Math.max(0, prefixLength - contextLength);
        int contextEnd = prefixLength;
        return expected.substring(contextStart, contextEnd);
    }

    private String delta(String s) {
        int deltaStart = prefixLength;
        int deltaEnd = s.length() - prefixLength;
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
```

코드가 상당히 깔끔하다. 코드를 리팩터링 하다 보면 원래 했던 변경을 되돌리는 경우가 흔하다. 리팩터링은 코드가 어느 수준에 이를 때까지 수많은 시행착오를 반복하는 작업이기 때문이다.

**결론**

세상에 개선이 불필요한 모듈은 없다. 코드를 처음보다 조금 더 깨끗하게 만드는 책임은 우리 모두에게 있다.
