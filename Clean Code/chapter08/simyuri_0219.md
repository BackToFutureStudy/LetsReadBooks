## Chapter 8. 경계

   시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다. 어떤식으로든 이 외부 코드를 우리 코드에 깔끔하게 통합해야만 한다. 

   **외부 코드 사용하기**

   패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애쓴다. 반면, 사용자는 자신의 요구에 집중하는 인터페이스를 바란다. 

   Map과 같은 경계 인터페이스를 이용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의한다. Map 인스턴스를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않는다.

   **경계 살피고 익히기**

   외부코드를 사용하면 적은 시간에 더 많은 기능을 출시하기 쉬워진다.

   외부 코드를 익히기는 어렵다. 외부 코들르 통합하기도 어렵다. 두 가지를 동시에 하기는 두 배나 어렵다. 곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성ㅇ해 외부 코드를 익히면 어떨까> 짐 뉴커크는 이를 **학습 테스트**라 부른다.

   학습 테스트는 프로그램에서 사용하려는 방식대로 외부 API를 호출한다. 통제된 환경에서 API를 제대로 이해하는지를 확인하는 셈이다. 학습 테스트는 API를 사용하려는 목적에 초점을 맞춘다.

   **log4j 익히기**

   로깅 기능을 직접 구현하는 대신 아파치의 log4j 패키지를 사용하려 한다고 가정하자. 

   ~~~java
      @Test
      public class LogTest {
         private Logger logger;

         @Before 
         public void initialize() {
            logger = Logger.getLogger("logger");
            logger.removeAllAppenders();
            logger.getRootLogger().removeAllAppenders();
         }

         @Test
         public void basicLogger() {
            BasicConfigurator.configure();
            logger.info("basicLogger");
         }

         @Test
         public void addAppenderWithStream() {
            logger.addAppender(new ConsoleAppender(new PatternLayout("%p %t %m%n"), ConsoleAppender.SYSTEM_OUT));
            logger.info("addAppenderWithStream");
         }

         @Test
         public void addAppenderWithoutStream() {
            logger.addAppender(new ConsoleAppender(new PatternLayout("%p %t %m%n")));
            logger.info("addAppenderWithoutStream");
         }
      }
   ~~~

   이제 모든 지식을 독자적인 로거 클래스로 캡슐화한다. 그러면 나머지 프로그램은 log4j 경계 인터페이스를 몰라도 된다.

   **학습 테스트는 공짜 이상이다**

   학습 테스트에 드는 비용은 없다. 오히려 필요한 지식만 확보하는 손쉬운 방법이다. 학습 테스트는 이해도를 높여주는 정확한 실험이다.

   학습 테스트를 이용한 학습이 필요하든 그렇지 않든, 실제 코드와 동일한 방식으로 인터페이스를 사용하는 테스트 케이스가 필요하다. 이런 경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬워진다. 그렇지 않다면 낡은 버전을 필요 이상으로 오랫동안 사용하려는 유혹에 빠지기 쉽다.

   **아직 존재하지 않는 코드를 사용하기**

   경계와 관련해 또 다른 유형은 아는 코드와 모르는 코드를 분리하는 경계다. 때로는 우리 지식이 경계를 너머 미치지 못하는 코드 영역도 있다. 

   우리가 바라는 인터페이스를 구현하면 우리가 인터페이스를 전적으로 통제한다는 장점이 생긴다. 또한 코드 가독성도 높아지고 코드 의도도 분명해진다. 

   **꺠끗한 경계**

   소프트웨어 설계가 우수하다면 변경하는데 많은 투자와 재작업이 필요하지 않다. 엄청난 시간과 노력과 재작업을 요구하지 않는다. 통제하지 못하는 코드를 사용할 때는 너무 많은 투자를 하거나 향후 변경 비용이 지나치게 커지지 않도록 각별히 주의해야 한다. 

   외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하자. Map에서 봤듯이, 새로운 클래스로 경계를 감싸거나 아니면 ADAPTER 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환핮. 

   어느 방법이든 코드 가독성이 높아지며, 경계 인터페이스를 사용하는 일관성도 높아지며, 외부 패키지가 변했을 때 변경할 코드도 줄어든다.
   

   > ADAPTER 패턴 : 클래스를 어댑터로서 사용되는 구조 패턴. 호환성이 없는 인터페이스 때문에 함께 동작할 수 없는 클래스들을 함께 작동해주도록 변환 역할을 해주는 행동 패턴이라고 보면 된다.  예를들어 기존에 있는 시스템에 새로운 써드파티 라이브러리를 추가하고 싶거나, Legacy 인터페이스를 새로운 인터페이스로 교체하는 경우에 어댑터 패턴을 사용하면 코드의 재사용성을 높일 수 있다. 
   >
   >출처: https://inpa.tistory.com/entry/GOF-💠-어댑터Adaptor-패턴-제대로-배워보자 [Inpa Dev 👨‍💻:티스토리]