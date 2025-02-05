# 🔖 책 내용 정리

프로그래밍 초창기에는 시스템을 `루틴`과 `하위루틴`으로 나눴다.

어떤 프로그램이든 가장 기본적인 단위 : 함수

---

# 함수 설계 원칙 요약

1. **작은 함수가 좋은 함수다**
    - 함수는 짧아야 한다. 가능하면 20줄 이하가 이상적이며, 10줄 이하가 더욱 좋다.
    - 들여쓰기는 2단 내에서 하는 게 좋다. (if 안에 if 안에 if x)
    - 한 함수는 `한 가지 작업만` 수행해야 하며, 중복을 최소화해야 한다.
    - [switch 문을 간단하게 만들자 → abstract factory에 숨기자.](https://www.notion.so/3-18b1eb2dd4d1808d8572e6154056fa43?pvs=21)
2. **의미 있는 이름을 사용하라**
    - 함수 이름은 기능을 명확히 설명해야 한다.
    - 이름이 길어도, 서술적인 이름을 짓자.ㅎㅎ
    - 예: `testableHtml()` 대신 `renderPageWithSetupsAndTeardowns()`로 변경하여 의미를 명확히 표현한다.
3. **좋은 코드란 하나의 이야기처럼 읽혀야 한다**
    - 함수는 위에서 아래로 논리적으로 흐르도록 작성해야 한다.
        - 예제에서 `SetupTeardownIncluder` 클래스는 `render()` → `includeSetupAndTeardownPages()` → `includePageContent()` 순으로 구성되어 있어 읽기 쉽다.
4. **인수는 최소화하라**
    - 함수 인수는 0~2개가 이상적이며, 가능하면 단일 객체로 감싸야 한다.
    - 플래그 인수(예: `render(true)`)는 피하고, 별도 함수(`renderForSuite()`, `renderForSingleTest()`)로 분리해야 한다.
5. **Switch 문은 다형성으로 대체하라**
    - Switch 문은 객체 생성 시에만 사용하고, 이후에는 다형성을 활용하여 처리한다.
    - 예제에서는 `EmployeeFactory`를 사용해 `switch` 문을 숨겼다.
6. **명령과 조회를 분리하라**
    - 함수는 상태를 변경(명령)하거나 정보를 반환(조회)하는 둘 중 하나만 수행해야 한다.
    - 예: `if (set("username", "unclebob"))` → 의미가 모호하므로 `attributeExists()`와 `setAttribute()`로 분리해야 한다.
7. **오류 코드보다 예외를 사용하라**
    - 오류 코드를 반환하면 중첩된 `if` 문이 증가해 코드가 복잡해진다.
    - 예외 처리를 사용하면 오류 처리 코드와 비즈니스 로직이 분리되어 가독성이 좋아진다.
8. **Try/Catch 블록을 별도 함수로 추출하라**
    - 예외 처리는 별도 함수에서 수행하여 정상적인 흐름과 분리해야 한다.
    - 예: `delete()` 함수에서 `try` 블록을 분리해 `deletePageAndAllReferences()`와 `logError()`로 구성한다.

    ```java
    // Exception이 발생하면 여기서 잡아서 처리한다.
    // 오류 처리도 1가지 작업 -> 따라서 오류 처리하는 함수는 오류만 처리해야 한다.
    public void delete(Page page) {
        try {
            deletePageAndAllReferences(page);
        } catch (Exception e) {
            logError(e);
        }
    }
    
    private void deletePageAndAllReferences(Page page) throws Exception {
        deletePage(page);
        registry.deleteReference(page.name);
        configKeys.deleteKey(page.name.makeKey());
    }
    
    private void logError(Exception e) {
        logger.log(e.getMessage());
    }
    
    ```

9. **반복하지 마라 (DRY: Don’t Repeat Yourself)**
    - 같은 코드를 여러 번 작성하지 말고, 중복되는 부분을 별도 함수로 추출하라.
    - 예제 코드에서는 `includeSetupPages()`, `includeTeardownPages()` 등을 별도 함수로 분리해 중복을 제거했다.
10. **구조적 프로그래밍의 원칙을 지키되, 필요하면 return, break, continue를 활용하라**
    - 함수가 작다면 다중 return, break, continue를 활용해 코드 가독성을 높일 수 있다.
    - goto 문은 절대 사용하지 않는다.

### 결론

좋은 함수는 짧고, 한 가지 작업만 수행하며, 명확한 이름을 가진다.

깨끗한 코드는 읽기 쉬운 코드이며, 유지보수가 쉬운 코드다.

함수를 개선하려면 먼저 기능을 완성한 후, 점진적으로 정리하고 최적화해야 한다.

---

## switch 문 간결하게 만들고, SRP를 지키도록 하자.

switch 문은 작게 만들기 어렵다.

case 분기가 단 두 개인 switch 문도 내 취향에는 너무 길며, 단일 블록이나 함수를 선호한다.

또한 ‘한 가지’ 작업만 하는 switch 문도 만들기 어렵다.

본질적으로 switch 문은 N가지를 처리한다. 불행하게도 switch 문을 완전히 피할 방법은 없다.

하지만 각 switch 문을 `저차원 클래스에 숨기고` 절대로 반복하지 않는 방법은 있다.

물론 `다형성(polymorphism)`을 이용한다.

```java
public Money calculatePay(Employee e)
throws InvalidEmployeeType {
    switch (e.type) {
        case COMMISSIONED:
            return calculateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            throw new InvalidEmployeeType(e.type);
    }
}
```

<aside>

이 함수의 문제점

1. 함수가 길다. 새 직원 유형을 추가하면 더 길어진다.
2. 한 가지 작업만 수행하지 않는다.
3. SRP를 위반한다.
4. OCP를 위반한다.
    1. 새 직원 유형을 추가할 때마다 코드를 변경하기 때문이다.
</aside>

- 한가지 작업만 수행 vs SRP : 둘이 같은 거 아님?

  비슷하지만 정확히 같은 의미는 아닙니다. 차이를 정리하면 다음과 같습니다.

  ### 1. **한 가지 작업만 수행하지 않는다.**

    - **함수가 여러 개의 책임(작업)을 수행하는 경우**를 의미합니다.
    - 예제 코드에서는 `calculatePay()` 함수가 단순히 급여를 계산하는 것이 아니라 **직원 유형을 확인하고**, 그에 따라 다른 계산 로직을 호출하고 있습니다.
    - 즉, **여러 개의 기능을 포함하고 있어 한 가지 작업만 수행하지 않는다**고 볼 수 있습니다.

  ### 2. **SRP(Single Responsibility Principle, 단일 책임 원칙) 위반**

    - **클래스나 함수가 한 가지 책임(변경 이유)을 가져야 한다**는 원칙입니다.
    - `calculatePay()` 함수는 **새로운 직원 유형이 추가될 때마다 변경해야 하는 책임**을 가지게 됩니다.
    - 즉, 급여 계산 로직뿐만 아니라 **직원 유형을 분류하는 책임까지 가지므로 SRP를 위반**하게 됩니다.

  ### **결론: 차이점 정리**

  ✅ "한 가지 작업만 수행하지 않는다"는 단순히 함수가 여러 작업을 한다는 의미입니다.

  ✅ "SRP를 위반한다"는 **이로 인해 코드가 변경될 이유가 여러 가지 생긴다**는 의미입니다.

  즉, **한 가지 작업만 하지 않으면 SRP도 위반할 가능성이 높지만, SRP 위반은 더 넓은 개념**입니다.


### → 고친 부분

```java
// Employee.java
public abstract class Employee {
    public abstract boolean isPayday();
    public abstract Money calculatePay();
    public abstract void deliverPay(Money pay);
}

// EmployeeFactory.java
public interface EmployeeFactory {
    Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}

// EmployeeFactoryImpl.java
public class EmployeeFactoryImpl implements EmployeeFactory {
    @Override
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r);
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmployee(r);
            default:
                throw new InvalidEmployeeType(r.type);
        }
    }
}

```

### 변경 전과 변경 후의 차이점 분석

| 항목 | 변경 전 | 변경 후 | 개선 효과 |
| --- | --- | --- | --- |
| **파일 구조** | 모든 코드가 하나의 블록에 있음 | `Employee`, `EmployeeFactory`, `EmployeeFactoryImpl`로 분리 | 유지보수성 증가, 역할별 코드 구분 |
| **가독성** | 주석과 URL로 지저분함 | 불필요한 부분 제거, 정리된 코드 | 코드 가독성 증가 |
| **switch 문 스타일** | 블록 들여쓰기 불규칙 | 일관된 형식으로 정리 | 가독성 증가 |
| **인터페이스 구현 명확화** | `@Override` 없음 | `@Override` 추가 | 컴파일러 체크 가능, 오류 방지 |

기존에는 모든 코드가 하나의 클래스 내에서 처리되었다면,

각각 다른 클래스에게 역할을 부여함으로써 → 다른 클래스에서는 직원의 유형을 몰라도 해당 함수를 실행할 수 있게 됨.

--- 