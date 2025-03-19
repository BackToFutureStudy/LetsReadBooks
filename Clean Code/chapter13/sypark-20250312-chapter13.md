# 13장 동시성

동시성과 깨끗한 코드는 유지하기 아주 어렵다. 
멀티스레드 코드는 부하가 걸리기 전까지는 문제가 드러나지 않으며, 디버깅이 어렵다. 
동시성이 필요한 이유, 동시성의 어려움, 깨끗한 동시성 코드를 작성하는 방법을 다룬다.

## 동시성이 필요한 이유?
- 동시성은 `무엇(What)`과 `언제(When)`를 분리하는 전략이다.
- 싱글 스레드 프로그램은 `무엇`과 `언제`가 밀접하게 연관된다.
- 동시성을 활용하면 시스템의 구조적 개선과 성능 향상 가능하다.
- 다중 사용자 환경에서 응답 시간을 줄이고, 대량의 데이터를 빠르게 처리하는 데 필수적이다.

## (동시성의) 미신과 오해
- **동시성은 항상 성능을 높인다** → 성능을 높일 수도 있지만, 항상 그런 것은 아니다.
- **동시성을 구현해도 설계는 변하지 않는다.** → 단일 스레드 시스템과는 완전히 다른 설계를 필요로 한다.
- **웹 또는 EJB 컨테이너를 사용하면 동시성을 이해할 필요가 없다.** → 실제로는 컨테이너의 동작을 이해하고, 동시 수정, 데드락 문제 등을 고려해야 한다.

## (동시성의) 난관
- 다중 스레드 환경에서 변수를 공유하면 예상치 못한 결과 발생 가능.
- 다음과 같은 코드에서 문제 발생 가능.

```java
public class X {
  private int lastIdUsed;

  public int getNextId() {
    return ++lastIdUsed;
  }
}
```

- 두 개의 스레드가 동시에 `getNextId()`를 호출하면, `lastIdUsed` 값이 올바르게 증가하지 않을 가능성이 있다.
- 해결책: **동기화(synchronized) 또는 원자적 연산(Atomic Variables)을 활용**.

## 동시성 방어 원칙

### 단일 책임 원칙 (SRP, Single Responsibility Principle)
- 동시성과 관련된 코드는 별도로 분리해야 한다.
- 동시성은 자체적인 복잡성을 가지므로, 다른 코드와 혼합하지 말아야 한다.

### 따름 정리 corollary : 자료 범위 제한하기, 자료 사본을 사용하라
- 공유 객체의 범위를 최소화해야 한다.
- 가능한 한 데이터를 불변(immutable)으로 유지한다.
- 필요한 경우 복사본을 사용하여 동기화를 피할 수 있다.

### 따름 정리: 스레드는 가능한 독립적으로 구현하라
- 스레드는 가능한 독립적으로 구현해야 한다.
- 독립적인 단위로 데이터를 분할하고, 가능하면 다른 프로세서에서 동작하도록 설계한다.

## 동시성 패턴과 모델

### 생산자-소비자 패턴
- 하나 이상의 생산자가 데이터를 생성하여 큐에 추가하고, 하나 이상의 소비자가 데이터를 소비하는 구조.
- `BlockingQueue`를 활용하면 안전한 큐 운영 가능.

```java
class Producer implements Runnable {
  private BlockingQueue<Integer> queue;

  public Producer(BlockingQueue<Integer> queue) {
    this.queue = queue;
  }

  @Override
  public void run() {
    try {
      for (int i = 0; i < 10; i++) {
        queue.put(i);
      }
    } catch (InterruptedException e) {
      Thread.currentThread().interrupt();
    }
  }
}
```

### 읽기-쓰기 패턴
- 읽기 스레드와 쓰기 스레드가 동시에 접근할 경우, 성능 저하 또는 기아(starvation) 문제 발생 가능.
- `ReadWriteLock`을 사용하여 읽기와 쓰기를 분리할 수 있음.

```java
private final ReadWriteLock lock = new ReentrantReadWriteLock();

public void read() {
  lock.readLock().lock();
  try {
    // 읽기 작업 수행
  } finally {
    lock.readLock().unlock();
  }
}

public void write() {
  lock.writeLock().lock();
  try {
    // 쓰기 작업 수행
  } finally {
    lock.writeLock().unlock();
  }
}
```

## 동기화 전략

### 동기화 최소화
- `synchronized` 키워드를 남발하면 성능 저하 및 데드락 가능성 증가한다.
- 가능한 한 동기화 블록을 작게 유지해야 한다.

### 적절한 락(lock) 사용
- `ReentrantLock`, `Semaphore`, `CountDownLatch` 등 다양한 락을 적절하게 활용.
- 공유 자원 접근이 필요한 경우, 경쟁 조건을 방지하기 위해 락을 신중하게 설계.

## 다중 스레드 코드 테스트

### 기본 원칙
- 스레드 관련 문제는 재현이 어렵기 때문에 다양한 환경에서 테스트해야 한다.
- 실행 속도를 조절하며 테스트한다.
- `Thread.yield()` 또는 `sleep()`을 이용해 경쟁 상태를 인위적으로 유도한다.

### 흔들기(jiggling) 기법
- 특정 지점에서 `Thread.yield()`를 삽입하여 실행 순서를 의도적으로 변경한다.
- `ThreadJigglePoint` 클래스를 활용하여 코드의 다양한 실행 경로를 테스트 가능하게 한다.

```java
public class ThreadJigglePoint {
  public static void jiggle() {
    if (Math.random() > 0.5) {
      Thread.yield();
    }
  }
}
```

## 결론
- 동시성 코드는 철저한 계획과 검증이 필요.
- 가능한 한 SRP를 지켜 동시성 코드 분리.
- 공유 객체를 최소화하고 불변성을 유지.
- 적절한 락과 동기화 전략을 사용하여 경쟁 조건을 방지.
- 다양한 환경에서 충분한 테스트를 수행.
