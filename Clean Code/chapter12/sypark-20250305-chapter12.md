## 12장 창발성(創發性)

### 창발적 설계로 깔끔한 코드를 구현하자
- 켄트 벡이 제시한 **단순한 설계 규칙** 네 가지:
  1. 모든 테스트를 실행할 것.
  2. 중복을 없앨 것.
  3. 프로그래머 의도를 표현할 것.
  4. 클래스와 메서드 수를 최소로 줄일 것.

### 단순한 설계 규칙 1: 모든 테스트를 실행하라
테스트 가능한 시스템은 설계 품질이 자동으로 향상된다.

### 단순한 설계 규칙 2~4: 리팩터링
테스트 주도 개발(TDD)을 활용하면 SRP, DIP 등의 원칙을 자연스럽게 적용할 수 있다.

### 중복을 없애라
- 코드 중복은 유지보수성과 확장성을 저해하는 주요 원인이기 때문에 꼭 정리헤야한다.
- **TEMPLATE METHOD 패턴** 등을 활용하여 공통 로직을 정리해야한다.

#### 예제 코드: 중복 제거 (TEMPLATE METHOD 패턴)
```java
abstract class VacationPolicy {
    public void accrueVacation() {
        calculateBaseVacationHours();
        alterForLegalMinimums();
        applyToPayroll();
    }

    private void calculateBaseVacationHours() { /* ... */ }
    abstract protected void alterForLegalMinimums();
    private void applyToPayroll() { /* ... */ }
}

class USVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        // 미국 최소 법정 일수 적용
    }
}

class EUVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        // 유럽연합 최소 법정 일수 적용
    }
}
```

### 표현하라
- 코드는 명확해야 하며, 개발자의 의도를 분명하게 보여줘야한다.
- 좋은 이름을 사용하고, 작은 함수와 클래스를 유지하며, 표준 패턴을 활용해야한다.

### 클래스와 메서드 수를 최소로 줄여라
- 불필요한 클래스를 늘리는 것보다 적절한 추상화와 모듈화를 유지하는 것이 중요하다.
- **SRP를 유지하면서도 과도한 모듈화를 피해야한다.**

### 결론
- 창발적 설계를 위해서는 **테스트를 철저히 수행하고, 중복을 제거하며, 가독성을 높이고, 적절한 모듈화를 유지**해야한다.
- TDD와 리팩토링을 적극적으로 활용하면 보다 깨끗한 설계를 유지할 수 있다.

