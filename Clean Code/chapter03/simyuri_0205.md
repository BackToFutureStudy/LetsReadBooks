## Chapter 3. 함수
#### 함수를 잘 만드는 법
~~~java
public static String renderPageWithSetupsAndTeardowns (
  PageData pageData, boolean isSuite
)throws Exception {
  boolean isTestPage = pageData.hasAttribute("Test");
  if(isTestPage) {
    WikiPage testPage = pageData.getWikiPage();
    StringBuffer newPageContent = new StringBuffer();
    includeSetupPages(testPage, newPageContent, isSuite);
    newPageContent.append(pageData.getContent());
    includeTeardownPages(testPage, newPageContent, isSuite);
    pageData.setContent(newPageContent.toString());
  }
  return pageContent.getHtml();
}
~~~

- 작게 만들어라

   위의 코드를 이렇게 작게도 만들 수 있다.

   ```java
   public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception {
     if(isTestPage(pageData))
       includeSetupAndTeardownPages(pageData, isSuite);
     return pageData.getHtml();
   }
   ```

   블록과 들여쓰기

   if문 / else문 / while 문 등에 들어가는 블록은 한 줄이어야 한다.

   -> 중첩 구조가 생길만큼 함수거 커져서는 안된다. 

   - 한 가지만 해라

   _함수는 한 가지를 해야 한다. 그 한 가지를 잘 해야 한다. 그 한 가지만을 해야 한다._

   1. 페이지가 테스트 페이지인지 판단한다.

   2. 그렇다면 설정 페이지와 해제 페이지를 넣는다.

   3. 페이지를 HTML로 렌더링 한다.

   함수는 간단한 TO문단으로 기술할 수 있다.
   
   <small> LOGO 언어에서 사용하는 키워드 'TO'는 루비나 파이썬에서 사용하는 'def'와 똑같다. LOGO에서 모든 함수는 키워드 'TO'로 시작한다. 이는 함수를 설계하는 방식에 흥미로운 영향을 미쳤다.</small>

   _함수 내 섹션_

   - 함수당 추상화 수준은 하나로!
   함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.

   _위에서 아래로 코드 일기: 내려가기 규칙_

   **Switch문**
   
    각 switch문을 저차원 클래스에 숨기고 절대로 반복하지 않는 방법은 있다. 다형성을 이용한다.
   
   ~~~java
    public Money calculatePay(Employee e) throws InvalidEmployeeType {
      switch(e.type) {
        case COMMISSIONED:
          return calculateCommisionedPay(e);
        case HOURLY: 
          return calculateHourlyPag(e);
        case SALARIED:
          return calculateSalariedPay(e);
        default: 
          throw new InvalidEmployeeType(e.type);
      }
    }
   ~~~

- 서술적인 이름을 사용하라!

   길고 서술적인 이름이 짧고 어려운 이름보다 좋다. 서술적인 주석보다 좋다.

   함수 이름을 정할 때는 여러 단어가 쉽게 읽히는 명명법을 사용한다.

   __함수 인수__

   함수에서 이상적이 인수 갯수는 0개(무항)이다.

   그 다음운 1개(단항), 다음은 2개 (2항)이다. 

   _많이 쓰는 단항 형식_

   함수에 인수 1개를 넘기는 이유

   1. 인수에 질문을 던지는 경우 (ex.`boolean fileExists("MyFile")`)
   2. 인수가 뭔가로 변환해 결과를 반환하는 경우(ex. `inputStream fileOpen("MyFile")`)\

      => String형의 파일 이름을 InputStream으로 변환한다.
   
   다소 드물게 사용하지만 그래도 아주 유용한 단항 함수 형식이 이벤트

   이벤트 함수는 입력 인수만 있고 출력 인수는 없다.(`passwordAttemptFailedNtimes(int attempts)`)

   위에서 설명한 경우가 아니라면 단항 함수는 가급적이면 피한다.

   __플래그 인수__

   함수로 boolean 값을 넘기는 관례는 정말 끔찍하다.

   __이항 함수__

   인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다. 기능하면 단항 함수로 바꾸도록 애써야 한다.

   __삼항 함수__

   삼항 함수를 만들 때에는 신중하게 고려하라.

   _인수 객체_

   인수가 2-3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 가능성을 짚어 본다.

   `Circle makeCircle(double x, double y, double radius);`

   `Circle makeCircle(Point center, double radius);`

   객체를 생성해 인수를 줄이는 방법이 눈속임이라 여겨질 지 모르지만 그렇지 않다.

   _인수 목록_

   때로는 인수 개수가 가변젹인 함수도 필요하다.

   `String.format("%s worked %.2f hours.", name, hours);`

   가변 인수 전부를 동등하게 취급하면 List형 인수 하나로 취급할 수 있다.

   _동사와 키워드_

   함수의 의도나 인수의 순서와 의도를 제대로 표현하려면 좋은 함수 이름이 필수다. 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다. (ex. `write(name)`). 좀 더 나은 이름은 `writeField(name)`

   마지막 예제는 함수 이름에 **키워드**를 추가하는 형식이다. 함수 이름에 인수 이름을 넣는다.(ex. `assertExpectedEqualsActual(expected, actual)`) 이러면 인수 순서를 기억할 필요가 없어진다.

