# Clean Code 클린 코드 3장 요약: 함수

### 함수의 기본 원칙
- 함수는 **한 가지 작업만 수행해야 함.**
- 함수 이름이 하는 일을 정확하게 나타내야 함.
- 추상화 수준을 일관되게 유지해야 함.
- 함수 길이는 짧을수록 좋음.

### 좋은 함수의 특징

#### 1. 한 가지만 수행할 것
- 함수는 단일 책임 원칙(SRP, Single Responsibility Principle)을 따라야 함.
- 하나의 함수가 여러 가지 작업을 수행하면 유지보수가 어려워짐.

```java
// Bad Example
public void processEmployee(Employee e) {
  validateEmployee(e);
  calculateSalary(e);
  generateReport(e);
}

// Good Example
public void processEmployee(Employee e) {
  validate(e);
  calculate(e);
  report(e);
}
```

#### 2. 의미 있는 이름 사용
- 함수 이름은 **명확한 동사**를 포함해야 함.
- 예측 가능한 동작을 수행해야 함.

```java
// Bad Example
void delete() {}

// Good Example
void deleteUserAccount() {}
```

#### 3. 매개변수는 적게 사용
- 가장 좋은 함수는 매개변수를 **0개 또는 1개**만 가짐.
- 3개 이상이면 데이터 구조로 묶는 것이 좋음.

```java
// Bad Example
void setUser(String name, int age, String address) {}

// Good Example
void setUser(User user) {}
```

#### 4. 부수 효과(Side Effect) 최소화
- 함수는 예상치 못한 상태 변경을 하지 않아야 함.
- 전역 변수 변경을 피하고, 입력을 변경하는 대신 새로운 값을 반환해야 함.

```java
// Bad Example
boolean checkPassword(String user, String pass) {
  if (db.validate(user, pass)) {
    Session.initialize(); // 예상치 못한 부수 효과
    return true;
  }
  return false;
}

// Good Example
boolean checkPassword(String user, String pass) {
  return db.validate(user, pass);
}
```

#### 5. Switch 문은 최대한 피하기
- Switch 문은 다형성을 활용해 대체할 수 있음.

```java
// Bad Example
switch (employee.type) {
  case "FULL_TIME":
    return fullTimePay();
  case "PART_TIME":
    return partTimePay();
  default:
    return defaultPay();
}

// Good Example - 다형성 활용
class Employee {
  abstract double calculatePay();
}

class FullTimeEmployee extends Employee {
  double calculatePay() { return fullTimePay(); }
}

class PartTimeEmployee extends Employee {
  double calculatePay() { return partTimePay(); }
}
```
