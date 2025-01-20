# Clean Code 클린 코드 2장 요약: 의미 있는 이름

### 1. 이름의 중요성
- 이름은 코드 전반에 걸쳐 사용되므로, 좋은 이름이 코드 가독성과 유지보수성을 크게 향상시킴.
- 좋은 이름은 개발자 간의 의사소통을 돕고, 코드의 품질을 높임.
- 이름을 신중히 선택하고, 시간이 지나도 명확성을 유지할 수 있도록 개선할 필요가 있음.

---

### 2. 의도를 분명히 밝혀라
- 이름은 변수, 함수, 클래스의 존재 이유와 기능, 사용 방법을 명확히 드러내야 함.
- 주석 없이도 이름만으로 코드를 이해할 수 있어야 함.
- **예:**  
    ```java
    // 나쁜 예
    int d; // 경과 시간 (단위: 날짜)

    // 좋은 예
    int elapsedTimeInDays;
    int daysSinceModification;
    ```

---

### 3. 그릇된 정보를 피하라
- 변수 이름이 널리 알려진 의미와 다르게 사용되면 혼란을 초래함.
- 예를 들어, `accountList`는 실제로 `List`가 아니라면 부적절. `accountGroup`과 같은 이름이 더 적합함.
- **예:**  
    ```java
    // 나쁜 예
    List<Account> accountList; // 실제로는 HashMap임

    // 좋은 예
    Map<String, Account> accountGroup;
    ```

---

### 4. 의미 있게 구분하라
- 이름은 단순히 서로 다르기만 해서는 안 되며, 각각의 의미를 명확히 전달해야 함.
- 연속된 숫자나 불필요한 불용어(예: `a`, `the`)를 사용하는 것은 피해야 함.
- **예:**  
    ```java
    // 나쁜 예
    public static void copyChars(char a1[], char a2[]) {
        for (int i = 0; i < a1.length; i++) {
            a2[i] = a1[i];
        }
    }

    // 좋은 예
    public static void copyChars(char source[], char destination[]) {
        for (int i = 0; i < source.length; i++) {
            destination[i] = source[i];
        }
    }
    ```

---

### 5. 발음하기 쉬운 이름을 사용하라
- 발음하기 어려운 이름은 의사소통을 방해하고 협업에 불편을 초래함.
- 팀원과의 대화에서 불필요한 혼란을 줄이기 위해, 누구나 쉽게 읽고 발음할 수 있는 이름을 사용.
- **예:**  
    ```java
    // 나쁜 예
    private Date genymdhms; // 생성 날짜 및 시간

    // 좋은 예
    private Date generationTimestamp;
    ```

---

### 6. 검색하기 쉬운 이름을 사용하라
- 한 문자 변수나 매직 넘버는 검색을 어렵게 만듦.
- 이름이 길더라도 검색 가능한 이름이 가독성과 유지보수에 유리함.
- **예:**  
    ```java
    // 나쁜 예
    for (int j = 0; j < 34; j++) {
        s += (t[j] * 4) / 5;
    }

    // 좋은 예
    int realDaysPerIdealDay = 4;
    const int WORK_DAYS_PER_WEEK = 5;
    int sum = 0;
    for (int j = 0; j < NUMBER_OF_TASKS; j++) {
        int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
        int realTaskWeeks = realTaskDays / WORK_DAYS_PER_WEEK;
        sum += realTaskWeeks;
    }
    ```

---

### 7. 인코딩을 피하라
- 변수명에 접두어나 접미어(`m_`, `I`)를 붙이는 방식은 현대 IDE에서 불필요함.
- 헝가리식 표기법 등 불필요한 인코딩은 코드 가독성과 변경을 방해함.
- **예:**  
    ```java
    // 나쁜 예
    private String m_dsc;  // 설명 문자열

    // 좋은 예
    private String description;
    ```

---

### 8. 의미 있는 맥락을 추가하라
- 이름만으로 충분한 정보를 제공하지 못한다면 클래스, 함수, 네임스페이스 등을 사용해 맥락을 추가해야 함.
- 맥락을 제공하지 않으면 변수나 함수가 어디에 속하는지 혼란을 초래함.
- **예:**  
    ```java
    // 나쁜 예
    String state;

    // 좋은 예
    Address address;
    String state = address.getState();
    ```

---

### 9. 불필요한 맥락을 없애라
- 클래스나 변수명에 불필요한 접두어나 불용어를 추가하지 말아야 함.
- 이름은 짧으면서도 명확해야 함.
- **예:**  
    ```java
    // 나쁜 예
    class GSDAccountAddress {
        ...
    }

    // 좋은 예
    class AccountAddress {
        ...
    }
    ```

---

### 10. 마치면서
- 좋은 이름은 코드 가독성을 높이고, 유지보수성을 향상시키는 핵심적인 요소임.
- 이름을 개선하는 노력을 게을리하지 말고, 코드의 품질을 지속적으로 높이는 데 집중해야 함.
