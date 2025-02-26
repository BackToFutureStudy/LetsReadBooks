# 10장 클래스 
클린한 코드로 클린한 함수를 만들어도, 클린한 클래스까지 생각해야한다.

## 클래스 체계
1. **변수**는 맨 위에 선언.
2. **공개(public) 함수**를 먼저 선언.
3. **비공개(private) 함수**는 호출하는 공개 함수 바로 아래에 배치.
4. **추상화 수준을 순차적으로 낮춰야 함.**

### 캡슐화
변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 
테스트 코드한테까지 숨기란 건 아니며, 캡슐화가 깨지는 건 가장 최후의 보류로 둔다.


## 클래스 === 작아야 한다!
클래스가 맡는 책임에 맞춰서 명확한 클래스 이름을 만들어야한 모호한 책임을 지지 않는다.

## 단일 책임 원칙 (SRP Single Responsibility Principle)
- 클래스는 변경할 이유가 하나여야 한다.
- 한 클래스가 여러 책임을 가지면 유지보수가 어려워진다.

### AS-IS
```java
public class SuperDashboard {
  public String getCustomizerLanguagePath();
  public void setSystemConfigPath(String systemConfigPath);
  public boolean getGuruState();
  public void showObject(MetaObject object);
  public void addProject(Project project);
  // ... 많은 기능 포함 ...
}
```

### TO-BE
```java
public class Version {
  public int getMajorVersionNumber();
  public int getMinorVersionNumber();
  public int getBuildNumber();
}
```
- **SuperDashboard** 클래스는 너무 많은 책임을 가짐.
- **Version** 클래스로 버전 관리 역할을 분리하여 단일 책임을 유지.

## 응집도와 결합도
- **응집도(Cohesion)**: 하나의 클래스가 관련 있는 메서드와 변수를 묶어야 한다.
- **결합도(Coupling)**: 서로 다른 클래스 간의 의존성을 최소화해야 한다.

## 변경하기 쉬운 클래스
- 새로운 기능 추가 시 기존 클래스를 변경하지 않아야 한다.
- OCP(Open-Closed Principle) 적용하여 확장에는 열려 있고, 수정에는 닫혀 있어야 한다.

### 개선된 SQL 클래스
```java
abstract public class Sql {
  public Sql(String table, Column[] columns);
  abstract public String generate();
}

public class CreateSql extends Sql {
  public CreateSql(String table, Column[] columns) { super(table, columns); }
  @Override public String generate() { return "CREATE SQL"; }
}
```
- 새로운 기능을 추가할 때 기존 클래스를 수정하지 않아도 된다.
- **UpdateSql** 같은 새로운 클래스를 추가하면 기존 코드에 영향 없이 기능 확장 가능하다.

## DIP(Dependency Inversion Principle)
결합도가 낮다 = 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다.
- 구체적인 클래스가 아니라 추상화에 의존해야 한다.

### AS-IS
```java
public class Portfolio {
  private TokyoStockExchange exchange;
  public Portfolio() { this.exchange = new TokyoStockExchange(); }
}
```
- **TokyoStockExchange**에 직접 의존하여 변경이 어렵고 테스트하기 어려움.

### TO-BE
```java
public class Portfolio {
  private StockExchange exchange;
  public Portfolio(StockExchange exchange) { this.exchange = exchange; }
}
```
- **StockExchange** 인터페이스를 사용하여 유연성과 테스트 가능성을 높임.

## 결론
- 클래스는 작게 유지하고 단일 책임 SRP을 가져야 한다.
- 응집도를 높이고 결합도를 낮추는 OCP을 적용해야 유지보수가 용이하다.
- 변경이 발생할 가능성을 고려하는 DIP를 생각하며 설계해야 한다.
