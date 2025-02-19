## Chapter 7. 오류 처리

   오류 처리는 프로그램에 반드시 필요한 요소 중 하나이다. 상당수 코드 기반은 전적으로 오류 처리 코드에 좌우된다.

   **오류 코드보다 예외를 사용하라**

   오류가 발생하면 예외를 던지는 편이 낫다. 그러면 호출자 코드가 더 깔끔해진다.

   ~~~java
      public class DeviceController {
         ...
         public void sendShutDown() {
            try {
               tryToSuhtDown();
            } catch (DeviceShutDownError e) {
               loger.log(e);
            }
         }

         private void tryToShutDown() throws DeviceShutDownError {
            DeviceHandle handle = getHandle(DEV1);
            DeviceRecord record = retrieveDeviceRecord(handle);

            pauseDevice(handle);
            clearDeviceWorkQueue(handle);
            closeDevice(handle);
         }

         private DeviceHandle getHandle(DeviceID id) {
            ...
            throw new DeviceShutDownError("Invalid handle for: " + id.toString());
         }
         ...
      }
   ~~~

   **Try-Catch-Finally 문부터 작성하라**

   어떤 면에서 try 블록은 트랜잭션과 비슷하다. try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다. 그러므로 예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하는 편이 낫다. 그러면 try 블록에서 무슨 일이 생기든지 호출자가 기대하는 상태를 정의하기 쉬워진다.

   try-catch 구조로 범위를 정의하고 TDD(Test-Driven Development, 테스트 주도 개발)를 사용해 필요한 나머지 논리를 추가한다. 

   먼저 강제로 예외를 일으키는 테스트 케이스를 작서안 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다. 그러면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

   **미확인(unchecked) 예외를 사용하라**

   지금은 안정적인 소프트웨어를 제작하는 요소로 확인된 예외가 반드시 필요하지도 않다는 사실이 분명해졌다. 그러므로 확인된 오류가 치르는 비용에 상응하는 이익을 제공하는지 (철저히) 따져봐야 한다. 
   
   확인된 예외는 OCP(Open Closed Principle)를 위반한다. 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다는 말이다. 모듈과 관련된 코드가 전혀 바뀌지 않았더라도 (선언부가 바뀌었으므로) 모듈을 다시 빌드한 다음 배포해야 한다는 말이다.

   때로는 확인된 예외도 유용하다. 아주 중요한 라이브러리를 작성한다면 모든 예외를 잡아야 한다. 하지만 일반적인 애플리케이션은 의존성이라는 비용이 이익보다 크다.

   **예외에 의미를 제공하라**

   자바는 모든 예외에 호출 스텍을 제공한다. 하지만 실패한 코드의 의도를 파악하려면 호출 스택만으로 부족하다. 오류메세지에 정보를 담아 예외와 함께 던진다. 실패한 연산 이름과 실패 유형도 언급한다. 애플리케이션이 로깅 기능을 사용한다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨준다.

   **호출자를 고려해 예외 클래스를 정의하라**

   애플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 오류를 잡아내는 방법이 되어야 한다. 외부 API를 사용할 때는 감싸기 기법이 최선이다. 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다. 나중에 다른 라이브러리로 갈아타도 비용이 적다. 또한 감싸기 클래스에서 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방법으로 프로그램을 테스트하기도 쉬워진다.

   **정상 흐름을 정의하라**

   특수 사례 패턴(SPECIAL CASE PATTERN): 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식

   이를 이용하면 클래스나 객체가 예외적인 상황을 캡슐화해서 처리하므로 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다. 

   **null을 반환하지 마라**

   null을 반환하는 코드는 일거리를 늘릴 뿐만 아니라 호출자에게 문제를 떠넘긴다. 빈 리스트를 반환한다면 코드가 훨씬 깔끔해진다. 

   자바에는 ```Collections.emptyList()```가 있어 미리 정의된 읽기 전용 리스트를 반환할 수 있다. 이렇게 코드를 변경하면 코드도 깔끔해질 뿐더러 ```NullPointerException```이 발생할 가능성도 줄어든다.
   
   **null을 전달하지 마라**

   메서드로 null을 전달하는 방식은 더 나쁘다. 정상적인 인수로 null을 기대하는 API가 아니라면 메서드로 null을 전달하는 코드는 최대한 피한다. 

   assert문을 사용해도 된다.

   ~~~java
      public class MetricsCalculator {
         public double xProjection(Point p1, Point p2) {
            assert p1 != null : "p1 should not be null";
            assert p2 != null : "p2 should not be null";
            return (p2.x - p1.x) * 1.5;
         }
      }
   ~~~

   대다수 프로그래밍 언어는 호출자가 실수로 넘기는 null을 적절히 처리하는 방법이 없다. 그렇다면 애초에 null을 넘기지 못하도록 금지하는 정책이 합리적이다. 즉, 인수로 null이 넘어오면 코드에 문제가 있다는 말이다. 

   **결론**

   깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다. 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.