# Chapter 10. 클래스

이전 장에서는 코드 행과 코드 블록을 올바르게 작성하는 방법, 함수 구현 방식, 그리고 함수 간의 관계에 대해 다루었다. 이번 장에서는 한 단계 더 나아가, **깨끗한 클래스를 만드는 원칙**에 대해 살펴본다.



## 클래스 체계

클래스를 정의할 때는 표준적인 자바 관례를 따르는 것이 좋다.

1. **변수 선언 순서**
   - `public static final` 정적 상수 → 맨 처음
   - `private static final` 정적 비공개 변수 → 다음
   - `private` 인스턴스 변수 → 그다음
   - **공개 변수는 거의 필요하지 않음**
2. **메서드 선언 순서**
   - **공개(public) 메서드**가 가장 먼저
   - **비공개(private) 메서드**는 자신을 호출하는 공개 메서드 직후에 배치

이처럼 클래스 내부의 구성 요소를 논리적인 순서로 정리하면, **위에서 아래로 읽기 쉬운 구조(신문 기사처럼 추상화 단계가 자연스럽게 내려가는 구조)**가 된다.



## 클래스는 작아야 한다!

클래스를 설계할 때 가장 중요한 원칙은 **크기를 작게 유지하는 것**이다.

- 함수 크기는 **물리적인 코드 행 수**로 판단할 수 있지만,
- 클래스 크기는 **책임의 수**로 판단해야 한다.



❌ **잘못된 방법(책임이 많은 클래스)**

```java
public class SuperDashboard extends JFrame implments MetaDateUser {
  public Component getLastFocusedComponent();
  public void setLastFocused(Component lastFocused);
  public int getMajorVersionNumber();
  public int getMinorVersionNumber();
  public int getBuildNumber();
}
```

위 클래스는 **메서드 수는 적지만, 너무 많은 책임을 맡고 있다.**

- UI 관련 기능 (`getLastFocusedComponent`, `setLastFocused`)
- 버전 관리 기능 (`getMajorVersionNumber`, `getMinorVersionNumber`, `getBuildNumber`)

➡ 클래스 이름에 `Processor`, `Manager`, `Super` 등이 들어간다면 **책임이 많은 클래스일 가능성이 높음**



### 단일 책임 원칙(Single Responsibility Principle, SRP)

> **"클래스나 모듈은 변경할 이유가 하나뿐이어야 한다."**



위의 `SuperDashboard` 클래스는 **두 가지 이유로 변경될 가능성이 있음**:

1. 소프트웨어의 버전 관리 로직이 변경될 경우
2. UI(스윙 컴포넌트) 관리 방식이 변경될 경우

➡ 이 클래스는 SRP를 위반하고 있음. **책임을 분리해야 한다.**



✅ **올바른 설계 예시 (책임 분리)**

이제 `Version` 클래스를 **독립적인 모듈**로 사용할 수 있으며, `SuperDashboard`에서 불필요한 책임을 제거할 수 있다.

```java
// 단일 책임 클래스 예
public class Version {
  public int getMajorVersionNumber();
  public int getMinorVersionNumber();
  public int getBuildNumber();
}
```



이처럼 '<u>돌아가는 소프트웨어</u>'를 만드는 것이 우선하는 것이 맞지만, 이후에 '<u>깨끗하고 체계적인 소프트웨어</u>'를 만드는 작업도 반드시 필요하다. 



정리하자면, **작은 클래스는 각각 하나의 책임만 맡고, 서로 협력하여 시스템을 구성해야 한다.**



### 응집도(Cohesion)

> 응집도가 높다는 말은 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미이다.

- **클래스 내부에서 모든 메서드가 인스턴스 변수를 활용하는 것이 이상적**
- 메서드가 너무 적은 변수를 사용하면, 응집도가 낮아지고 분리할 필요가 있음
- 일반적으로 <u>메서드가 인스턴스 변수를 더 많이 사용할수록 해당 클래스는 응집도가 높다</u>.



✅ **올바른 설계**: 클래스 내부에서 모든 변수와 메서드가 **서로 밀접한 관련이 있는 구조**

