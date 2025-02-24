# Chapter 7. 오류 처리



## 오류 코드보다 예외를 사용하라

❌ **잘못된 방법)** - 오류 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 방법

```java
public class DeviceController {

	...

	DeviceHandle handle = getHandle(DEV1);
	if (handle != DeviceHandle.INVALID) {
		retrieveDeviceRecord(handle);
		if (record.getStatus() != DEVICE_SUSPENDED) {
			closeDevice(handle);
		} else {
			logger.log("Device suspended. Unable to shut down");
		}
	} else {
		logger.log("Invalid handle");
	}

	...
}
```

-> 호출자 코드가 복잡해 진다. 함수를 호출한 즉시 오류를 확인해야 하기 때문이다. 예외를 던지는 것이 낫다.



✅ **옳은 방법)** - 예외 사용

```java
public class DeviceController {

	...

	public void sendShutDown() {
		try {
			tryToShutDown();
		}
		catch (DeviceShutDownError e) {
			logger.log(e);
		}
	}

	private void tryToShutDown() {
		DeviceHandle handle = getHandle(DEV1);
		DeviceRecord record = retrieveDeviceRecord(handle);

		pauseDevice(handle);
		clearDeviceWorkQueue(handle);
		closeDevice(handle);
	}

	private DeviceHandle getHandle(DeviceId id) {
		...
		throw new DeviceShutDownError("Invalid handle for: " + id.toString());
		...
	}
	
	...

}
```

보기 좋아졌을 뿐 아니라, 코드 품질도 나아졌다.
디바이스를 종료하는 알고리즘과 오류를 처리하는 알고리즘을 분리했기 때문이다. **각 개념을 독립적으로 살펴보고 이해**할 수 있다.



## Try-Catch-Finally 문부터 작성하라

try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다. 이런 측면에서 볼 때 try 블록은 트랜잭션과 비슷하다. **예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하는 편이 낫다.**



❌ **잘못된 방법)** - 파일이 없으면 예외를 던지는지 알아보는 단위 테스트

```java
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
	sectionStore.retrieveSection("invalid - file");
}

// 단위 테스트에 맞춰 구현한 코드
public List<RecordedGrip> retrieveSection(String sectionName) {
  return new ArrayList<RecordedGrip>(); // 실제로 구현할 때까지 비어 있는 더미를 반환
}

```

코드가 예외를 던지지 않으므로 단위 테스트는 실패하게 된다.



✅ **옳은 방법)** - 단위 테스트에 맞춰 구현한 코드

