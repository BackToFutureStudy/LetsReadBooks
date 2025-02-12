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

* 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 **함수만 공개**한다.
* 자료 구조는 **자료를 그대로 공개**하며 별다른 함수는 제공하지 않는다.



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



