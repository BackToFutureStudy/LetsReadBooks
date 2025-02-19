# 8장 경계
패키지, 오픈 소스, 다른 컴포넌트들을 내가 개발하는 코드에 깔끔하게 통합하는 방법

### 외부 코드 사용하기
- 외부 라이브러리는 인터페이스를 통해 감싸서 사용하자.
- `Map<String, Sensor>`과 같이 클래스 밖으로 노출되게 사용하지 말고
`Sensors`와 같은 클래스를 안으로 Map을 숨겨서 사용하도록 한다.
*매번 캡슐화하란 의미가 아닌, Map 인스턴스 자체를 공개 API의 인자나 반환 타입으로 사용하지 말란 의미?

```java
public class Sensors {
  private Map<String, Sensor> sensors = new HashMap<>();
  public Sensor getById(String id) { return sensors.get(id); }
}
```

### 경계 살피고 익히기
- 외부 라이브러리를 바로 실제 코드에 사용하기 전에 
학습 테스트를 작성해서 예상대로 도는지 검증 호환해보며 외부 라이브러리의 기능을 익힐 것.

#### `log4j` 예제:

```java
@Test
public void testLogAddAppender() {
  Logger logger = Logger.getLogger("MyLogger");
  logger.addAppender(new ConsoleAppender(new PatternLayout("%p %t %m%n"), ConsoleAppender.SYSTEM_OUT));
  logger.info("hello");
}
```

### 아직 존재하지 않는 코드 사용하기
- 인터페이스를 미리 정의해 구현을 나중으로 미루며 개발 진행.
- 'API 사용을 캡슐화해 API가 바뀔 때 수정할 코드를 한곳으로 모으는' ADAPTER 패턴 사용:

```java
public interface Transmitter {
  void transmit(Frequency freq, Stream data);
}
```

### 깨끗한 경계
- 외부 패키지를 직접 사용하지 말고 ADAPTER나 Wrapping을 사용.
- 경계 학습 테스트 예시들을 작성해둬서 라이브러리 변경에 대비.