이번에는 잘못된 파일 접근을 시도하게 구현을 변경해보자. 아래 코드는 예외를 던진다.

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
  try{
    FileInputStream stream = new FileInputStream(sectionName);
    stream.close();
  } catch (FileNotFoundException e) {
    throw new StorageException("retrieval error", e);
  }
  return new ArrayList<RecordedGrip>();
}
```

코드가 예외를 던지므로 이제는 테스트가 성공한다. 이 시점에서 리팩터링이 가능하다.
catch 블록에서 예외 유형을 좁혀 실제로 `FileInputStream` 생성자가 던지는 `FileNotFoundException`을 잡아낸다.
**이제 try-catch 구조로 범위를 정의했으므로 나머지 논리를 추가하면 된다.** 나머지 논리는 `FileInputStream`을 생성하는 코드와 `close` 호출문 사이에 넣으며 오류나 예외가 전혀 발생하지 않는다고 가정한다.



따라서 다음과 같은 프로세스로 진행하면 코드를 짜기가 수월해진다.

1. **강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장한다.**
   * 그러면 자연스럽게 try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

2. **try-catch 구조로 범위를 정의했으므로 TDD를 사용해 필요한 나머지 논리를 추가한다.**



## 미확인(unchecked) 예외를 사용하라

예전부터 확인된 예외와 미확인 예외로 논쟁이 있었지만, 현재는 **'미확인 예외를 사용하는 것이 낫다'**는 것이 일반적이다.
또한 여러 프로그래밍 언어에서 확인된 예외를 지원하지 않아도 안정적인 소프트웨어를 제작하는 요소로 무리가 없다는 것은 분명해졌다.



### 확인된 예외의 비용

1. 확인된 예외는 OCP(Open Closed Principle) 원칙을 위반한다.
   * 하위 단계에서 코드를 변경하면 상위 단계 메서드를 전부 고쳐야 한다.
   * catch 블록에서 새로운 예외를 처리하거나, 선언부에 throws 절을 추가해야 한다.
2. 캡슐화가 깨진다.
   * `throws` 경로에 위치하는 모든 함수가 최하위 함수에서 던지는 예외를 알아야 하므로 캡슐화가 깨진다.
3. 꼭 확인해야 할 중요한 처리나 라이브러리는 예외를 명시적으로 표시하기도 한다.



## 예외에 의미를 제공하라

자바는 모든 예외에 호출 스택을 제공하지만 이것만으론 실패한 코드를 파악하기 힘들다.
따라서 다음과 같은 가이드를 따른다.

* 예외를 던질 땐 오류 메시지에 정보(실패한 연산 이름과 실패 유형도 언급)를 담아 예외와 함께 던진다.
* 로깅 기능을 사용해 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨준다.



## 호출자를 고려해 예외 클래스를 정의하라

오류를 분류하는 방법은 다양하다. 오류가 발생한 위치나 유형으로도 분류가 가능하다. 예를 들어 디바이스, 네트워크, 프로그래밍 오류 등으로 분류할 수 있다. **하지만 프로그래머에게 가장 중요한 관심사는 오류를 잡아내는 방법이 되어야 한다.**



❌ **잘못된 방법)** - 오류를 형편없이 분류한 사례

외부 라이브러리를 호출하는 try-catch-finally 문을 포함한 코드로, 라이브러리가 던질 예외를 모두 잡아낸다.

```java
ACMEPort port = new ACMEPort(12);

try{
	  port.open();
} catch (DeviceResponseException e) {
		reportPortError(e);
} catch (ATM1212UnlockedException e) {
		reportPortError(e);
} catch (GMXError e) {
		reportPortError(e);
} finally {
	...
}
```



✅ **옳은 방법)** - 호출하는 라이브러리의 API를 감싸면서 예외 유형을 하나 반환한다.

```java
LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}

// 외부 라이브러리(ACMEPort)를 감싼 Wrapper 클래스
public class LocalPort {
  private ACMEPort innerPort;
  
  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }
  
  public void open() {
    try{
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e); // port 디바이스 실패를 표현하는 예외 유형 하나를 정의
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e); // port 디바이스 실패를 표현하는 예외 유형 하나를 정의
    } catch (GMXError e) {
      throw new PortDeviceFailure(e); // port 디바이스 실패를 표현하는 예외 유형 하나를 정의
    }
  }
  
  ...
}
```

**외부 API를 사용할 땐 Wrapper 클래스로 감싸는 방법이 최선이다.** 다음과 같은 이점이 있다.

* 외부 API를 감싸면 외부 라이브러리와 프로그램 사이에서 의존성이 크게 줄어든다.
  * 나중에 다른 라이브러리로 갈아타도 비용이 적다.
* 외부 API를 호출하는 대신 테스트 코드를 넣어주는 방법으로 프로그램을 테스트하기도 쉬워진다.
* 외부 API를 설계한 방식에 발목 잡히지 않는다. 위의 예제처럼 외부 라이브러리 실패 시 프로그램이 사용하기 편리한 API를 직접 정의할 수 있다. (`throw new PortDeviceFailure(e);`)



## 정상 흐름을 정의하라

**특수 상황을 처리하는 예외 때문에 코드가 복잡하고 어려워질 때가 있다.**
이런 경우 특수 사례 패턴(Special Case Pattern)을 통해 처리하면 좋다.
**이 패턴의 주로 사용되는 상황은 특정 조건에서 다른 흐름을 유지해야 할 때이다.** 예를 들어, 함수나 메서드가 특정 입력에 대해 '정상적인' 동작을 하지 않고, 대신 다른 행동을 취해야 할 경우가 있다. 이런 특수한 조건을 일반적인 로직에서 분리하여 명확하게 처리하는 것이 중요하다.



✅ **값이 없을 때 처리)**

예를 들어, 함수가 리스트나 배열을 처리하는데, 리스트가 비어있는 경우를 다뤄야 한다면, 이를 "특수 사례"로 보고 별도의 코드로 처리할 수 있다.

```java
import java.util.List;

