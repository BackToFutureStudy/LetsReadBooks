# 7장 오류 처리
'오류 처리'는 중요하지만, 그 '오류 처리 코드'로 인해서 다른 코드와의 가독성이 해쳐지면 안 됨.

### 오류 코드보다 예외를 사용하라
- 예외를 사용하면 오류 처리 코드와 비즈니스 로직이 분리되어 코드가 깨끗해짐.

AS-IS 오류 코드 반환 :
```java
public class DeviceController {
  ...
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    if (handle != DeviceHandle.INVALID) {
      retrieveDeviceRecord(handle);
      if (record.getStatus() != DEVICE_SUSPENDED) {
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
      } else {
        logger.log("Device suspended. Unable to shut down");
      }
    } else {
      logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
}
```

TO-BE 예외 사용:

```java
public class DeviceController {
  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }

  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);
    pauseDevice(handle);
    clearDeviceWorkQueue(handle);
    closeDevice(handle);
  }
}
```

### Try-Catch-Finally 문부터 작성하라
- 프로그램 상태를 일관적으로 유지해서 무슨 일이 생기든 예상되는 상태를 정의하기 쉬워지도록,
예외 발생 가능성이 있는 코드는 Try-Catch-Finally 문으로 감싸도록 한다.

- 예제 코드:

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
  try {
    FileInputStream stream = new FileInputStream(sectionName);
    stream.close();
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```

### 미확인unchecked 예외를 사용하라
- 확인된 예외는 OCP Open Closed Principle를 위반한다. 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다. 모듈과 관련된 코드가 전혀 바뀌지 않았더라도 (선언부가 바뀌었으므로) 모듈을 다시 빌드한 다음 배포해야 한다는 뜻. > 코드 수정 비용이 크다.

- 최상위 함수에서 단계가 내려갈수록 호출하는 함수 수는 늘어나면
1. catch 블록에서 새로운 예외 처리
2. 최상위 선언부에 throw 절을 추가해야한다.
throws 경로에 위치하는 모든 함수가 최하위 함수에서 던지는 예외를 알아야 하므로 캡슐화가 깨진다.

- C#, C++, 파이썬, 루비 등은 확인된 예외를 지원하지 않는다.

### 예외에 의미를 제공하라
- 오류가 발생한 원인과 위치를 찾기 쉽게 전후 상황 정보를 메시지에 담아서 예외와 함께 던진다.
- 실패 연산, 실패 유형을 로깅 기능을 사용하여 넘겨 기록하도록 한다.

### 호출자를 고려한 예외 클래스 설계
- 외부 라이브러리 예외를 직접 하드코딩하지 말고 Wrapper Class를 만들어서 대신 감싸면,
외부 라이브러리와의 의존성도 크게 줄어서 다른 라이브러리로 갈아타도 유지보수 비용이 적다.
- 예제 코드:

```java
public class LocalPort {
  private ACMEPort innerPort;
  public LocalPort(int portNumber) { innerPort = new ACMEPort(portNumber); }
  public void open() {
    try { innerPort.open(); }
    catch (DeviceResponseException | ATM1212UnlockedException | GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
}
```

### 정상 흐름을 정의하라
클라 코드가 예외적인 상황을 처리할 필요가 없게 하기 위해서,
클래스를 만들거나 객체를 조작해 특수 사례를 처리(캡슐화)하는 방식을 "특수 사례 패턴 SPECIAL CASE PATTERN"이라고 한다. 

### null을 반환하지 마라
- null을 반환하면 코드가 복잡해지고 NullPointerException 위험 증가.
- 빈 컬렉션을 반환하는 것이 더 나음.

AS-IS
```java
public void registerItem(Item item) { 
  if (item != null) { 
    ItemRegistry registry = peristentStore.getItemRegistry();
      if (registry != null) {
        ...
      }
  }
}
```
null 확인도 빠졌고 null 확인이 너무 많다. 이와 같은 경우에 '특수 사례 객체'로 해결한다.


TO-BE
```java
public List<Employee> getEmployees() {
  return Collections.emptyList();
}
```

### null을 전달하지 마라
메서드로 null을 반환하지도 말고, 
메서드의 인자로 null을 전달하지도 말라.
인자가 null이 되어버리면 이전 코드에 문제가 있다는 의미이고
적절히 처리하는 방법이 없기 때문에 애초에 null을 넘기지 않도록 한다.
