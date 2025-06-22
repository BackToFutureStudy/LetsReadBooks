# Clean Code 17장: 냄새와 휴리스틱

## 📖 개요

이 장은 마틴 파울러의 **Refactoring**에서 언급된 '코드 냄새'와 저자의 경험에서 나온 **휴리스틱(경험법칙)**을 종합적으로 정리한 실무 가이드입니다.

> **휴리스틱**: 문제의 답을 경험법칙, 직관적 판단, 상식, 시행착오 등의 방법으로 구하는 것 (논리적 알고리즘의 반대 개념)

---

## 💬 주석

### 부적절한 정보
```java
// ❌ 나쁜 예 - 주석에 부적절한 정보
/**
 * 작성자: 김개발 (kim.dev@company.com)
 * 생성일: 2023-01-15
 * 마지막 수정: 2024-03-22 - 성능 개선 (이슈 #1234)
 * 승인자: 박팀장
 */
public class UserService {
    // ...
}
```

**문제점**: 주석은 코드와 설계에 기술적인 설명을 부연하는 수단이어야 합니다.

### 쓸모없는 주석
```java
// ❌ 쓸모없는 주석들
public class Calculator {
    // 기본 생성자
    public Calculator() {
        // 아무것도 하지 않음
    }
    
    // 두 수를 더하는 메서드
    public int add(int a, int b) {
        return a + b; // a와 b를 더해서 반환
    }
}
```

**해결책**: 오래된, 엉뚱한, 잘못된 주석은 즉시 삭제합니다.

### 중복된 주석
```java
// ❌ 코드만으로 충분한 불필요한 주석
i++; // i 증가

/**
 * @param sellRequest
 * @return
 * @throws ManagedComponentException
 */
public SellResponse beginSellItem(SellRequest sellRequest) 
    throws ManagedComponentException {
    // 구현...
}
```

### 성의없는 주석
```java
// ❌ 성의없는 주석
// 이것은 중요한 메서드다
// 조심해서 사용하라

// ✅ 성의있는 주석
/**
 * 사용자의 마지막 로그인 시간을 업데이트합니다.
 * 동시성 문제를 방지하기 위해 낙관적 락을 사용합니다.
 */
```

### 주석 처리된 코드
```java
// ❌ 주석 처리된 코드는 즉시 삭제
public void processOrder(Order order) {
    validateOrder(order);
    // calculateDiscount(order); // 할인 기능 임시 제거
    // updateInventory(order);   // 재고 업데이트 나중에 추가
    saveOrder(order);
}
```

**이유**: 소스 코드 관리 시스템이 모든 변경 이력을 기억하므로 걱정할 필요 없습니다.

---

## 🏗️ 환경

### 여러 단계로 빌드해야 한다
```bash
# ❌ 복잡한 빌드 과정
git checkout main
git submodule update --init --recursive
./configure --with-custom-flags
make deps
make compile
make package

# ✅ 간단한 빌드 과정
./build.sh
```

**원칙**: 한 명령으로 전체를 체크아웃하고 한 명령으로 빌드할 수 있어야 합니다.

### 여러 단계로 테스트해야 한다
```bash
# ❌ 복잡한 테스트 실행
./run-unit-tests.sh
./run-integration-tests.sh  
./run-e2e-tests.sh
./generate-coverage.sh

# ✅ 간단한 테스트 실행
npm test
# 또는 IDE에서 버튼 하나로
```

**원칙**: 모든 테스트를 한 번에 실행하는 능력은 빠르고, 쉽고, 명백해야 합니다.

---

## 🔧 함수

### 너무 많은 인수
```java
// ❌ 너무 많은 인수
public void createUser(String firstName, String lastName, String email, 
                      String phone, String address, String city, 
                      String state, String zipCode, Date birthDate) {
    // ...
}

// ✅ 객체로 묶어서 인수 줄이기
public void createUser(UserInfo userInfo, ContactInfo contactInfo) {
    // ...
}
```

**원칙**: 인수는 0개가 가장 좋고, 4개 이상은 피해야 합니다.

### 출력 인수
```java
// ❌ 출력 인수 - 직관을 위배
public void appendFooter(StringBuffer report) {
    report.append("Report generated at: " + new Date());
}

// ✅ 객체 상태 변경
public class Report {
    public void appendFooter() {
        this.content.append("Report generated at: " + new Date());
    }
}
```

