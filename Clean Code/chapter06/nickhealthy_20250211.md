# Chapter 6. 객체와 자료 구조

변수를 비공개(`private`)로 정의하는 이유

* 남들이 변수에 의존하지 않게 만들고 싶어서다.



## 자료 추상화

* 구현을 감추려면 **추상화**가 필요하다.
* 추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스다.
  * 조회 함수(`Getter`)와 설정 함수(`Setter`)로 변수를 다룬다고 클래스가 되지는 않는다.
  * 자료를 세세하게 공개하기보다는 **추상적인 개념으로 표현하는 편이 좋다.**
* 개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 심각하게 고민해야 한다.
* 예제 1

```java
// 구체적인 Point 클래스
public class Point {
  public double x;
  public double y;
}

// 추상적인 Point 클래스
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y);
  double getR();
  double getTheta();
  void setPolar(double r, double theta);
}
```

* 예제 2

```java
// 구체적인 Vehicle 클래스
public interface Vehicle {
    double getFuelTankCapacityInGallons(); //자동차 연료 상태를 구체적인 숫자 값으로 알려줌
    double getGallonsOfGasoline();
}

// 추상적인 Vehicle 클래스
public interface Vehicle {
    double getPercentFuelRemaining(); // 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려줌
}
```



## 자료/객체 비대칭

* 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 **함수만 공개**한다.(<u>내부 구조를 감춤</u>)
* 자료 구조는 **자료를 그대로 공개**하며 별다른 함수는 제공하지 않는다.(<u>내부 구조를 드러냄</u>)



### [절차적 구조]

* 각 도형 클래스는 간단한 자료 구조로 아무 메서드도 제공하지 않고, 자료를 그대로 공개한다.
* 동작 방식은 `Geometry` 클래스에서 구현한다.
* `Geometry` 클래스에 새로운 함수를 추가하고 싶다면?
  * 도형 클래스는 아무런 영향을 받지 않는다.
* 새 도형을 추가하고 싶다면? 
  * `Geometry` 클래스에 속한 함수를 모두 고쳐야 한다.

```java
public class Square {
    public Point topLeft;
    public double side;
}

public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public final double PI = 3.141592653589793;
    
    public double area(Object shape) throws NoSuchShapeException {
        if(shape instanceof Square){
            Square s = (Square)shape;
            return s.side * s.side;
        }
        else if(shape instanceof Rectangle){
            Rectangle r = (Rectangle)shape;
            return r.height * r.width;
        }
        else if(shape instanceof Circle){
            Circle c = (Circle)shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}
```



### [객체 지향적인 구조]

* `area()`는 다형 메서드이므로 `Geometry` 클래스가 필요 없다.
* 새 도형을 추가하고 싶다면?
  * 기존 함수에 아무런 영향을 미치지 않는다.
* 새 함수를 추가하고 싶다면?
  * 도형 클래스를 전부 고쳐야한다.

```java
public class Square implements Shape {
    private Point topLeft;
    private double side;
    
    public double area(){
        return side * side;
    }
}

public class Rectangle implements Shape {
    private Point topLeft;
    private double height;
    private double width;
    
    public double area(){
        return height * widht;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;
    
    public double area(){
        return PI * radius * radius;
    }
}
```



### [절차지향 vs 객체지향 코드]

객체 지향 코드에서 어려운 변경(함수 추가)은 절차 절차적인 코드에서 쉽고, 절차적인 코드에서 어려운 변경(새로운 자료구조)은 객체 지향 코드에서 쉽다.

* 절차지향 코드
  * 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다.
  * 새로운 자료 구조를 추가하기 어렵다. 만약 추가하려면 모든 함수를 고쳐야 한다.
* 객체지향 코드
  * 기존 함수를 변경하지 않으면서 새 클래스(자료구조)를 추가하기 쉽다.
  * 새로운 함수를 추가하기 어렵다. 만약 추가하려면 모든 클래스를 고쳐야 한다.



## 디미터 법칙

**"최소 지식 원칙 (Principle of Least Knowledge)"**이라고도 불리며, 객체 지향 프로그래밍(OOP)에서 **객체 간의 결합도를 낮추고, 캡슐화를 유지**하는 설계 원칙이다.

