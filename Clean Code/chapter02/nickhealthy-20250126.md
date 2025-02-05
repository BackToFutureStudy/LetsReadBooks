# 2. 의미 있는 이름

## 의도를 분명히 밝혀라

* 변수나 함수 그리고 클래스 이름은 의도가 분명히 드러나 있어야 한다.
* 주석이 필요하다면 의도를 분명히 드러내지 못했을 가능성이 크다.

```java
// 나쁜 예
public List<int[]> getThem() {
  List<int[]> list1 = new ArrayList<>();
  for (int[] x: theList)
    if (x[0] == 4)
      list1.add(x);
 
  return list1;
}

// 좋은 예
public List<int[]> getFlaggedCells() {
  List<int[]> flaggedCells = new ArrayList<>();
  for (int[] cell: gameBoard)
    if (cell.isFlagged())
      flaggedCells.add(cell);
 
  return flaggedCells;
}
```



## 그릇된 정보를 피하라

*  프로그래머는 코드에 그릇된 단서를 남겨서는 안된다. 변수 이름과 목적에 맞게 이름을 명명해야 한다.
  * 예를 들어, 여러 그룹을 묶을 때 `accountList` 명명하고 실제로 `List` 자료형이 아니라면 그릇된 정보를 제공하는 셈이다.
  * 따라서 실제 `List`가 아니라면 `accountGroup` 같은 이름이 적절하다.
* 유사한 개념은 유사한 표기법을 사용한다. 이것도 정보다. 일관성이 떨어지는 표기법은 그릇된 정보다.



## 의미 있게 구분하라

* 컴파일러나 인터프리터만 통과하려는 생각으로 코드를 구현하는 프로그래머는 스스로 문제를 일으킨다.
* 연속적인 숫자를 덧붙인 이름은 그릇된 정보를 제공하는 것도 아니고, 아무런 정보를 제공하지 못한다.
  * 예를 들어, 다음과 같은 코드이다.
  * 함수 인수 이름으로 `source`와 `destination`을 사용한다면 코드를 읽기 훨씬 수월하다. 

```java
public static void copyChars(char a1[], char a2[]) {
  for (int i = 0; i < a1.length; i++) {
    a2[i] = a1[i];
  }
}
```

* **불용어(noise word)**를 추가하는 방식 역시 아무런 정보도 제공하지 못한다.

  * 예를 들어, `Product` 클래스가 있다고 했을 때 `ProductInfo` 혹은 `ProductData`라 부른다면 개념을 구분하지 않은 채 이름만 달리한 경우다.
  * Info나 Data는 a, an, the와 마찬가지로 의미가 불분명한 불용어다.

* 불용어는 중복이다.

  * 다음과 같은 메서드에서 어떤 메서드가 적절한지 찾기 힘들다.

  * ```java
    getActiveAccount();
    getActiveAccounts();
    getActiveAccountInfo();
    ```



## 발음하기 쉬운 이름을 사용하라

* 발음하기 어려운 이름은 협업에 불편을 초래하고 의사소통을 방해한다.





