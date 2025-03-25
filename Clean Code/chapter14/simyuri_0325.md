## Chapter 14. 점진적인 개선

이 장은 점진적인 개선을 보여주는 사례 연구다.

프로그램을 짜다 보면 종종 명령행 인수의 구문을 분석할 필요가 생긴다. 편리한 유틸리티가 없다면 main함수로 넘어오는 문자열 배열을 직접 분석하게 된다. 편리한 유틸리티가 없다면 직접 짜겠다고 결심한다. 새로 짤 유틸리티를 Args라 하자.

생성자에 (입력으로 들어온) 인수 문자열과 형식 문자열을 넘겨 Args 인스턴스를 생성한 후 Args 인스턴스에다 인수 값을 정의한다.

```java
public static void main(String[] args) {
 try {
     Args arg = new Args("l,p#,d*", args);
     boolean logging = arg.getBoolean('l');
     int port = arg.getInt('p');
     String directory = arg.getString('d');
     executeApplication(logging, port, directory);
 } catch (ArgsException e) {
     System.out.printf("Argument error: %s\n", e.errorMessage());
 }
}
```

매개변수 2개로 Args 클래스의 인스턴스를 만든다.

**Args 구현**

스타일과 구조에 신경을 써 Args 클래스를 만든다.

```java
package com.objectmentor.utilities.args;

import static com.objectmentor.utilities.ArgsException.ErrorCode.*;
import java.util.*;

public class Args {
    private Map<Character, ArgumentMarshaler> marshalers;
    private Set<Character> argsFound;
    private ListIterator<String> currentArgument;

    public Args(String schema, String[] args) throws ArgsException {
        marshalers = new HashMap<Character, ArgumentMarshaler>();
        argsFound = new HashSet<Character>();

        parseSchema(schema);
        pasreArgumentStrings(Arrays.asList(args));
    }

    private void parseSchema(String schema) throws ArgsException {
        for(String element : schema.split(","))
            if(element.length() > 0)
                parseSchemaElement(element.trim());
    }

    private void parseSchemaElement(String element) throws ArgsException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (elementTail.length() == 0)
            marshalers.put(elementId, new booleanArgumentMarshaler());
        else if (elementTail.equals("*"))
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if(elementTail.equals("#"))
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        else if(elementTail.equals("##"))
            marshalers.put(elementId, new DoubleArgumentMarshaler());
        else if(elementTail.equals("[*]"))
            marshalers.put(elementId, new StringArrayArgumentMarshaler());
        else
            throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
    }

    private void parseArgumentStrings(List<String> argsList) throws ArgsException {
        for (currentArgument = argsList.listIterator(); currentArgument.hasNext();) {
            String argsString = currentArgument.next();
            if(argString.startsWith("-")) {
                parseArgumentCharacters(argString.substring(1));
            } else {
                currentArgument.previous();
                break;
            }
        }
    }

    private void parseArgumentCharacters(String argChars) throws ArgsException {
        for (int i = 0; i < argChars.length(); i++)
            parseArgumentCharacter(argChars.charAt(i));
    }

    private void parseArgumentCharacter(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        if (m == null) {
            throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null);
        } else {
            argsFound.add(argChar);
            try {
                m.set(currentArgument);
            } catch (ArgsException e) {
                e.setErrorArgumentId(argChar);
                throw e;
            }
        }
    }

    public boolean has(char arg) {
        return argsFound.contains(arg);
    }

    public int nextArgument() {
        return currentArgument.nextIndex();
    }

    public boolean getBoolean(char arg) {
        return BooleanArgumentMarshaler.getValue(marshalers.get(arg));
    }

    public String getString(char arg) {
        return StringArgumentMarshaler.getValue(marshalers.get(arg));
    }

    public int getInt(char arg) {
        return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
    }

    public double getDouble(char arg) {
        return DoubleargumentMarshaler.getValue(marshalers.get(arg));
    }

    public String[] getStringArray(char arg) {
        return StringArrayArgumentMarshaler.getValue(marshalers.get(arg));
    }
}
```

여기저기 뒤적일 필요 없이 위에서 아래로 코드가 읽힌다. 아래는 ArgumentMarshaler 인터페이스와 파생 클래스다.

_ArgumentMarshaler.java_

```java
public interface ArgumentMarshaler {
    void set(Iterator<String> currentArgument) throws ArgsException;
}
```

_BooleanArgumentMarshaler.java_

```java
public class BooleanArgumentMarshaler implements ArgumentMarshaler {
    private boolean booleanValue = false;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        booleanValue = true;
    }

    public static boolean getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof BooleanArgumentMarshaler)
            return ((BooleanArgumentMarshaler) am).booleanValue;
        else
            return false;
    }
}
```

_StringArgumentMarshaler.java_

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class StringArgumentMarshaler implements ArgumentMarshaler {
    private String stringValue = "";

    public void set(Iterator<String> currentArgument) throws ArgsException {
        try {
            stringValue = currentArgument.next();
        } catch (NoSuchElementException e) {
            throw new ArgsExceptoin(MISSING_STRING);
        }
    }

    public static String getValue(ArgumentMarshaler am) {
        if (am != null &&n am instanceof StringArgumentMarshaler)
            return ((StringArgumentMarshaler) am).stringValue;
        else
            return "";
    }
}
```

_IntegerArgumentMarshaler.java_

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

public class IntegerArgumentMarshaler implements ArgumentMarshaler {
    private int intValue = 0;

    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_INTEGER);
        } catch (NumberFormatException e) {
            throw new ArgsException(INVALID_ID_INTEGER, parameter);
        }
    }

    public static int getValue(ArgumentMarshaler am) {
        if(am != null && am instanceof IntegerArgumentMarshaler)
            return ((IntegerArgumentMarshaler) am).intValue;
        else
            return 0;
    }
}
```

나머지 DoubleArgumentMarshaler, StringArrayArgumentMarshaler는 다른 파생 클래스와 똑같은 패턴이므로 코드 생략.

자바는 정적 타입 언어라서 타입 시스템을 만족하려면 많은 단어가 필요하다. 루비, 파이썬, 스몰토크 등과 같은 언어를 사용했다면 프로그램이 훨씬 작아졌을 것이다.

_어떻게 짰느냐고?_

깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다.

_점진적으로 개선한다_

프로그램을 ㅁ아치는 가장 좋은 방법 중 하나는 개선이라는 이름 아래 구조를 크게 뒤집는 행위다. 어떤 프로그램은 그저 그런 '개선'에서 결코 회복되지 못한다. '개선'전과 똑같이 프로그램을 돌리기가 아주 어렵기 때문이다.

그래서 테스트 주도 개발(Test-Driven Development, TDD)라는 기법을 사용한다. TDD는 언제 어느 때라도 시스템을 망가뜨리는 변경을 허용하지 않는다. 변경을 가한 후에도 시스템이 변경 전과 똑같이 돌아가야 한다는 말이다.