➡ 클래스가 점점 커지고 응집도가 낮아진다면? **여러 클래스로 분리해야 함.**



### 응집도를 유지하면 작은 클래스 여럿이 나온다

'함수를 작게, 매개변수 목록을 짧게'라는 전략을 따르다보면 때때로 몇몇 메서드만이 사용하는 인스턴스 변수가 아주 많아진다. 이는 십중팔구 새로운 클래스로 쪼개야 한다는 신호다. 

예를 들어, 큰 함수를 작은 함수로 쪼갤 때 변수들을 인스턴스 변수로 바꾼다면 함수를 쪼개기 쉬워진다. 하지만 해당 변수를 몇몇 함수만 사용하는 인스턴스 변수가 늘어나기 때문에 응집도는 낮아진다. 이는 클래스를 쪼갤 수 있다는 의미이다.

이처럼 클래스 응집도가 낮아진다면 여러 클래스로 쪼갠다. 그러면서 프로그램은 점점 더 체계가 잡히고, 작은 단위의 클래스로 서로 협력해 시스템에 필요한 동작을 수행할 수 있게 된다.



## 변경하기 쉬운 클래스

시스템은 **지속적으로 변경**된다. 따라서, 변경이 용이한 구조를 갖춰야 한다.



❌ **잘못된 방법(변경하기 어려운 클래스)**

* `UPDATE`문을 지원하지 않은 미완성 클래스인데, 이를 새로 추가하려면 클래스에 변경이 필요하다.
* 기존 메서드 하나를 수정할 때도 반드시 해당 클래스를 변경해야 한다.
* 이렇듯 변경할 이유가 두 가지이므로 `Sql` 클래스는 SRP를 위반한다.

```java
public class Sql {
  public Sql(String table, Colum[] columns)
  public String create()
  public String insert(Object[] fields)
  public String selectAll()
  public String findByKey(String keyColumn, String keyValue)
  public String select(Column column, String pattern)
  public String select(Criteria criteria)
  public String preparedInsert()
  private String columnList(Column[] columns)
  private String valueList(Object[] fields, final Column[] columns)
  private String selectWithCriteria(String criteria)
  private String placeholderList(Column[] columns)
}
```



✅ **옳은 방법(확장 가능한 구조)** - 하나의 클래스에서 제공하던 인터페이스를 `Sql` 추상 클래스를 상속받아 개별적인 클래스로 만듬

* 메서드를 새로 추가하려면 `Sql` 추상 클래스를 상속받아 만들면 된다.
* 메서드를 고치고 싶으면 해당 클래스에서 고치면 된다. 다른 클래스에 영향을 미치지 않는다.
* 클래스가 단순하기 때문에 테스트 관점에서도 모든 논리를 구석구석 증명하기도 쉬워지게 된다.

```java
abstract public class Sql {
  public Sql(String table, Column[] columns)
  abstract public String generate();
}

public class CreateSql extends Sql {
  public CreateSql(String table, Column[] columns)
  @Override public String generate()
}

public class SelectSql extends Sql {
  public SelectSql(String table, Column[] columns)
  @Override public String generate()
}

public class InsertSql extends Sql {
  public InsertSql(String table, Column[] columns, Object[] fields)
  @Override public String generate()
  private String valueList(Object[] fields, final Column[] columns)
}

public class SelectWithCriteriaSql extends Sql {
  public SelectWithCriteriaSql(String table, Column[] columns, Criteria criteria)
  @Override public String generate()
}

public class SelectWithMatchSql extends Sql {
  public SelectWithMatchSql(String table, Column[] columns, String pattern)
  @Override public String generate()
}

public class FindBykeySql extends Sql {
  public FindBykeySql(String table, Column[] columns, String keyColumn, String keyValue)
  @Override public String generate()
}

public class PreparedInsertSql extends Sql {
  public PreparedInsertSql(String table, Column[] columns)
  @Override public String generate()
  private String placeholderList(Column[] columns)
}

public class Where {
  public Where(String criteria)
  public String generate()
}

public class ColumnList {
  public ColmnList(Column[] columns)
  public String generate()
}
```



