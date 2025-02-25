## Chapter 10. 클래스

   코드의 표현력과 그 코드로 이루어진 함수에 아무리 신경 쓸지라도 좀 더 차원 높은 단계까지 신경 쓰지 않으면 깨끗한 코드를 얻기는 어렵다. 

   **클래스의 체계**

   클래스를 정의하는 표준 자바 관례에 따르면, 가장 먼저 변수 목록이 나온다. 정적(static) 공개(public) 상수가 있다면 맨 처음에 나온다. 다음으로 정적 비공개(private) 변수가 나오며, 이어서 비공개 인스턴스 변수가 나온다. 공개 변수가 필요한 경우는 거의 없다.

   변수 목록 다음에는 공개 함수가 나온다. 비공개 함수는 자신을 호출하는 공개 함수 직후에 넣는다. 즉, 추상화 단계가 순차적으로 내려간다. 그래서 프로그램은 신문 기사처럼 읽힌다. 

   *캡슐화*

   변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 한다는 법칙도 없다. 때로는 변수나 유틸리티 함수를 protected로 선언해 테스트 코드에 접근을 허용하기도 한다. 캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.

   **클래스는 작아야 한다!**

   클래스는 작아야 한다. 클래스를 설계할 때도, 함수와 마찬가지로 '작게'가 기본 규칙이라는 의미이다. 클래스는 다른 척도를 사용한다. 클래스가 맡은 책임(RDD)을 센다.

   클래스 이름은 해당 클래스 책임을 기술해야 한다. 실제로 작명은 클래스 크기를 줄이는 첫 번째 관문이다. 간결한 이름이 떠오르지 않는다면 필경 클래스 크기가 너무 커서 그렇다. 클래스 이름이 모호하다면 필경 클래스 책임이 너무 많아서다.

   크래스 설명은 만일(if), 그리고(and), -(하)며(or), 하지만(but)을 사용하지 않고서 25단어 내외로 가능해야 한다.

   _단일 책임 원칙_

   단일 책임 원칙(Single Responsibility Principle, SRP)은 클래스나 모듈을 *변경할 이유*가 단 하나뿐이어야 한다는 원칙이다. SRP는 '책임'이라는 개념을 정의하며 적절한 클래스 크기를 제시한다. 클래스는 책임, 즉 변경할 이유가 하냐여야 한다는 의미다.

   책임, 즉 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다.

   SRP는 객체 지향 설계에서 더욱 중요한 개념이다. 또한 이해하고 지키기 수월한 개념이기도 하다.

   규모가 어느 수준에 이르는 시스템은 논리가 많고도 복잡하다. 이런 복잡성을 다루려면 체계적인 정리가 필수다. 그래야 개발자가 무엇이 어디에 있는지 쉽게 찾는다.

   큰 클래스 몇 개가 아니라 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직하다. 작은 클래스는 각자 맡은 책임이 하나며, 변경할 이유가 하념, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.

   *응집도(Cohesion)*

   클래스는 인스턴스 변수 수가 작아야 한다. 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다. 일반적으로 메서드가 변수를 더 많이 사용할수록 메서드와 클래스는 응집도가 더 높다. 모든 인스턴스 변수를 메서드마다 사용하는 클래스는 응집도가 가장 높다.

   응집도가 높다는 말은 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미다. 응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두세 개로 쪼개준다.

   *응집도를 유지하면 작은 클래스 여럿이 나온다*

   큰 함수를 작은 함수 여럿으로 나누기만 해도 클래스 수가 많아진다.

   만약 네 변수를 클래스 인스턴스 변수로 승격한다면 새 함수는 인수가 *필요없다.* 그만큼 함수를 쪼개기 *쉬워진다.* 불행히도 이렇게 하면 클래스가 응집력을 잃는다. 몇몇 함수만 사용하는 인스턴스 변수가 점점 더 늘어나기 때문이다.

   큰 함수를 작은 함수 여럿으로 쪼개다 보면 종종 작은 클래스 여럿으로 쪼갤 기회가 생긴다. 그러면서 프로그램에 점점 더 체계가 잡히고 구조가 투명해진다.

   1. 리팩터링한 프로그램은 좀 더 길고 서술적인 변수 이름을 사용한다.
   2. 리팩터링한 프로그램은 코드에 주석을 추가하는 수단으로 함수 선언과 클래스 선언을 활용한다.
   3. 가독성을 높이고자 공백을 추가하고 형식을 맞추었다.

   가장 먼저, 원래 프로그램의 정확한 동작을 검증하는 테스트 슈트를 작성했다. 그런 다음, 한 번에 하나씩 수 차례에 걸쳐 조금씩 코드를 변경했다. 코드를 변경할 때마다 테스트를 수행해 원래 프로그램과 동일하게 동작하는지 확인했다.

   **변경하기 쉬운 클래스**

   대다수 시스템은 지속적인 변경이 가해진다. 그리고 뭔가 변경할 때마다 시스템이 의도대로 동작하지 않을 위험이 따른다. 깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위함을 낮춘다. 

   ~~~java
    abstract public class Sql {
        public Sql(String table, Column[] columns)
        abstract public  String generate();
    }

    public class CreateSql extends Sql {
        public CreateSql(String table, Column[] columns)
        @Override public String generate()
    }

    public class SelectSql extends Sql {
        public SelectSql(String table, Column[] columns)
        @Override public String generate()
    }

    public class InsertSql extends Sql {
        public InsertSql(String table, Column[] columns, Object[] fields)
        @Override public String generate()
        private String valuesList(Object[] fields, final Column[] columns)
    }

    public class SelectWithCriteriaSql extends Sql {
        public SelectWithCriteriaSql (String table, Column[] columns, Criteria criteria)
        @Override public String generate()
    }

    public class SelectWithMatchSql extends Sql {
        public SelectWithMatchSql (String table, Column[] columns, Column column, String pattern)
        @Override public String generate()
    }

    public class FindByKeySql extends Sql {
        public FindByKeySql(String table, Column[] columns, String keyColumn, String keyValue)
        @Override public String generate()
    }

    public class PreparedInsertSql extends Sql {
        public PreparedInsertSql(String table, Column[] columns)
        @Override public String generate()
        private String placeholderList(Column[] columns)
    }

    public class Where {
        public Where(String criteria)
        public String generate()
    }

    public class ColumnList {
        public ColumnList(Column[] columns)
        public String generate()
    }
   ~~~

   각 클래스는 극도로 단순하다. 코드는 순식간에 이해된다. 함수 하나를 수정했다고 다른 함수가 망가질 위험도 사실상 사라졌다. 테스트 고나점에서 모든 논리를 구석구석 증명하기도 쉬워졌다. 클래스가 서로 분리되었기 때문이다.

   여기다 객체 지향 설계에서 또 다른 핵심 원칙인 OCP(Open-Closed Principle)도 지원한다.
   
   OCP란 클래스는 확장에 개방적이고 수직에 폐쇄적이어야한다는 원칙이다. 우리가 재구성한 Sql 클래스는 파생 클래스를 생성하는 방식으로 새 기능에 개방적인 동시에 다른 클래스를 닫아놓는 방식으로 수정해 폐쇄적이다. 그저 UpdateSql 클래스를 제자리에 끼워 넣으면 끝이다.

   새 기능을 수정하거나 기존 기능을 변경할 때 건드릴 코드가 최소인 시스템 구조가 바람직하다. 이상적인 시스템이라면 새 기능을 추가할 때 시스템을 확장할 뿐 기존 코드를 변경하지 않는다.

   _변경으로부터 격리_

   요구사항은 변하기 마련이다. 객체 지향 프로그래밍 입문에서 우리는 구체적인(concrete) 클래스와 추상(abstract) 클래스가 있다고 배웠다. 구체적인 클래스는 상세한 구현(코드)를 포함하며 추상 클래스는 개념만 포함한다고도 배웠다. 상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험에 빠진다. 그래서 우리는 인터페이스와 추상 클래스를 사용해 구현이 미치는 영향을 격리한다.

   시스템의 결합도를 낮추면 유연성과 재사용성도 더욱 높아진다. 결합도가 낮다는 소리는 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다는 의미다. 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 더 쉬워진다.

   이렇게 결합도를 최소로 줄이면 자연스럽게 또 다른 클래스 설계 원칙인 DIP(Dependency Inversion Principle)를 따르는 클래스가 나온다. 본질적으로 DIP는 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙이다.