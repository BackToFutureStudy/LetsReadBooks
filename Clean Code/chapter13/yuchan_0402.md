# Clean Code Chapter 13. 동시성

## 📌 동시성이란 무엇인가?

동시성이란 간단히 말해 **멀티 스레드를 사용하는 프로그래밍 기법**입니다. 하지만 이것이 왜 중요할까요?

동시성은 본질적으로 **결합을 없애는 작업**입니다. 즉, '무엇'과 '언제'를 분리하는 전략이죠. 단일 스레드 프로그램에서는 무엇을, 언제 실행할지 명확하게 예측할 수 있습니다. 이는 디버깅에 유리하지만, 작업 효율성 측면에서는 제한이 있습니다.

### 💡 실생활 예시

카카오 알림톡을 100명의 사용자에게 보내는 상황을 생각해봅시다. 한 명에게 알림을 보내는 데 1초가 걸린다고 가정해보겠습니다:

- **단일 스레드**: 한 명씩 차례대로 보내면 100초 소요
- **멀티 스레드**(3개 스레드 사용): 동시에 3명에게 보낼 수 있어 약 33초 소요

이런 차이가 동시성을 사용하는 주된 이유입니다.

## 🤔 동시성에 관한 미신과 오해

동시성이 항상 좋아 보이지만, 실제로는 오해와 미신이 많습니다.

### ❌ 미신 1: "동시성은 항상 성능을 높여준다"

실제로는 **특정 상황에서만** 성능 향상을 기대할 수 있습니다:
- 대기 시간이 긴 작업에서 여러 스레드가 프로세스를 공유할 때
- 독립적인 계산이 충분히 많을 때

또한 중요한 점은 **수신측에서 동시성을 처리할 수 있는지**도 고려해야 합니다.

### ❌ 미신 2: "동시성을 구현해도 설계는 변하지 않는다"

단일 스레드와 멀티 스레드 시스템은 설계 자체가 다릅니다. '무엇'과 '언제'를 분리하면 필연적으로 시스템 구조가 달라집니다.

### ❌ 미신 3: "스프링 컨테이너를 사용하면 동시성을 이해할 필요가 없다"

프레임워크가 많은 것을 처리해주지만, 근본적인 이해 없이는 문제 발생 시 대응이 어렵습니다.

### ⚠️ 사실인 경우도 있습니다

1. **멀티 스레드 시스템은 부하가 많을 것이다** - 맞습니다. 동시에 많은 작업을 처리하면 당연히 부하가 증가합니다.
2. **동시성은 복잡하다** - 맞습니다. 고려해야 할 사항이 정말 많습니다.
3. **동시성 버그는 재현하기 어렵다** - 단순 로직 문제라면 확인하기 쉽지만, 특정 타이밍에만 발생하는 버그는 재현이 매우 어렵습니다.
4. **단일에서 멀티 스레드로 설계 변경 시 근본적인 전략 변화 필요** - 경우에 따라 맞을 수 있습니다.

## ⚠️ 동시성 구현이 어려운 이유

다음 코드를 예로 들어보겠습니다:

```java
public class Sequence {
    private int seq = 7;

    public int getNextSeq() {
        return ++seq;
    }
}