위 ✅ **옳은 방법)** 코드는 SRP뿐 아니라, OCP(Open-Closed Principle)도 지원한다.
**OCP란 클래스는 확장에 개방적이고, 수정에 폐쇄적이어야 한다는 원칙이다.**
위의 코드는 `Sql` 클래스를 파생해 새로운 클래스를 만드는 방식으로, 새 기능을 추가하는 것은 개방적인 동시에, 수정에는 다른 클래스의 영향이 없으므로 OCP 원칙이 적용되는 것이다.

<u>이처럼 이상적인 시스템이라면 새 기능을 추가할 때 시스템을 확장할 뿐, 기존 코드를 변경하지 않아야 한다.</u>



### 변경으로부터 격리

<u>요구사항은 변하기 마련이다.</u> 따라서 코드도 그에 따라 변한다. 
상세한 구현에 의존하는 클라이언트 클래스는 OCP 원칙이 위반할 가능성이 커진다. 왜냐하면 요구사항에 따라 구현이 바뀌게 되면 의존하던 클래스도 동시에 바뀔 가능성이 크기 때문이다. **그래서 인터페이스와 추상 클래스를 사용해 구현에 미치는 영향을 줄여야 한다.**



✅ **전략 패턴을 이용한 예시** (DIP: Dependency Inversion Principle 준수) - 주식을 계산하는 클래스

`Portfolio` 클래스는 특정 주식 거래소(`StockExchange`)에 의존하지 않고 **추상화된 인터페이스**에 의존 
➡ 새로운 주식 거래소 API가 추가되어도 `Portfolio` 클래스는 변경되지 않음

```java
public interface StockExchange {
  Money currentPrice(String symbol);
}

public Portfolio {
  private StockExchange exchage;
  public Portfolio(StockExchange exchange) {
    this.exchange = exchange;
  }
  ...
}
```



✅ **테스트 가능성 증가** - [테스트] 특정 주식(전략, 구현)에 의존하지 않은 덕분에, 여러 전략을 마음껏 테스트할 수 있다.

아래 코드에서는 마이크로소프트 주가(`MSFT`)를 테스트하였다.

* 인터페이스를 사용해 클라이언트(`Portfolio` 클래스)는 자세한 내용은 모른채 추상화에 의존한다.
* 구현체(전략)를 바꾸더라도 클라이언트의 코드는 바뀌지 않는다. 하지만 요구사항에 맞춰 로직(전략)은 쉽게 바꿀 수 있다.

```java
public class PortfolioTest {
  private FixedStockExchangeStub exchane;
  private Portfolio portfolio;

  @Before
  protected void setUp() throws Exception {
    exchange = new FixedStockExchangeStub();
    exchange.fix("MSFT", 100);
    portfolio = new Portfolio(exchange);
  }

  @Test
  public void GivenFiveMSTFTotalShouldBe500() throws Exception {
    portfoilo.add(5, "MSFT");
    Assert.assertEquals(500, portfolio.value());
  }
}
```



위와 같이 테스트가 가능할 정도로 시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다.
결합도가 낮다는 것은 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다는 의미다.
이렇게 결합도를 최소로 줄이면, 또 다른 클래스 설계 원칙인 **DIP(Dependency Inversion Principle)**를 자연스럽게 따르게 된다. **DIP는 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙이다.**



## 핵심 정리

1. **클래스는 작아야 한다.** 하나의 책임만 가지도록 설계하라. (SRP 원칙 준수)
2. **응집도를 높여라.** 모든 메서드가 관련된 인스턴스 변수를 사용해야 한다.
3. **OCP(개방-폐쇄 원칙)를 준수하라.** 확장은 쉽게, 기존 코드 수정은 최소화할 것.
4. **DIP(의존성 역전 원칙)를 따르라.** 구체적인 구현이 아니라 **추상화(인터페이스)**에 의존하도록 설계하라.

➡ **결과적으로 변경에 강하고, 유지보수하기 쉬운 깨끗한 클래스를 만들 수 있다.**