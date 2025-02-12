# 6장 객체와 자료 구조

## 자료 추상화
- 변수에 의존하지 않게 하기 위해 private를 쓰지만 조회, 설정함수는 public을 쓰는지 알아본다. 

- 데이터 구조를 직접 노출하지 않고 추상화를 통해 감출 것. 개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 심각하게 고민해야 한다.

- 예제:

```java
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y);
  double getR();
  double getTheta();
  void setPolar(double r, double theta);
}
```

## 객체와 자료 구조의 차이
- 상호 보완적인 특질로 인해 객체와 자료 구조는 근본적으로 양분된다.
- 객체 지향 코드는 동작을 공개하고 데이터를 숨기기 때문에, 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다. 또 새로운 함수를 추가하기 어려워서 모든 클래스를 고쳐야 한다.
- 자료 구조(절차적인 코드)는 데이터를 공개하고 동작을 숨겨서, 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 또 새로운 자료 구조를 추가하기 어려워서 모든 함수를 고쳐야 한다.

## 절차적인 도형 vs 객체 지향적 도형
- 절차적인 방식:

```java
public class Square {
  public Point topLeft;
  public double side;
}
```

- 객체 지향적인 방식:

```java
public class Square implements Shape {
  private Point topLeft;
  private double side;

  public double area() {
    return side * side;
  }
}
```

## 디미터 법칙
- “클래스 C의 메서드 f는 [클래스 C
, f가 생성한 객체, f 인수로 넘어온 객체, C 인스턴스 변수에 저장된 객체]과 같은 객체의 메서드만 호출해야 한다”는 주장.

- 휴리스틱 heuristic 로, 객체는 자신의 내부 구조를 숨기고 외부 객체의 속성을 직접 탐색하면 안 됨.

- 예제(잘못된 방식(기차 충돌 train
wreck,잡종 구조 등)):

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

- 예제(올바른 방식):

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

## 자료 전달 객체(DTO)
- 자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스로, 자료 전달 객체 Data Transfer Object, DTO라 한다. 

- 데이터베이스와의 통신을 위해 사용되며, 일반적으로 getter/setter만 포함하고 형태로는 'Bean빈' 구조이다.

```java
public class Address {
  private String street;
  private String city;

  public String getStreet() {
    return street;
  }

  public String getCity() {
    return city;
  }
}
```

## 활성 레코드
- 데이터베이스 테이블을 객체로 표현하는 방식.
- 비즈니스 로직을 포함하는 것은 피해야 함.

## 결론
- 객체는 새 데이터 타입 추가가 쉬움, 자료 구조는 새 동작 추가가 쉬움.
- 프로젝트에 따라 적절한 방법을 선택할 것.
