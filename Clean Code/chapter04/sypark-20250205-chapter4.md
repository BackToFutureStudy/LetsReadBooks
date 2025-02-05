# Clean Code 클린 코드 4장 요약: 주석

### 주석의 필요성과 문제점
- **좋은 주석은 거의 필요 없는 주석임.**
- **코드 자체가 설명이 되어야 함.**
- 불필요한 주석은 코드 품질을 떨어뜨림.

### 나쁜 주석의 유형

#### 1. 중복된 주석
- 코드가 수행하는 동작을 그대로 설명하는 주석은 불필요함.

```java
// Bad Example
// 사용자의 나이를 증가시킴
user.age += 1;

// Good Example
user.incrementAge();
```

#### 2. 거짓말하는 주석
- 코드가 변경되었는데 주석을 수정하지 않으면 오히려 혼란을 줌.
- 유지보수가 어렵고 신뢰성이 낮아짐.

```java
// 사용자를 비활성화한다.
user.activate(); // 실제론 활성화하는 코드
```

#### 3. 잡음이 많은 주석
- 불필요한 정보나 의미 없는 설명을 주석으로 다는 것은 피해야 함.

```java
// Bad Example
// 변수 선언
int count = 0;
```

#### 4. 의무적으로 다는 주석 (Javadoc 남용)
- 단순한 Getter/Setter에 Javadoc을 다는 것은 의미 없음.

```java
/**
 * @return 사용자의 이름을 반환한다.
 */
public String getName() {
  return name;
}
```

### 좋은 주석의 예시

#### 1. 법적 주석
- 라이선스 정보가 포함된 주석은 필수적으로 들어가야 함.

```java
// Copyright (C) 2023, Company Name. All rights reserved.
```

#### 2. 의도를 설명하는 주석
- 코드만으로 이해하기 어려운 경우, 의도를 분명하게 설명하는 주석은 유용함.

```java
// 사용자가 처음 로그인하면 기본 설정을 로드해야 함.
if (isFirstLogin(user)) {
  loadDefaultSettings(user);
}
```

#### 3. TODO 주석
- 나중에 수정해야 할 부분을 명확하게 남기는 경우.

```java
// TODO: 성능 개선 필요 - 현재 O(n^2) 복잡도를 가짐.
optimizeSortingAlgorithm();
```
