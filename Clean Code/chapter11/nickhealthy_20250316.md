# Chapter 11. 시스템(Systems)

이 장은 코드 레벨을 넘어서 시스템 전체의 설계와 구조에 관한 중요한 원칙들을 다루고 있습니다.

## 시스템 구축과 사용의 분리

**이 장의 핵심 원칙 중 하나는 '시스템 구축'과 '시스템 사용'을 명확히 분리해야 한다는 것입니다.** 

소프트웨어 시스템은 애플리케이션 객체의 생성과 객체들의 의존성 연결을 '시작 단계'에서 처리하고, 그 후 '실행 단계'에서는 이미 준비된 객체들을 사용하는 방식으로 설계되어야 합니다.

이를 위한 여러 접근법이 소개됩니다:

1. **Main 분리**: 모든 설정 로직을 main 함수나 main이 호출하는 모듈에 두어 나머지 시스템에서는 모든 객체가 적절히 구성되었다고 가정하게 합니다.
2. **팩토리(Factory)**: 객체 생성 로직을 팩토리 클래스로 분리하여 시스템이 필요할 때 객체를 생성하도록 합니다.
3. **의존성 주입(Dependency Injection)**: 객체는 자신이 필요로 하는 의존성을 직접 생성하지 않고, 외부에서 주입받습니다. 이것은 객체 간 결합도를 낮추고 테스트 용이성을 높입니다.

### 왜 분리가 필요한가?

예를 들어 다음과 같은 코드를 생각해 봅시다:

```java
public Service getService() {
    if (service == null)
        service = new MyServiceImpl(...);
    return service;
}
```

이 코드는 지연 초기화(lazy initialization)의 예시인데, 얼핏 보기에는 효율적으로 보일 수 있습니다. 
하지만 이 패턴은 몇 가지 문제점을 가지고 있습니다:

1. `getService()` 메서드가 '구축'과 '사용'이라는 **두 가지 책임**을 동시에 가지게 됩니다.
2. 이 메서드를 사용하는 클라이언트는 실제로 서비스 생성 과정과 결합되어 있어, **테스트하기 어렵습니다.**
3. 전체 애플리케이션에서 이런 패턴이 반복되면 **의존성 관리가 복잡**해집니다.

### 해결책: 의존성 주입

이런 문제를 해결하기 위한 방법으로 의존성 주입(Dependency Injection)이 제시됩니다. 의존성 주입은 객체가 자신의 의존성을 직접 생성하는 대신, 외부에서 주입받는 방식입니다.

```java
public class MovieLister {
    private MovieFinder finder;
    
    // 생성자 주입
    public MovieLister(MovieFinder finder) {
        this.finder = finder;
    }
    
    public Movie[] moviesDirectedBy(String director) {
        List<Movie> movies = finder.findAll();
        // 필터링 로직
        return movies.toArray(new Movie[0]);
    }
}
```

이렇게 의존성 주입을 사용하면:

1. `MovieLister`는 `MovieFinder`의 구현체를 알 필요가 없습니다.
2. 테스트 시 Mock 객체를 쉽게 주입할 수 있습니다.
3. 애플리케이션 구성이 한 곳에서 관리되므로 변경이 용이합니다.

### 제어의 역전(IoC)과 의존성 주입 프레임워크

의존성 주입은 제어의 역전(Inversion of Control) 원칙의 한 형태입니다. 
전통적인 프로그래밍에서는 개발자가 흐름을 제어했지만, IoC에서는 프레임워크가 제어를 맡습니다.

스프링 프레임워크를 예로 들면:

```java
// 스프링 설정 파일(XML 또는 Java Config)
@Configuration
public class AppConfig {
    @Bean
    public MovieFinder movieFinder() {
        return new ColonDelimitedMovieFinder("movies.txt");
    }
    
    @Bean
    public MovieLister movieLister() {
        return new MovieLister(movieFinder());
    }
}

// 메인 애플리케이션
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MovieLister lister = context.getBean(MovieLister.class);
        Movie[] movies = lister.moviesDirectedBy("Christopher Nolan");
        // 결과 처리
    }
}
```

이 접근 방식은 애플리케이션 로직과 객체 생성/조립 로직을 완전히 분리합니다. 
**시스템 시작 시점에 모든 의존성이 해결되므로, 런타임에 의존성 문제가 발생할 가능성이 줄어듭니다.**

## 확장 - 점진적인 개선을 통한 시스템 설계

**시스템은 처음부터 완벽하게 설계될 수 없으므로, 시간이 지남에 따라 진화하고 확장할 수 있어야 합니다.** 
**관심사를 적절히 분리해 관리**한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있습니다.

### 횡단 관심사(Cross-Cutting Concerns)

많은 시스템에서 로깅, 보안, 트랜잭션 관리 등의 비즈니스 기능이 아닌 '부가 기능'은 여러 모듈에 걸쳐 적용되어야 합니다. 
이러한 기능을 '횡단 관심사'라고 부릅니다.

전통적인 객체지향 프로그래밍에서는 이러한 관심사를 깔끔하게 분리하기 어렵습니다. 
예를 들어, 모든 메서드에 로깅을 추가하려면:

