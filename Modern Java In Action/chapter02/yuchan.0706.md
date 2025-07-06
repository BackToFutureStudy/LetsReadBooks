## Chapter 2 – 동작 파라미터화(Behavior Parameterization)

### 언제, 어떤 방식으로 적용할까?

1. **하드코딩된 메서드 복사 (구식 방식)**
   - **적용대상**: 단일 조건(예: 검은색 차량)만 필요할 때  
   - **적용전**  
     ```java
     public static List<Car> filterBlackCar(List<Car> cars) {
         List<Car> result = new ArrayList<>();
         for (Car c : cars) {
             if (c.getColor() == Color.BLACK) {
                 result.add(c);
             }
         }
         return result;
     }
     ```
   - **적용후**  
     > 해당 없음 (구식 방식)  
   - **팁**: 유지보수·확장성이 전혀 없습니다. 바로 리팩터링 시작!

2. **속성 파라미터화 (Color, Price 등)**
   - **적용대상**: 특정 속성 하나만 자주 변경될 때  
   - **적용전**  
     ```java
     public static List<Car> filterCarByColor(
         List<Car> cars, Color color) { … }
     public static List<Car> filterCarByPrice(
         List<Car> cars, int maxPrice) { … }
     ```
   - **적용후**  
     > 통합 리팩터링 필요 (다음 단계 참고)  
   - **팁**: 단일 속성 변경까진 OK, 여러 속성 조합 요구엔 비효율적.

3. **전략(Strategy) 패턴 + Predicate 인터페이스**
   - **적용대상**: 다양한 조건을 클래스 단위로 분리하고 싶을 때  
   - **적용전**  
     ```java
     // filterByColor, filterByPrice 등 메서드가 잔뜩…
     ```
   - **적용후**  
     ```java
     public interface CarPredicate {
         boolean test(Car c);
     }
     public static List<Car> filterCars(
         List<Car> cars, CarPredicate p) {
         List<Car> result = new ArrayList<>();
         for (Car c : cars) {
             if (p.test(c)) result.add(c);
         }
         return result;
     }
     // 구현체 예시
     class PricePredicate implements CarPredicate {
         public boolean test(Car c) { return c.getPrice() <= 7000; }
     }
     ```
   - **팁**: 새로운 조건은 클래스 하나만 추가하면 끝!

4. **익명 클래스(Anonymous Class) 간소화**
   - **적용대상**: 한두 군데, 잠깐 테스트용 로직이 필요할 때  
   - **적용전**  
     ```java
     filterCars(carList, new PricePredicate());
     ```
   - **적용후**  
     ```java
     filterCars(carList, new CarPredicate() {
         public boolean test(Car c) {
             return c.getColor() == Color.RED;
         }
     });
     ```
   - **팁**: 별도 파일 없이 즉석 전략 주입. 장기간 사용 시 람다로 전환을 추천!

5. **람다 표현식 (Java 8)**
   - **적용대상**: 익명 클래스도 과하다고 느껴질 때  
   - **적용전**  
     ```java
     filterCars(carList, new CarPredicate() {
         public boolean test(Car c) { return c.isBlue(); }
     });
     ```
   - **적용후**  
     ```java
     filterCars(carList, c -> c.getColor() == Color.BLUE);
     ```
   - **팁**: 한눈에 동작이 파악되며, 보일러플레이트 대폭 감소!

6. **제네릭 Predicate + 범용 필터**
   - **적용대상**: 타입에 구애받지 않고 재활용 API를 만들고 싶을 때  
   - **적용전**  
     ```java
     // CarPredicate 전용 filterCars만 존재…
     ```
   - **적용후**  
     ```java
     public interface Predicate<T> {
         boolean test(T t);
     }
     public static <T> List<T> filter(
         List<T> list, Predicate<T> p) {
         List<T> result = new ArrayList<>();
         for (T e : list) {
             if (p.test(e)) result.add(e);
         }
         return result;
     }
     // 숫자 필터링
     List<Integer> evens = filter(numbers, i -> i % 2 == 0);
     ```
   - **팁**: 객체 종류가 늘어나도 한 가지 메서드로 OK!

---

**정리**  
- **“플래그 메서드”** → 즉시 버려라!  
- **인터페이스 기반 전략** → 구조적이고 명확  
- **익명 클래스 → 람다 → 제네릭** 단계별 도입으로  
- **유연성↑, 중복↓, 유지보수성↑** 를 동시에 잡자! 🚀  