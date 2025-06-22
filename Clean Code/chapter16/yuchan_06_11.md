# Clean Code 16장: SerialDate 리팩토링

## 📖 개요

이 장은 실제 오픈소스 프로젝트인 **SerialDate** 클래스를 대상으로 한 종합적인 리팩토링 사례입니다. 날짜 처리를 담당하는 이 클래스를 통해 실무에서 마주할 수 있는 다양한 코드 개선 기법들을 살펴봅니다.

> **SerialDate**: 날짜를 일련번호로 표현하는 Java 클래스 (JFreeChart 라이브러리의 일부)

---

## 🔍 1단계: 둘러보기 (First, Make it Work)

### 테스트 커버리지 확인

**현재 상태**:
- SerialDate 클래스의 테스트 커버리지: **약 50%**
- 많은 메서드가 테스트되지 않은 상태
- 일부 테스트는 부실하거나 누락된 상태

**개선 작업**:
1. **누락된 테스트 추가**: 커버리지가 없는 메서드들에 대한 테스트 작성
2. **부실한 테스트 보강**: 기존 테스트의 엣지 케이스 추가
3. **통합 테스트 추가**: 실제 사용 시나리오를 반영한 테스트

```java
// 추가된 테스트 예시
public void testLeapYear() {
    assertTrue(SerialDate.isLeapYear(2000));   // 400으로 나누어떨어짐
    assertFalse(SerialDate.isLeapYear(1900));  // 100으로 나누어떨어지지만 400으로는 안됨
    assertTrue(SerialDate.isLeapYear(2004));   // 4로 나누어떨어짐
    assertFalse(SerialDate.isLeapYear(2001));  // 평년
}
```

**💡 핵심 교훈**: 리팩토링 전에 충분한 테스트 케이스 확보가 필수입니다.

---

## 🧹 2단계: 주석 정리

### 쓸모없는 주석 제거

**제거할 주석들**:
```java
// Before - 불필요한 주석들
/**
 * 작성자: John Doe
 * 생성일: 2001-01-15
 * 수정일: 2003-03-22 - 버그 수정
 * 수정일: 2004-05-10 - 성능 개선
 * 수정일: 2005-08-03 - 새로운 메서드 추가
 */
public class SerialDate {
    // 생성자
    public SerialDate() {
        // 아무것도 하지 않음
    }
    
    // getter 메서드
    public int getDay() {
        return this.day; // day 반환
    }
}
```

**After - 정리된 주석**:
```java
/**
 * SerialDate represents a date using a serial number.
 * The serial number 1 corresponds to 1 January 1900.
 * 
 * Licensed under GNU Lesser General Public License.
 */
public class SerialDate {
    public SerialDate() {
    }
    
    public int getDay() {
        return this.day;
    }
}
```

### 유지할 가치있는 주석

```java
// 좋은 주석 - 변수의 역사와 의미 설명
private static final int EARLIEST_DATE_ORDINAL = 2;      // 1/1/1900
private static final int LATEST_DATE_ORDINAL = 3344231;  // 12/31/9999

// 좋은 주석 - 복잡한 알고리즘 설명
/**
 * 율리우스력에서 그레고리력으로 전환된 1582년 10월 4일 이후의
 * 날짜에 대해서만 정확한 계산을 보장합니다.
 */
public int calculateDayOfWeek() {
    // 복잡한 계산 로직...
}
```

---

## 🏗️ 3단계: 클래스 구조 개선

### 변수와 메서드의 적절한 배치

**Before - 추상 클래스에 구체적 구현이 섞여있음**:
```java
public abstract class SerialDate {
    // 모든 하위 클래스에서 사용되지 않는 변수들
    private static final int INCLUDE_NONE = 0;
    private static final int INCLUDE_FIRST = 1;
    private static final int INCLUDE_SECOND = 2;
    private static final int INCLUDE_BOTH = 3;
    
    // SpreadsheetDate에서만 사용되는 메서드
    public static int stringToWeekdayCode(String s) {
        // 구체적인 구현...
    }
}
```