```java
public void transferMoney(Account from, Account to, double amount) {
    logger.info("Money transfer started");
    try {
        from.withdraw(amount);
        to.deposit(amount);
        logger.info("Money transfer completed");
    } catch (Exception e) {
        logger.error("Money transfer failed", e);
        throw e;
    }
}
```

이 코드에서 로깅 로직이 비즈니스 로직과 섞여 있어 코드가 복잡해집니다.

### AOP(Aspect-Oriented Programming)

이 문제를 해결하기 위해 AOP가 도입되었습니다. 
AOP는 횡단 관심사를 별도의 '관점(Aspect)'으로 분리하여 필요한 시점에 코드에 '위빙(Weaving)'하는 방식입니다.

> 위빙은 기존 코드에 '부가 기능(코드)'를 덧붙이는 걸 의미합니다.

스프링 AOP를 사용한 예:

```java
@Aspect
public class LoggingAspect {
    @Before("execution(* com.example.banking.*.transfer*(..))")
    public void logBeforeTransfer(JoinPoint joinPoint) {
        logger.info("Transfer operation started: " + joinPoint.getSignature().getName());
    }
    
    @AfterReturning("execution(* com.example.banking.*.transfer*(..))")
    public void logAfterTransfer(JoinPoint joinPoint) {
        logger.info("Transfer operation completed: " + joinPoint.getSignature().getName());
    }
    
    @AfterThrowing(pointcut = "execution(* com.example.banking.*.transfer*(..))", throwing = "ex")
    public void logTransferError(JoinPoint joinPoint, Exception ex) {
        logger.error("Transfer operation failed: " + ex.getMessage());
    }
}

// 비즈니스 로직 클래스
public class BankService {
    public void transferMoney(Account from, Account to, double amount) {
        // 순수한 비즈니스 로직만 포함
        from.withdraw(amount);
        to.deposit(amount);
    }
}
```

이렇게 하면 로깅 코드가 비즈니스 로직과 완전히 분리되어, 코드가 더 깨끗해지고 유지보수하기 쉬워집니다.

## 테스트 주도 시스템 아키텍처

> 애플리케이션 도메인 논리를 POJO로 작성할 수 있다면, 즉 코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능해진다. 

TDD(Test-Driven Development)의 원칙을 시스템 수준으로 확장하는 개념입니다. 깨끗한 테스트 코드를 작성하면서 시스템을 점진적으로 발전시키는 방식으로, 이를 통해 결합도가 낮고 응집도가 높은 모듈화된 시스템을 구축할 수 있습니다.

## 명백한 가치가 있을 때 표준을 현명하게 사용하라

표준(프레임워크, 패턴 등)은 많은 이점을 제공하지만, 맹목적으로 모든 상황에 적용해서는 안 된다고 강조합니다.
표준을 채택할 때는 실제 가치를 제공하는지 신중하게 평가해야 합니다.

## 시스템 도메인 특화 언어(DSL)

도메인 특화 언어(DSL)는 특정 비즈니스 도메인의 문제를 해결하기 위해 만들어진 특수 언어입니다. 내부 DSL(기존 언어를 확장)과 외부 DSL(완전히 새로운 언어)이 있으며, 이를 통해 코드를 도메인 전문가가 이해하기 쉽게 만들 수 있습니다.

내부 DSL의 예

```java
// 일반적인 코드
Order order = new Order();
order.setCustomer("John Doe");
order.addItem(new Item("Book", 29.99));
order.addItem(new Item("Pen", 5.99));
order.setShippingAddress(new Address("123 Main St", "Anytown", "CA", "12345"));
order.setPaymentMethod(new CreditCard("1234-5678-9012-3456", "12/25"));

// DSL을 사용한 코드
Order order = new OrderBuilder()
    .forCustomer("John Doe")
    .withItem("Book", 29.99)
    .withItem("Pen", 5.99)
    .shippingTo("123 Main St", "Anytown", "CA", "12345")
    .paidWith(CreditCard.number("1234-5678-9012-3456").expiresOn("12/25"))
    .build();
```

DSL을 사용하면 코드가 더 선언적이고 도메인 중심적이 되어, 비즈니스 로직을 더 명확하게 표현할 수 있습니다.

## 결론 

깨끗한 시스템 설계의 핵심 원칙들을 정리하면:

1. **관심사 분리**: 시스템 구축과 사용을 분리하고, 각 모듈이 단일 책임을 갖도록 설계하세요.
2. **의존성 관리**: 객체 간의 의존성을 명시적으로 관리하고, 의존성 주입을 활용하세요.
3. **점진적 개선**: 시스템을 한 번에 완벽하게 설계하려 하지 말고, 테스트와 리팩토링을 통해 점진적으로 개선하세요.
4. **횡단 관심사 분리**: AOP와 같은 기술을 활용하여 로깅, 보안 등의 횡단 관심사를 분리하세요.
5. **도메인 중심 설계**: DSL을 활용하여 코드를 도메인 전문가가 이해할 수 있는 수준으로 추상화하세요.

이러한 원칙들을 따르면 시스템이 더 유연하고, 테스트 가능하며, 유지보수하기 쉬워집니다. 또한 시스템이 성장함에 따라 새로운 요구사항에 더 쉽게 적응할 수 있게 됩니다.

깨끗한 코드는 단순히 메서드나 클래스 수준에서 끝나는 것이 아니라, **시스템 전체의 설계와 아키텍처에까지 적용되어야 합니다.** 