public class ItemFinder {

    // 리스트가 비어있는지 확인하는 메서드
    public static boolean isEmpty(List<?> items) {
        return items == null || items.isEmpty();
    }

    public static String findItem(List<String> items, String target) {
        if (isEmpty(items)) {  // 리스트가 비어있는지 확인
            return null;
        }
        for (String item : items) {
            if (item.equals(target)) {
                return item;
            }
        }
        return null;
    }

    public static void main(String[] args) {
        List<String> items = List.of("apple", "banana", "cherry");
        String target = "banana";
        
        String result = findItem(items, target);
        System.out.println(result);  // 출력: banana
    }
}

```



✅ **파일이 없는 경우를 처리하는 함수 분리)**

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

public class FileReader {

    // 파일이 없을 때 처리하는 메서드
    public static void handleFileNotFound(String fileName) {
        System.out.println("파일 " + fileName + "을 찾을 수 없습니다.");
    }

    public static String readFile(String fileName) {
        try {
            Scanner scanner = new Scanner(new File(fileName));
            StringBuilder content = new StringBuilder();
            while (scanner.hasNextLine()) {
                content.append(scanner.nextLine()).append("\n");
            }
            scanner.close();
            return content.toString();
        } catch (FileNotFoundException e) { // 파일이 없는 경우
            handleFileNotFound(fileName);
            return null;
        }
    }

    public static void main(String[] args) {
        String fileName = "nonexistent_file.txt";
        
        String fileContent = readFile(fileName);
        if (fileContent != null) {
            System.out.println(fileContent);
        }
    }
}

```



✅ **예외적인 조건을 처리하는 경우**

```java
// 원본 코드
try {
	MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
	m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
	m_total += getMealPerDiem();
}

// 특수 사례 패턴을 적용한 코드
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

// 특수 사례 패턴
public class PerDiemMealExpenses implements MealExpenses {
	public int getTotal() {
		// 기본값으로 일일 기본 식비를 반환한다.
	}
}
```



이러한 처리를 통해 **코드의 가독성을 높이고, 예외적인 상황에서의 로직을 분리하여 다른 코드 흐름이 깨지지 않도록 하기 위함**이다. 



## null을 반환하지 마라

`null`을 반환하고 이를 `if(object != null)`으로 확인하는 방식은 **나쁘다**.

* 메서드에서 `null`을 반환하고 싶은 유혹이 든다면 그 <u>대신 예외를 던지거나 특수 사례 객체(ex. `Collections.emptyList()`)를 반환</u>한다.
* 사용하려는 외부 API가 null을 반환한다면 **Wrapper를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려**한다.

> 많은 경우 특수 사례 객체가 손쉬운 해결책이다.



❌ **잘못된 방법)** - null을 반환하는 메서드

```java
List<Employee> employees = getEmployees();
if (employees != null) {
	for(Employee e : employees) {
		totalPay += e.getPay();
	}
}
```



✅ **옳은 방법)** - 특수 사례 객체를 반환

```java
List<Employee> employees = getEmployees();
for(Employee e : employees) {
	totalPay += e.getPay();
}

public List<Employee> getEmployees() {
	if ( ...직원이 없다면... ) {
		return Collections.emptyList();
	}
}
```



코드도 깔끔해지며 NPE이 발생할 가능성도 줄어든다.



## null을 전달하지 마라

**null을 반환하는 방식보다 메서드로 null을 전달하는 방식은 더 나쁘다.** 
정상적인 인수로 null을 기대하는 API가 아니람녀 메서드로 null을 전달하는 코드는 최대한 피한다.

* 예외를 던지거나 `assert` 문을 사용할 수는 있지만 완벽한 해결책은 아니다.
* 애초에 `null`을 전달하는 경우는 금지하는 것이 바람직하다.



## 결론

깨끗한 코드는 읽기도 좋아야 하지만 **안전성도 높아야 한다.** 이 둘은 상충하는 목표가 아니다.

* **오류 처리를 프로그램 논리와 분리**해야 튼튼하고 깨끗한 코드를 작성할 수 있다.
* 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성도 크게 높아진다.