### 플래그 인수
```java
// ❌ 플래그 인수 - 여러 기능을 수행
public void render(boolean isSummary) {
    if (isSummary) {
        renderSummary();
    } else {
        renderDetail();
    }
}

// ✅ 별도 함수로 분리
public void renderSummary() { /* ... */ }
public void renderDetail() { /* ... */ }
```

### 죽은 함수
```java
// ❌ 아무도 호출하지 않는 함수
private void calculateSomethingNoOneUses() {
    // 복잡한 계산...
    // 하지만 아무도 호출하지 않음
}
```

**해결책**: 과감히 삭제하세요. 소스 코드 관리 시스템이 기억합니다.

---

## 🌐 일반

### 한 소스 파일에 여러 언어 사용
```java
// ❌ 여러 언어 혼재
public class UserController {
    public String renderUserPage(User user) {
        String html = "<div>" +
                     "<script>" +
                     "function validateUser() {" +
                     "  if (user.age < 18) alert('Too young');" +
                     "}" +
                     "</script>" +
                     "<style>.user { color: blue; }</style>" +
                     "</div>";
        return html;
    }
}
```

**해결책**: 각 언어를 별도 파일로 분리하고 범위를 최소화합니다.

### 당연한 동작을 구현하지 않는다
```java
// ❌ 당연한 동작을 구현하지 않음
public class DayOfWeek {
    public static DayOfWeek monday() { return new DayOfWeek(1); }
    // tuesday(), wednesday() 등은 구현하지 않음 - 예상과 다름
}

// ✅ 당연한 동작 모두 구현
public enum DayOfWeek {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

**원칙**: **최소 놀람의 원칙**에 따라 사용자가 당연하게 여길 동작을 모두 제공해야 합니다.

### 중복 ⭐️ (가장 중요한 규칙 중 하나)

#### 🔸 가장 뻔한 유형: 동일한 코드 반복
```java
// ❌ 코드 중복
public void saveUser(User user) {
    if (user.getName() == null || user.getName().trim().isEmpty()) {
        throw new IllegalArgumentException("Name is required");
    }
    if (user.getEmail() == null || user.getEmail().trim().isEmpty()) {
        throw new IllegalArgumentException("Email is required");
    }
    // save logic...
}

public void updateUser(User user) {
    if (user.getName() == null || user.getName().trim().isEmpty()) {
        throw new IllegalArgumentException("Name is required");
    }
    if (user.getEmail() == null || user.getEmail().trim().isEmpty()) {
        throw new IllegalArgumentException("Email is required");
    }
    // update logic...
}

// ✅ 중복 제거
private void validateUser(User user) {
    validateRequired(user.getName(), "Name is required");
    validateRequired(user.getEmail(), "Email is required");
}

private void validateRequired(String value, String message) {
    if (value == null || value.trim().isEmpty()) {
        throw new IllegalArgumentException(message);
    }
}
```

#### 🔸 미묘한 유형: switch/case 중복
```java
// ❌ 여러 곳에서 같은 조건 확인
public class EmployeePayCalculator {
    public double calculatePay(Employee emp) {
        switch (emp.getType()) {
            case HOURLY: return calculateHourlyPay(emp);
            case SALARIED: return calculateSalariedPay(emp);
            case COMMISSIONED: return calculateCommissionedPay(emp);
        }
    }
    
    public String getPayStub(Employee emp) {
        switch (emp.getType()) {  // 동일한 조건 반복
            case HOURLY: return getHourlyPayStub(emp);
            case SALARIED: return getSalariedPayStub(emp);
            case COMMISSIONED: return getCommissionedPayStub(emp);
        }
    }
}

// ✅ 다형성으로 중복 제거
public abstract class Employee {
    public abstract double calculatePay();
    public abstract String getPayStub();
}

public class HourlyEmployee extends Employee {
    public double calculatePay() { /* ... */ }
    public String getPayStub() { /* ... */ }
}
```

### 추상화 수준이 올바르지 못하다
```java
// ❌ 고차원과 저차원 개념이 섞임
public abstract class Shape {
    // 고차원 개념
    public abstract double area();
    
    // 저차원 구현 정보 - 여기 있으면 안됨
    protected int pixelX, pixelY;  
    public void setPixelPosition(int x, int y) {
        this.pixelX = x;
        this.pixelY = y;
    }
}

