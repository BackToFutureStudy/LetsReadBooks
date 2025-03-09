## 11장 시스템

도시의 큰 그림을 이해하지 못하더라도, 
각 분야를 관리하는 '구성요소' 관리팀들이 효율적으로 돌아간다.

소프트웨어 시스템도 도시와 같이 관심사를 분리하고 모듈화를 고려해야 한다.
코드를 깨끗하게 구현하면 낮은 추상화에서부터 관심사 분리가 쉬워진다.

### 시스템 제작과 사용을 분리하라
애플리케이션 객체를 제작하고 의존성을 서로 ‘연결’하는, "소프트웨어 제작 과정"과
준비 과정 이후에 이어지는, "런타임에서의 사용"은 분리되어야한다.
마구 뒤섞이면 안된다.

현실적으로 한 객체 유형이 모든 문맥에 적합할 가능성은 없다.
시스템 생성과 시스템 사용을 분리하려면
- `Main` 함수를 활용하여 객체 생성을 담당하고, 애플리케이션은 이를 단순히 사용하도록 구성해야한다.
객체가 생성되는 시점을 애플리케이션이 결정해야한다면
- `팩토리` `ABSTRACT FACTORY 패턴`을 사용한다.

### 의존성 주입
사용과 제작을 분리하는 강력한 메커니즘 하나로,
전적으로 보조 책임만을 넘겨 받은 새로운 객체는 '단일 책임 원칙 Single Responsibility Principle SRP'을 지킨다.
인스턴스 만드는 책임은 지지 않기 때문에 그 부분은 '전담' 매커니즘에 넘긴다. 

- **의존성 주입 (DI, Dependency Injection)** 을 활용하여 런타임 로직과 객체 생성을 분리할 수 있음.
- DI 컨테이너를 사용하면 초기화 지연과 같은 기법을 보다 체계적으로 적용 가능.

('JNDI', 'EJB1', 'EJB2' 아키텍처, 'JBoss', 'POJO', 'Colyer' 등 모르는 개념, 용어들이 많아서 추가 공부가 더 필요할 것 같다.)

---
- JNDI (Java Naming and Directory Interface): 자바에서 객체를 이름으로 찾아서 사용할 수 있도록 도와주는 시스템.
- EJB1 (Enterprise JavaBeans 1.0): 초창기 자바 엔터프라이즈 개발을 위한 기술로, 분산 시스템에서 컴포넌트를 쉽게 관리하도록 만든 것.
- EJB2 (Enterprise JavaBeans 2.0): EJB1의 개선 버전으로, 성능과 구조가 좀 더 발전한 자바 서버 기술.
- JBoss: 자바 기반 웹 애플리케이션을 실행할 수 있는 오픈소스 서버 소프트웨어.
- POJO (Plain Old Java Object): 특별한 기술 없이 단순한 자바 객체로 설계된 일반적인 클래스.
- Colyer (Adrian Colyer): AOP(Aspect-Oriented Programming, 관점 지향 프로그래밍)를 발전시킨 유명한 소프트웨어 엔지니어.
---


#### 예제 코드: 의존성 주입
```java
public class Service {
    private Repository repository;

    public Service(Repository repository) {
        this.repository = repository;
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        Repository repo = new DatabaseRepository();
        Service service = new Service(repo);
    }
}
```

### 시스템의 확장과 유지보수
- 관심사를 분리하고 유연한 설계를 도입해야 시스템 확장이 쉬워진다.
- **AOP (Aspect-Oriented Programming)**, **DI**, **팩토리 패턴** 등을 활용하여 관심사를 적절히 분리해야한다.

### 결론
- 깨끗한 시스템을 만들기 위해서는 **의존성 관리, 관심사 분리, 테스트 가능성 확보** 등이 중요하다.
- 복잡성을 줄이기 위한 구조적인 접근이 필요하다.