- 부수 효과를 일으키지 마라!

   부수 효과는 거짓말이다. 

   ~~~java
    public class UserValidator{
      private Cryptographer cryptographer;

      public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);
        if(user != User.NULL) {
          String codedPhrase = user.getPhraseEncodedByPassword();
          String phrase = cryptographer.decrypt(codePhrase, password);
          if("Valid Password".equals(phrase)) {
            Session.initialize();
            return true;
          }
        }
        return false;
      }
    }
    ~~~

    여기서 함수가 일으키는 부수 효과는 `Session.initialize()` 호출

    이름만 봐서는 세션을 초기화한다는 사실이 드러나지 않는다. 그래서 함수 이름만 보고 함수를 호출하는 사용자는 사용자를 인증하면서 기존 세션 정보를 지워버릴 위험에 처한다.

    _출력 인수_

    일반적으로 우리는 인수를 함수 **입력**으로 해석한다. 객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다. 출력 인수로 사용하라고 설계한 변수가 바로 this이기 때문이다. 

- 명령과 조회를 분리하라!

   함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다. 둘 다 하면 안 된다. 긱체 상태를 변경하거나 객체 정보를 반환하거나 둘 중 하나다. 명령과 조회를 분리한다.

    ~~~java
    if(attributeExists("username")) {
      setAttribute("username", "unclebob");
    }
    ~~~

- 오류 코드보다 예외를 사용하라!

   _Try/Catch블록 뽑아내기_
   
   try/catch블록은 별도 함수로 뽑아내는 편이 좋다.

   _오류 처리도 한 가지 작업이다._

   함수는 '한 가지' 작업만 해야 한다. 오류 처리도 '한 가지'작업에 속한다.

   _Error.java 의존성 자석_

   오류 코드를 반환한다는 이야기는, 클래스든 열거형 변수든, 어디선가 오류 코드를 정의한다는 뜻이다.

- 반복하지 마라!

   중복은 소프트웨어에서 모든 악의 근원이다.

- 구조적 프로그래밍

   함수는 return 문이 하나여아 한다. 루프 안에서 break나 continue를 사용해선 안되며 goto는 **절대로** 안된다. 함수를 작게 만든다면 간혹 return, break, continue를 여러 차례 사용해도 괜찮다. goto문은 큰 함수에서만 의미가 있으므로, 작은 함수에서는 피해야만 한다.

- 함수를 어떻게 짜죠?

   소프트웨어를 짜는 행위는 여느 글짓기와 비슷하다. 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. 메서드를 줄이고 순서를 바꾼다. 때로는 전체 클래스를 쪼개기도 한다. 이 와중에도 코드는 항상 단위 테스트를 통과한다.

- 결론

   대가 프로그래머는 시스템을 (구현할)프로그램이 아니라 (풀어갈)이야기로 여긴다. 