// ✅ 추상화 수준 분리
public abstract class Shape {
    public abstract double area();
    public abstract Point getCenter();
}

public class GraphicalShape extends Shape {
    private int pixelX, pixelY;  // 구현 세부사항은 여기에
    // ...
}
```

### 과도한 정보
```java
// ❌ 너무 많은 public 인터페이스
public class User {
    public String firstName;
    public String lastName; 
    public String email;
    public String password;
    public Date lastLoginTime;
    public int failedLoginAttempts;
    public boolean isAccountLocked;
    
    public void validatePassword() { /* ... */ }
    public void incrementFailedLogins() { /* ... */ }
    public void resetFailedLogins() { /* ... */ }
    public void lockAccount() { /* ... */ }
    public void unlockAccount() { /* ... */ }
    // 너무 많은 public 메서드와 변수들...
}

// ✅ 인터페이스 최소화
public class User {
    private String firstName;
    private String lastName;
    private String email;
    private AuthenticationInfo authInfo;  // 인증 관련 정보 캡슐화
    
    // 꼭 필요한 public 메서드만
    public String getFullName() { return firstName + " " + lastName; }
    public boolean authenticate(String password) { /* ... */ }
}
```

### 서술적 변수
```java
// ❌ 의도가 불분명한 복잡한 조건
public boolean isLeapYear(int year) {
    return year % 4 == 0 && (!(year % 100 == 0) || year % 400 == 0);
}

// ✅ 서술적 변수로 의도 명확화
public boolean isLeapYear(int year) {
    boolean divisibleByFour = (year % 4 == 0);
    boolean centuryYear = (year % 100 == 0);
    boolean quarterCenturyYear = (year % 400 == 0);
    
    return divisibleByFour && (!centuryYear || quarterCenturyYear);
}
```

### if/else 보다는 다형성 사용
```java
// ❌ 반복되는 switch 문
public class BirdBehavior {
    public void fly(Bird bird) {
        switch (bird.getType()) {
            case EAGLE: flyHigh(); break;
            case PENGUIN: cannotFly(); break;
            case SPARROW: flyLow(); break;
        }
    }
    
    public void makeSound(Bird bird) {
        switch (bird.getType()) {  // 같은 분기 조건 반복
            case EAGLE: screech(); break;
            case PENGUIN: trumpet(); break;
            case SPARROW: chirp(); break;
        }
    }
}

// ✅ 다형성으로 해결
public abstract class Bird {
    public abstract void fly();
    public abstract void makeSound();
}

public class Eagle extends Bird {
    public void fly() { flyHigh(); }
    public void makeSound() { screech(); }
}
```

### 조건을 캡슐화하라
```java
// ❌ 복잡한 조건이 드러남
if (timer.hasExpired() && !timer.isRecurrent()) {
    timer.delete();
}

// ✅ 조건을 의미있는 함수로 캡슐화
if (shouldBeDeleted(timer)) {
    timer.delete();
}

private boolean shouldBeDeleted(Timer timer) {
    return timer.hasExpired() && !timer.isRecurrent();
}
```

### 함수는 한 가지만 해야 한다
```java
// ❌ 여러 가지 일을 하는 함수
public void pay() {
    for (Employee e : employees) {
        if (e.isPayday()) {
            Money pay = e.calculatePay();
            e.deliverPay(pay);
        }
    }
}

// ✅ 각각 하나의 일만 하는 작은 함수들
public void pay() {
    for (Employee e : employees) {
        payIfNecessary(e);
    }
}

private void payIfNecessary(Employee e) {
    if (e.isPayday()) {
        calculateAndDeliverPay(e);
    }
}

private void calculateAndDeliverPay(Employee e) {
    Money pay = e.calculatePay();
    e.deliverPay(pay);
}
```

### 숨겨진 시간적 결합
```java
// ❌ 시간적 결합이 숨겨짐
public class MoogDriver {
    public void dive(String reason) {
        saturateGradient();     // 이 순서가 중요하지만
        reticulateSplines();    // 코드만 봐서는 알 수 없음
        diveForMoog(reason);
    }
}

// ✅ 시간적 결합을 명시적으로 드러냄
public class MoogDriver {
    public void dive(String reason) {
        Gradient gradient = saturateGradient();
        List<Spline> splines = reticulateSplines(gradient);
        diveForMoog(splines, reason);
    }
}
```

### 경계 조건을 캡슐화하라
```java
// ❌ 경계 조건이 여기저기 흩어짐
if (level + 1 < tags.length) {
    parts = new Parse(body, tags, level + 1);
    body = null;
}

