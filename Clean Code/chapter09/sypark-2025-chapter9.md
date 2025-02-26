# 9장 단위 테스트

TDD(Test-Driven Development) 단위 테스트는 작성하는 것만이 아닌 깨끗한 테스트 코드가 중요하다.

## TDD 법칙 세 가지
1. 실패하는 단위 테스트를 작성하기 전까지 실제 코드를 작성하지 않음.
2. 컴파일은 가능하지만 실행이 실패하는 수준의 테스트 코드 작성.
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드 작성.
하지만 방대한 테스트 코드는 심각한 관리 문제를 유발한다.

## 깨끗한 테스트 코드 유지하기
- 테스트 코드는 실제 코드 못지않게 중요하다.
- 유지보수성을 고려하여 설계해야 한다.
- 테스트 코드를 깨끗하게 작성하지 않으면 점점 유지보수가 어려워지고, 결국 안하느니만도 못하다.

## 깨끗한 테스트 코드 === 가독성
잡다하고 세세한 코드를 없애서 진짜 필요한 자료 유형과 함수만 사용한다.

### 개선 전 코드
```java
public void testGetPageHieratchyAsXml() throws Exception {
  crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response = (SimpleResponse) responder.makeResponse(
      new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
}
```

### 개선 후 코드
```java
public void testGetPageHierarchyAsXml() throws Exception {
  makePages("PageOne", "PageOne.ChildOne", "PageTwo");
  submitRequest("root", "type:pages");
  assertResponseIsXML();
  assertResponseContains("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}
```
- 중복된 코드들을 제거시킨다.
- BUILD-OPERATE-CHECK 패턴
  * 테스트 자료 생성
  * 테스트 자료 조작
  * 조작 결과 확인

## 도메인에 특화된 테스트 언어
- 도메인 특화 테스트 언어(DSL) 활용하여 간결하고 표현력이 풍부한 코드로 가독성을 향상한다.

## 이중 표준
메모리나 CPU 효율과 관련 있는 경우엔 실제 환경과 테스트 환경은 요구사항이 다르기 때문에
실제 환경에서는 절대로 안 되지만 테스트 환경에서는 전혀 문제없는 방식을 써도 무방하다.

## 테스트 당 assert 하나, 개념 하나
- 테스트 함수는 하나의 "개념"만 검증해야 한다.
- 불필요한 assert를 제거하여 테스트 코드의 목적을 명확하게 한다.
- TEMPLATE METHOD 패턴

## F.I.R.S.T
1. **Fast**: 테스트는 "빠르게" 실행되어야 한다.
2. **Independent**: 서로 "독립적으로" 실행 가능해야 한다.
3. **Repeatable**: 어떤 환경에서도 "반복가능하게" 실행되어야 함.
4. **Self-Validating**: "자가검증하는" bool 값으로 성공/실패가 명확해야 함.
5. **Timely**: "적시에" 작성되어야 함 (코드 구현 직전).