* 이 법칙의 핵심 개념은 **객체가 다른 객체의 내부 구조에 대해 너무 많이 알면 안 된다는 것**이다.
*  즉, **직접적인 관계가 없는 객체의 메서드를 호출하지 말라**는 원칙이다.



### 디미터 법칙의 규칙

디미터 법칙은 특정 객체가 메서드 내에서 직접 접근할 수 있는 대상에 대한 제한을 둔다. 다음 규칙을 따른다.

1. 자신의 메서드 및 필드만 접근 가능
   * `this` 객체 내부의 메서드와 필드만 직접 호출할 수 있음
2. 메서드의 매개변수로 전달된 객체의 메서드만 접근 가능
   * 메서드 호출 시 매개변수로 받은 객체에 대해서만 접근 가능 
3. 객체가 직접 생성한 인스턴스의 메서드만 접근 가능
   * 객체 내부에서 생성한 다른 객체(Composition 관계)는 접근 가능
4. 전역 객체(`Static` 변수 등)는 접근하지 않는 것이 좋음
   * 전역 객체는 어디서든 접근할 수 있으므로 결합도를 높일 위험이 있음



### 디미터 법칙의 추구하는 방향

* 객체 간의 결합도(Coupling) 최소화
  * 하나의 객체가 너무 많은 객체에 의존하지 않도록 하여 코드 변경 시 영향도를 줄임
* 높은 응집도(Cohesion) 유지
  * 객체가 자기 자신과 직접적으로 관련된 기능만 수행하도록 유도함
* 캡슐화(Encapsulation) 강화
  * 불필요한 객체 접근을 줄이고, 내부 구현을 숨겨 객체간 독립성을 보장함
* 유지보수성(Maintainability) 향상
  * 특정 객체의 변경이 다른 객체에 미치는 영향을 최소화하여 코드 수정이 쉬워짐



### 디미터 법칙의 적용 및 주의점

* 객체를 반환하는 Getter 메서드를 줄인다.
  * 대신 캡슐화된 메서드를 제공한다.
* 메서드 체이닝(Method Chaining) 사용을 주의한다.
  * `obj.getA().getB().getC().doSomething();` -> 이런 코드가 있다면 설계를 다시 고려
  * `final String outputDir = ctxt.options.scratchDir.absolutePath;` -> 자료 구조를 표현하는 것은 내부 구조를 노출하는 것이 맞으므로 디미터 법칙이 적용되지 않는다.
* 의존성 주입(Dependency Injection, DI) 활용
  * Spring의 `@Autowired`나 생성자 주입 방식을 사용해서 직접적인 객체 생성을 피하기



### 예제코드

❌ **Demeter 법칙을 위반한 코드 (Bad Example)**

```java
class Engine {
    public void start() {
        System.out.println("Engine started.");
    }
}

class Car {
    private Engine engine;

    public Car() {
        this.engine = new Engine();
    }

    public Engine getEngine() {  // 엔진 객체를 직접 노출 (위반)
        return engine;
    }
}

class Driver {
    public void startCar(Car car) {
        car.getEngine().start();  // Car를 통해 Engine의 내부 메서드 호출 (위반)
    }
}

```

❌ `Driver` 클래스가 `Car`의 내부 구조(엔진)를 직접 참조하므로 **Demeter 법칙 위반**



✅ **Demeter 법칙을 지킨 코드 (Good Example)**

```java
class Engine {
    public void start() {
        System.out.println("Engine started.");
    }
}

class Car {
    private Engine engine;

    public Car() {
        this.engine = new Engine();
    }

    public void start() {  // 엔진을 직접 노출하지 않고, Car가 책임지도록 함
        engine.start();
    }
}

class Driver {
    public void startCar(Car car) {
        car.start();  // 엔진에 직접 접근하지 않음
    }
}

```

✅ `Car` 클래스가 `Engine`의 기능을 캡슐화하여 **Driver는 내부 구조를 몰라도 됨**



### 정리

디미터 법칙은 객체 간의 결합도를 줄이고 캡슐화를 강화하는 중요한 원칙이다. 
하지만 무조건적인 적용보다는 **유지보수성, 가독성, 확장성을 고려하여 적절한 균형을 찾는 것이 중요**하다.



##  자료 전달 객체

자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다.
이런 자료 구조체를 때로는 **자료 전달 객체(Data Transfer Obeject, DTO)**라 한다.