// 다른 곳에서도
processNextLevel(level + 1);  // +1이 반복됨

// ✅ 경계 조건을 한 곳에서 처리
int nextLevel = level + 1;
if (nextLevel < tags.length) {
    parts = new Parse(body, tags, nextLevel);
    body = null;
}
```

---

## ☕ 자바

### 긴 import 목록을 피하고 와일드카드 사용
```java
// ❌ 긴 import 목록
import java.util.List;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
import java.util.HashSet;

// ✅ 와일드카드 사용 (패키지에서 2개 이상 사용시)
import java.util.*;
```

### 상수는 상속하지 않는다
```java
// ❌ 상수를 인터페이스 상속으로 사용
public interface PayrollConstants {
    int HOURS_PER_WEEK = 40;
    double OVERTIME_RATE = 1.5;
}

public class HourlyEmployee extends Employee implements PayrollConstants {
    // HOURS_PER_WEEK 사용 - 어디서 왔는지 불분명
}

// ✅ static import 사용
public class PayrollConstants {
    public static final int HOURS_PER_WEEK = 40;
    public static final double OVERTIME_RATE = 1.5;
}

import static PayrollConstants.*;

public class HourlyEmployee extends Employee {
    // HOURS_PER_WEEK 사용 - 명확한 출처
}
```

### 상수 대 Enum
```java
// ❌ 옛날 방식의 상수
public static final int SPRING = 0;
public static final int SUMMER = 1;
public static final int FALL = 2;
public static final int WINTER = 3;

// ✅ 타입 안전한 Enum
public enum Season {
    SPRING, SUMMER, FALL, WINTER;
    
    public boolean isWarm() {
        return this == SPRING || this == SUMMER;
    }
}
```

---

## 🏷️ 이름

### 서술적인 이름을 사용하라
```java
// ❌ 의미없는 이름
int d; // elapsed time in days
List<int[]> list1 = new ArrayList<int[]>();

// ✅ 서술적인 이름
int elapsedTimeInDays;
List<Cell> gameBoard = new ArrayList<Cell>();
```

**중요**: 소프트웨어 가독성의 90%는 이름이 결정합니다.

### 적절한 추상화 수준에서 이름 선택
```java
// ❌ 구현을 드러내는 이름
public class ModemConnectController {
    private String phoneNumber;  // 구현 세부사항
    private Modem modem;         // 구현 세부사항
}

// ✅ 추상화 수준에 맞는 이름
public class CommunicationController {
    private String address;
    private Connection connection;
}
```

### 긴 범위는 긴 이름을 사용하라
```java
// ✅ 범위에 따른 적절한 이름 길이
public class UserAccountManagementService {
    private static final int DEFAULT_SESSION_TIMEOUT_IN_MINUTES = 30;
    
    public User authenticateUser(String username, String password) {
        for (int i = 0; i < attempts.length; i++) {  // 짧은 범위는 짧은 이름
            // ...
        }
        
        return authenticatedUserWithFullPermissions;  // 긴 범위는 긴 이름
    }
}
```

### 이름으로 부수 효과를 설명하라
```java
// ❌ 부수 효과를 숨기는 이름
public ObjectOutputStream getOos() throws IOException {
    if (m_oos == null) {
        m_oos = new ObjectOutputStream(m_socket.getOutputStream());
    }
    return m_oos;
}

// ✅ 부수 효과를 드러내는 이름
public ObjectOutputStream createOrReturnOos() throws IOException {
    if (m_oos == null) {
        m_oos = new ObjectOutputStream(m_socket.getOutputStream());
    }
    return m_oos;
}
```

---

## 🧪 테스트

### 불충분한 테스트
```java
// ❌ 불충분한 테스트
@Test
public void testCalculateArea() {
    Rectangle rect = new Rectangle(5, 10);
    assertEquals(50, rect.calculateArea());
}

