### 언제, 어떤 기능을 적용할까?

1. **람다와 메서드 참조 (Java 8)**
   - **적용대상**: 컬렉션 순회나 간단한 콜백이 있는 코드  
   - **적용전**  
     ```java
     button.addActionListener(new ActionListener() {
         public void actionPerformed(ActionEvent e) {
             doSomething();
         }
     });
     ```
   - **적용후**  
     ```java
     button.addActionListener(e -> doSomething());
     // 혹은 메서드 참조
     button.addActionListener(this::doSomething);
     ```
   - **팁**: 한눈에 동작이 보이고, 익명 클래스의 보일러플레이트를 제거할 때!

2. **스트림 API (Java 8)**
   - **적용대상**: `List`, `Set`, `Map` 등 컬렉션을 필터링·매핑·집계할 때  
   - **적용전**  
     ```java
     List<String> caps = new ArrayList<>();
     for (String s : list) {
         if (s.length() > 3) {
             caps.add(s.toUpperCase());
         }
     }
     ```
   - **적용후**  
     ```java
     List<String> caps = list.stream()
                             .filter(s -> s.length() > 3)
                             .map(String::toUpperCase)
                             .collect(Collectors.toList());
     ```
   - **팁**: 복잡한 루프 로직이 메서드 체이닝으로 깔끔해집니다.

3. **디폴트 메서드 (Java 8)**
   - **적용대상**: 기존 인터페이스에 기능 추가가 필요할 때  
   - **주의**: 너무 남발하면 ‘왜 인터페이스에 구현이…?’ 싶은 상황이 생길 수 있어요.  
   - **팁**: 구현 클래스를 일일이 모두 수정하기 귀찮을 때, 인터페이스에 `default`를 붙여보세요.

4. **모듈 시스템 (Java 9+)**
   - **적용대상**: 프로젝트 규모가 크고, 의존성·가시성을 명확히 관리하고 싶을 때  
   - **팁**: 작은 라이브러리나 샘플 프로젝트에는 오히려 복잡도를 높일 수 있으니 신중히!

5. **Optional (Java 8)**
   - **적용대상**: 반환 값이 `null`일 수도 있는 메서드  
   - **팁**: `if (obj != null)` → `Optional.ofNullable(obj).ifPresent(…)` 으로 가독성 UP.

6. **패턴 매칭·레코드·그 외 (Java 14+)**
   - **적용대상**: DTO나 단순 값 객체를 정의할 때 (`record`), `instanceof` 뒤 캐스팅이 반복될 때  
   - **팁**: 필요할 때 도입하고, 아직도 IDE나 빌드 환경이 안 따라온다면 한걸음 천천히!

---

**정리하자면**  
- **“모든 코드를 한 방에 바꾸자!”** 보다는  
- **“여기서 람다 쓰면 확실히 가독성이 좋아지네”** 하는 포인트부터  
- 한 단계씩 리팩터링 하는 게 안정적이고 현실적입니다.  

필요한 곳에, 필요한 만큼만! 😊