**After - 적절한 위치로 이동**:
```java
// 추상 클래스 - 공통 인터페이스만
public abstract class DayDate {
    // 모든 구현체에서 공통으로 사용되는 상수들만
    public static final int MONDAY = Calendar.MONDAY;
    public static final int TUESDAY = Calendar.TUESDAY;
    // ...
    
    // 추상 메서드들
    public abstract int toOrdinal();
    public abstract int getYear();
    public abstract Month getMonth();
    public abstract int getDayOfMonth();
}

// 구체 클래스 - 특정 구현에 필요한 것들
public class SpreadsheetDate extends DayDate {
    // SpreadsheetDate에서만 사용되는 상수들
    private static final int INCLUDE_NONE = 0;
    private static final int INCLUDE_FIRST = 1;
    // ...
    
    // 구체적인 구현
    public static int stringToWeekdayCode(String s) {
        // 구현...
    }
}
```

**개선 효과**:
- 추상화 레벨이 명확해짐
- 코드를 이해하기 위해 찾아다녀야 하는 거리가 줄어듦
- 각 클래스의 책임이 명확해짐

---

## 🏷️ 4단계: 명명(Naming) 개선

### 1. 범위 지정 상수 개선

**Before - 포함 관계가 불명확**:
```java
public static final int INCLUDE_NONE = 0;
public static final int INCLUDE_FIRST = 1; 
public static final int INCLUDE_SECOND = 2;
public static final int INCLUDE_BOTH = 3;
```

**After - 수학적 구간 표기법 사용**:
```java
public enum DateInterval {
    OPEN,        // (a, b) - 양 끝점 제외
    CLOSED_LEFT, // [a, b) - 시작점 포함, 끝점 제외  
    CLOSED_RIGHT,// (a, b] - 시작점 제외, 끝점 포함
    CLOSED       // [a, b] - 양 끝점 포함
}
```

**장점**: 수학에서 사용하는 표준 구간 표기법으로 직관적 이해 가능

### 2. 메서드명의 의도 명확화

**Before - 부작용이 불분명한 메서드명**:
```java
public DayDate addDays(int days) {
    return DayDateFactory.makeDate(toOrdinal() + days);
}

public DayDate addMonths(int months) {
    // 새로운 객체 반환
}
```

**After - 의도가 명확한 메서드명**:
```java
public DayDate plusDays(int days) {
    return DayDateFactory.makeDate(toOrdinal() + days);
}

public DayDate plusMonths(int months) {
    // 새로운 객체 반환 - 불변성 유지
}
```

**개선 사항**:
- `add` → `plus`: 새로운 객체 반환을 암시 (Java 8 LocalDate 스타일)
- 사용자가 원본 객체 변경 vs 새 객체 반환을 명확히 구분 가능

### 3. 변수명의 의미 명확화

**Before**:
```java
int m1 = startMonth;
int m2 = endMonth;
int y1 = startYear;
int y2 = endYear;
```

**After**:
```java
Month startingMonth = startMonth;
Month endingMonth = endMonth;
Year startingYear = startYear;
Year endingYear = endYear;
```

---

## 📝 5단계: 서술적 로컬 변수 도입

### 복잡한 조건문을 읽기 쉽게

**Before - 이해하기 어려운 한 줄 코드**:
```java
public static boolean isLeapYear(int year) {
    return year % 4 == 0 && (!(year % 100 == 0) || year % 400 == 0);
}
```

**After - 서술적 변수로 의도 명확화**:
```java
public static boolean isLeapYear(int year) {
    boolean fourthYear = (year % 4 == 0);
    boolean hundredthYear = (year % 100 == 0);
    boolean fourHundredthYear = (year % 400 == 0);
    
    return fourthYear && (!hundredthYear || fourHundredthYear);
}
```

**개선 효과**:
- 윤년 계산 규칙이 명확히 드러남
- 디버깅 시 각 조건을 쉽게 확인 가능
- 코드 자체가 문서 역할 수행

### 복잡한 계산식 분해