### 활성 레코드

활성 레코드도 DTO와 같이 데이터를 담고 있고, 조회/설정(Getter, Setter) 함수가 있는 구조이다.
이외에도 DTO와 차이점이 있는데 다음과 같다.

|   구분    |               DTO                |                  활성 레코드                  |
| :-------: | :------------------------------: | :-------------------------------------------: |
|   목적    |   데이터를 전달하기 위한 객체    | 데이터를 표현하면서 동시에 비즈니스 로직 포함 |
|   특징    |   단순히 데이터만 가지고 있음    |           데이터 + CRUD 기능을 포함           |
| 사용 방식 | 서비스, 리포지토리 계층에서 활용 |        모델 객체 자체에서 데이터 처리         |

즉, 활성 레코드는 데이터를 담는 역할을 하지만 데이터베이스 테이블이나 다른 소스에서 얻은 자료를 핸들링 할 수 있어 일반적인 DTO보다 더 많은 책임을 가지게 된다.



### 🔹 활성 레코드 예제 (Java + JPA)

대표적인 예제는 JPA의 엔티티 클래스다. 예를 들어 다음과 같은 엔티티가 있다.

```java
@Entity
public class User extends BaseEntity {  // BaseEntity는 공통 필드를 가지는 부모 클래스
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // 기본 생성자
    protected User() {}

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // 활성 레코드 패턴 적용: 엔터티 내부에서 데이터 저장
    public void save(EntityManager em) {
        em.persist(this);
    }

    // 엔터티 내부에서 삭제 기능 포함
    public void delete(EntityManager em) {
        em.remove(this);
    }
}

```

**DTO처럼 데이터를 담지만, 동시에 스스로 저장/삭제 등의 동작을 수행**한다는 점에서 활성 레코드라고 할 수 있다.



### 활성 레코드 단점

1. 객체 지향 원칙 위반
   * 엔티티가 비즈니스 로직과 데이터 접근을 동시에 처리하므로 **SRP(단일 책임 원칙)를 위반**할 수 있다.
   * 원래 **데이터 저장/조회 등의 기능은 리포지토리 계층이 담당해야 하는데**, 활성 레코드에서는 엔티티 자체가 이를 수행함
2. 테스트가 어렵다
   * 데이터베이스 연결 없이 단위 테스트를 하려면, 별도의 모킹(Mock) 처리가 필요할 수 있음
3. 확장성이 떨어짐
   * 서비스 계층이 복잡해질 경우, 엔티티가 너무 많은 역할을 맡게 되어 유지보수가 어려워질 수 있음



### 활성 레코드 vs. 리포지토리 패턴

대부분의 **Spring + JPA 프로젝트는 활성 레코드 패턴을 쓰지 않고, 리포지토리 패턴을 사용**한다.
즉, 엔티티는 데이터를 표현하는 역할만 하고, DB와 직접적인 작업은 리포지토리 계층이 담당한다.

❌ 활성 레코드 방식

```java
User user = new User("Alice", "alice@example.com");
user.save(entityManager);
```

* 엔티티 자체가 저장 로직을 수행(객체가 DB 관련 로직을 가짐)



✅ 리포지토리 패턴 방식 (Spring JPA 표준 방식)

```java
User user = new User("Alice", "alice@example.com");
userRepository.save(user);
```

* `UserRepository`라는 별도의 계층에서 저장 로직을 수행



### 정리

* 활성 레코드는 DTO처럼 데이터를 담지만, CRUD 기능도 수행할 수 있는 객체
* 하지만 객체의 책임이 많아지고 유지보수가 어려워질 수 있기 때문에, 일반적으로 Repository 패턴을 사용
  * **데이터 저장/조회 등의 기능은 리포지토리 계층에서 담당**



## 결론

* 객체는 동작을 공개하고 자료를 숨긴다.
  * 기존 동작을 변경하지 않고 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 동작을 추가하기는 어렵다.
    * 상속, 인터페이스 구현 등등
* 자료 구조는 별다른 동작 없이 자료를 노출한다.
  * 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.
  * 자료 구조를 사용하는 모든 객체의 메서드에 새로운 자료 구조를 사용하도록 수정해야 한다.
* **시스템을 구현할 때 새로운 자료 타입을 추가하면 객체, 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 적합하다.**