// ✅ 충분한 테스트 - 경계 조건까지 확인
@Test
public void testCalculateArea() {
    // 정상 케이스
    Rectangle rect = new Rectangle(5, 10);
    assertEquals(50, rect.calculateArea());
    
    // 경계 조건
    Rectangle zeroRect = new Rectangle(0, 10);
    assertEquals(0, zeroRect.calculateArea());
    
    // 예외 케이스
    assertThrows(IllegalArgumentException.class, 
                () -> new Rectangle(-5, 10));
}
```

### 경계 조건을 테스트하라
```java
@Test
public void testBoundaryConditions() {
    // 최소값
    assertTrue(isValidAge(0));
    
    // 최대값  
    assertTrue(isValidAge(150));
    
    // 경계 바로 밖
    assertFalse(isValidAge(-1));
    assertFalse(isValidAge(151));
    
    // 일반적인 실수가 일어나는 지점
    assertTrue(isValidAge(18));  // 성인 기준
    assertTrue(isValidAge(65));  // 노인 기준
}
```

### 버그 주변은 철저히 테스트하라
```java
// 한 함수에서 버그를 발견했다면
@Test
public void testParseDate_FoundBugHere() {
    // 버그가 발견된 케이스
    Date result = DateParser.parse("2023-02-29");  // 윤년 아닌데 2월 29일
    assertNull(result);
    
    // 관련된 모든 케이스도 테스트
    assertNotNull(DateParser.parse("2024-02-29"));  // 윤년인 경우
    assertNull(DateParser.parse("2023-02-30"));     // 존재하지 않는 날
    assertNull(DateParser.parse("2023-13-01"));     // 존재하지 않는 월
    // ... 더 많은 관련 테스트
}
```

### 테스트는 빨라야 한다
```java
// ❌ 느린 테스트
@Test
public void testUserCreation() {
    // 실제 데이터베이스 연결
    Connection conn = DriverManager.getConnection("jdbc:mysql://...");
    // 실제 파일 시스템 사용
    File tempFile = new File("/tmp/test-" + System.currentTimeMillis());
    // ...
}

// ✅ 빠른 테스트
@Test  
public void testUserCreation() {
    // 메모리 기반 모킹
    UserRepository mockRepo = mock(UserRepository.class);
    // 인메모리 데이터 구조 사용
    Map<String, User> testData = new HashMap<>();
    // ...
}
```

---

## 📊 냄새와 휴리스틱 요약표

| 카테고리 | 주요 규칙 | 핵심 원칙 |
|----------|-----------|-----------|
| **주석** | 부적절한 정보, 중복, 주석처리된 코드 제거 | 코드로 표현할 수 없는 것만 주석으로 |
| **환경** | 한 단계 빌드, 한 단계 테스트 | 간단하고 명확한 개발 환경 |
| **함수** | 적은 인수, 단일 기능, 플래그 인수 금지 | 작고 명확한 책임을 가진 함수 |
| **일반** | 중복 제거, 추상화 수준 일치, 조건 캡슐화 | DRY, SRP, 명확한 의도 표현 |
| **이름** | 서술적, 적절한 추상화 수준, 부수효과 표현 | 의도를 명확히 드러내는 이름 |
| **테스트** | 충분한 커버리지, 경계 조건, 빠른 실행 | 신뢰할 수 있고 유지보수 가능한 테스트 |

---

## 🎯 결론

### 가치 체계의 중요성

> **"일군의 규칙만 따른다고 깨끗한 코드가 얻어지지 않습니다. 휴리스틱 목록을 익힌다고 소프트웨어 장인이 되지는 못합니다."**

### 전문가 정신과 장인 정신

이 장에서 소개한 냄새와 휴리스틱들은 **가치 체계**를 보여줍니다. 진정한 전문가가 되기 위해서는:

1. **가치 기반 접근**: 단순한 규칙 따르기가 아닌 근본적인 가치 이해
2. **규율과 절제**: 일관된 기준으로 코드 품질 유지  
3. **지속적 개선**: 보이스카우트 규칙의 실천
4. **경험을 통한 직관**: 휴리스틱을 체화하여 자연스러운 판단력 기르기

### 실무 적용 가이드

1. **일상적 점검**: 코드 리뷰 시 이 체크리스트 활용
2. **우선순위**: 중복 제거와 명확한 이름 붙이기부터 시작
3. **팀 차원**: 공통된 가치 체계 구축과 일관된 적용
4. **지속적 학습**: 새로운 냄새 패턴 발견 시 목록에 추가

**최종 메시지**: 깨끗한 코드는 기술이 아닌 **마음가짐**입니다. 장인 정신을 가지고 지속적으로 개선해 나가는 것이 진정한 전문가의 자세입니다.