**Before**:
```java
public int getDayOfWeek() {
    return (toOrdinal() + 6) % 7 + 1;
}
```

**After**:
```java
public Day getDayOfWeek() {
    int ordinalDay = toOrdinal();
    int daysSinceEpoch = ordinalDay - EPOCH_ORDINAL;
    int dayOfWeek = (daysSinceEpoch + EPOCH_DAY_OF_WEEK) % 7;
    
    return Day.fromInt(dayOfWeek);
}
```

---

## 🛠️ 6단계: 종합적 구조 개선

### 1. Enum 활용한 타입 안전성 확보

**Before - 매직 넘버와 정적 상수**:
```java
public static final int MONDAY = 1;
public static final int TUESDAY = 2;
// ...

public static final int JANUARY = 1;
public static final int FEBRUARY = 2;
// ...
```

**After - 타입 안전한 Enum**:
```java
public enum Day {
    MONDAY(Calendar.MONDAY),
    TUESDAY(Calendar.TUESDAY),
    WEDNESDAY(Calendar.WEDNESDAY),
    THURSDAY(Calendar.THURSDAY),
    FRIDAY(Calendar.FRIDAY),
    SATURDAY(Calendar.SATURDAY),
    SUNDAY(Calendar.SUNDAY);
    
    private final int calendarConstant;
    
    Day(int calendarConstant) {
        this.calendarConstant = calendarConstant;
    }
    
    public int toCalendarConstant() {
        return calendarConstant;
    }
}

public enum Month {
    JANUARY(1), FEBRUARY(2), MARCH(3), APRIL(4),
    MAY(5), JUNE(6), JULY(7), AUGUST(8),
    SEPTEMBER(9), OCTOBER(10), NOVEMBER(11), DECEMBER(12);
    
    private final int monthNumber;
    
    Month(int monthNumber) {
        this.monthNumber = monthNumber;
    }
    
    public int toInt() {
        return monthNumber;
    }
    
    public static Month fromInt(int monthNumber) {
        for (Month month : Month.values()) {
            if (month.monthNumber == monthNumber) {
                return month;
            }
        }
        throw new IllegalArgumentException("Invalid month: " + monthNumber);
    }
}
```

### 2. 유틸리티 클래스 분리

**Before - 추상 클래스에 유틸리티 메서드 혼재**:
```java
public abstract class SerialDate {
    // 날짜 관련 유틸리티 메서드들
    public static boolean isValidWeekdayCode(int code) { ... }
    public static String weekdayCodeToString(int weekday) { ... }
    public static int stringToWeekdayCode(String s) { ... }
    
    // 핵심 추상 메서드들
    public abstract int toOrdinal();
    public abstract int getYear();
    // ...
}
```

**After - 책임에 따른 클래스 분리**:
```java
// 핵심 추상 클래스 - 날짜 개념만
public abstract class DayDate implements Comparable<DayDate>, Serializable {
    public abstract int toOrdinal();
    public abstract Year getYear();
    public abstract Month getMonth();
    public abstract int getDayOfMonth();
    
    // 핵심 비즈니스 로직만
    public DayDate plusDays(int days) { ... }
    public DayDate plusMonths(int months) { ... }
}

// 유틸리티 클래스 - 정적 헬퍼 메서드들
public class DateUtil {
    public static boolean isValidWeekdayCode(int code) { ... }
    public static String weekdayCodeToString(Day weekday) { ... }
    public static Day stringToWeekdayCode(String s) { ... }
    
    public static int daysBetween(DayDate start, DayDate end) { ... }
    public static DayDate createDate(int day, Month month, Year year) { ... }
}

// 팩토리 클래스 - 객체 생성 책임
public class DayDateFactory {
    public static DayDate makeDate(int ordinal) { ... }
    public static DayDate makeDate(int day, Month month, Year year) { ... }
    public static DayDate makeDate(LocalDate localDate) { ... }
}
```

### 3. 추상화 레벨 정리

**설계 원칙**:
- **추상 클래스**: 공통 인터페이스와 핵심 비즈니스 로직
- **구현 클래스**: 구체적인 저장 방식과 계산 로직  
- **유틸리티 클래스**: 정적 헬퍼 메서드들
- **팩토리 클래스**: 객체 생성 로직

---

## 📊 개선 결과 비교

### Before vs After

| 구분 | Before | After |
|------|--------|-------|
| **테스트 커버리지** | ~50% | ~90% |
| **클래스 수** | 2개 (SerialDate, SpreadsheetDate) | 6개 (DayDate, SpreadsheetDate, Day, Month, DateUtil, DayDateFactory) |
| **매직 넘버** | 다수 존재 | Enum으로 대체 |
| **책임 분리** | 혼재 | 명확히 분리 |
| **가독성** | 보통 | 크게 향상 |
| **타입 안전성** | int 기반 | Enum 기반 |

### 주요 개선 사항

1. **✅ 테스트 강화**: 50% → 90% 커버리지 달성
2. **✅ 주석 정리**: 불필요한 주석 제거, 의미있는 주석 보강
3. **✅ 명명 개선**: 의도가 명확한 변수명과 메서드명 사용
4. **✅ 구조 개선**: 추상화 레벨에 따른 적절한 클래스 분리
5. **✅ 타입 안전성**: Enum 도입으로 컴파일 타임 오류 검출
6. **✅ 가독성 향상**: 서술적 변수로 복잡한 로직 단순화

---

## 🎯 핵심 리팩토링 원칙

### 1. 테스트 우선 (Test First)
```java
// 리팩토링 전 반드시 테스트부터 작성
@Test
public void testLeapYearCalculation() {
    assertTrue(DayDate.isLeapYear(Year.make(2000)));
    assertFalse(DayDate.isLeapYear(Year.make(1900)));
    assertTrue(DayDate.isLeapYear(Year.make(2004)));
    assertFalse(DayDate.isLeapYear(Year.make(2001)));
}
```

### 2. 점진적 개선 (Incremental Improvement)
- 한 번에 모든 것을 바꾸지 않음
- 각 단계마다 테스트로 검증
- 작은 단위로 나누어 안전하게 진행

### 3. 의도 표현 (Express Intent)
```java
// Before - 의도가 불분명
if (d1.compare(d2) <= 0 && d2.compare(d3) <= 0) { ... }

// After - 의도가 명확
if (isWithinRange(date, startDate, endDate)) { ... }
```

### 4. 적절한 추상화 레벨 (Appropriate Abstraction Level)
- 각 클래스와 메서드가 하나의 추상화 레벨 유지
- 구체적인 구현과 추상적인 개념 분리
- 관련 있는 것들을 함께, 관련 없는 것들을 분리

---

## 🏆 결론

### 보이스카우트 규칙의 실천

> **"언제나 우리가 처음 소스코드를 본 것보다 나아지게 만들자"**

이번 SerialDate 리팩토링을 통해 다음을 달성했습니다:

1. **📈 품질 향상**: 테스트 커버리지 증가와 버그 발견/수정
2. **🔧 구조 개선**: 책임에 따른 클래스 분리와 추상화 레벨 정리  
3. **📖 가독성 향상**: 명확한 명명과 서술적 변수 사용
4. **🛡️ 안정성 확보**: 타입 안전성과 불변성 도입
5. **🧹 깔끔함**: 불필요한 주석과 코드 제거

### 지속적 개선의 중요성

- **작은 개선도 가치있음**: 변수명 하나, 주석 하나라도 의미있는 변화
- **점진적 접근**: 한 번에 완벽하게 만들려 하지 않음
- **테스트의 중요성**: 안전한 리팩토링의 전제 조건
- **팀 차원의 노력**: 모든 개발자가 코드 품질에 책임을 가져야 함

**최종 메시지**: 완벽한 코드는 없지만, 더 나은 코드는 언제나 가능합니다. 보이스카우트 규칙을 실천하여 코드베이스를 지속적으로 개선해 나가는 것이 우리의 책임입니다.