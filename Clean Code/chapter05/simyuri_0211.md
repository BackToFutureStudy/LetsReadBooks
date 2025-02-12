## Chapter 5. 형식 맞추기

__형식을 맞추는 목적__

   코드 형식은 의사소통의 일환이다. 오랜 시간이 지나 원래 코드의 흔적을 더 이상 찾아보기 어려울 정도로 코드가 바뀌어도 맨 처음 잡아놓은 구현 스타일과 가독성 수준은 유지보수 용이성과 확장성에 계속 영향을 미친다. 원래 코드는 사라질지라도 개발자의 스타일과 규율은 사라지지 않는다.

__적절한 행 길이를 유지하라__

   500줄을 넘지 않고 대부분 200줄 정도인 파일로도 커다란 시스템을 구축할 수 있다. 일반적으로 큰 파일보다 작은 파일이 이해하기 쉽다.

   _신문 기사처럼 작성하라_

   이름은 간단하면서도 설명이 가능하게 짓는다. 

   이름만 보고도 올바른 모듈을 살펴보고 있는지 아닌지를 판단할 정도로 짓는다.

   소스 파일 첫 부분은 고차원 개념과 알고리즘을 설명한다. 

   아래로 내려갈수록 의도를 세세하게 묘사한다. 

   마지막에는 가장 저차원 함수와 세부 내역이 나온다. 

   _개념은 빈 행으로 분리하라_

   각 행은 수식이나 절을 나타내고, 일련의 행 묶음은 완결된 생각 하나를 표현한다. 생각 사이에는 빈 행을 넣어 분리해야 마땅하다. 빈 행은 새로운 개념을 시작한다는 시각적 단서다.

   _세로 밀집도_

   줄바꿈이 개념을 분리한다면 세로 밀집도는 연관성을 의미한다. 
   
   서로 밀접한 코드 행은 세로로 가까이 놓여야 한다는 뜻이다.

   _수직 거리_

   서로 밀접한 개념은 세로로 가까이 두어야 한다. 두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않는다.

__변수 선언__
   
   변수는 사용하는 위치에 최대한 가까이 선언한다. 루프를 제어하는 변수는 흔히 루프 문 내부에 선언한다.

   ~~~java
    public int countTestCases() {
        int count = 0;
        for(Test each: tests) {
            count += each.countTestCases();
            return count;
        }
    }
   ~~~

- 인스턴스 변수

   인스턴스 변수는 클래스 맨 처음에 선언한다. 변수 간에 세로로 거리를 두지 않는다. 

   일반적으로 C++에서는 모든 인스턴스 변수를 클래스의 마지막에 선언한다는 소위 **가위 규칙**을 적용한다.

   자바에서는 보통 클래스 맨 처음에 인스턴스 변수를 선언한다. 

- 종속 함수

   한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다. 또한 가능하다면 호출하는 함수를 호출되는 함수보다 먼저 배치한다. 

- 개념적 유사성

   어떤 코드는 서로 끌어당긴다. 개념적인 친화도가 높기 때문인데, 친화도가 높을수록 코드를 가까이 배치한다.

   _세로 순서_

   일반적으로 함수 호출 종속성은 아래 방향으로 유지한다. 호출되는 함수를 호출하는 함수보다 나중에 배치한다.

   중요한 개념을 가장 먼저 표현한다.

**가로 형식 맞추기**

   프로그래머는 명백히 짧은 행을 선호한다. -> 짧은 행이 바람직하다.

   _가로 공백과 밀집도_

   가로로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현한다.

   할당 연산자를 강조하기 위해 앞뒤에 공백을 준다.

   함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않는다. -> 함수와 인수는 서로 밀접하기 때문이다. 

   함수를 호출하는 코드에서 괄호 안 인수는 공백으로 분리했다. 쉽표를 강조해 인수가 별개라는 사실을 보여준다.

   연산자 우선순위를 강조하기 위해서도 공백을 사용한다.

   ```java
    public class Quadratic {
        public static double root1(double a, double b, double c) {
            double determinant = determinant(a, b, c);
            return (-b + Math.sqrt(determinant)) / (2*a);
        }

        public static double root2(int a, int b, int c) {
            double determinant = determinant(a, b, c);
            return (-b - Math.sqrt(determinant)) / (2*a);
        }
        private static double determinant(double a, double b, double c) {
            return b*b - 4*a*c;
        }
    }
   ```

   승수 사이는 공백이 없다.

   _가로 정렬_

   선언문과 할당문을 별도로 정렬하지 않는다. 선언부가 길다면 클래스를 쪼개야 한다.

   _들여쓰기_

   소스 파일은 윤곽도(outline)과 비슷하다. 범위로 이뤄진 계층을 표현하기 위해 코드를 들여쓴다.

- 들여쓰기 무시하기

   때로는 간단한 if문, 짧은 while문, 짧은 함수에서 들여쓰기 규칙을 무시하고픈 유혹이 생긴다. 한 행에 범위를 뭉뚱그린 코드를 피한다.

   _가짜 범위_

   때로는 빈 while문이나 for문을 접한다. 피하지 못할 때는 빈 블록을 올바로 들여쓰고 괄호로 감싼다. 세미콜론은 새 행에 제대로 들여써서 넣어준다.

   __팀 규칙__

   팀은 한 가지 규칙에 합의해야 한다. 그리고 모든 팀워는 그 규칙을 따라야 한다. 그래야 소프트웨어가 일관적인 스타일을 보인다.

   좋은 소프트웨어는 읽기 쉬운 문서로 이뤄진다.

   __밥 아저씨의 형식 규칙__

   
   ```java
   public class CodeAnalyzer implements JavaFileAnalysis {
      private int lineCount;
      private int maxLineWidth;
      private int widestLineNumber;
      private LineWidthHistogram lineWidthHistogram;
      private int totalChars;

      public CodeAnalyzer() {
         lineWidthHistogram = new LineWidthHistogram();
      }

      public static List<File> findJavaFiles(File parentDirectory) {
         List<File> files = new ArrayList<File>();
         return files;
      }

      public static void fildJavaFiles(File parentDirectory, List<File> files) {
         for (File file : parentDirectory.listFiles()) {
            if (file.getName().endsWith(".java"))
               files.add(file);
            else if (file.isDirectory())
               findJavaFiles(file, files);
         }
      }

      public void analyzeFile(File javaFile) throws Exception {
         BufferedReader br = new BufferedReader(new FileReader(javaFile));
         String line;
         while((line = br.readLine()) != null) 
            measureLine(line);
      }

      private void measureLine(String line) {
         lineCount++;
         int lineSize = line.length();
         totalChars += lineSize;
         lineWidthHistogram.addLine(lineSize, lineCount);
         recordWidestLine(lineSize);
      }

      private void recordWidestLine(int lineSize) {
         if (lineSize > maxLineWidth) {
            maxLineWidth = lineSize;
            widestLineNumber = lineCount;
         }
      }

      public int getLineCount() {
         return lineCount;
      }

      public int getMaxLineWidth() {
         return maxLineWidth;
      }

      public int getWidestLineNumber() {
         return widestLineNumber;
      }

      public LineWidthHistogram getLineWidthHistogram() {
         return lineWidthHistogram;
      }

      public double getMeanLineWidth() {
         return (double)totalChars/lineCount;
      }

      public int getMedianLineWidth() {
         Integer[] sortedWidths = getSortedWidths();
         int cumulativeLineCount = 0;
         for (int width : sortedWidths) {
            cumulativeLineCount += lineCountForWidth(width);
            if (cumulativeLineCount > lineCount/2) 
               return width;
         }
         throw new Error("Cannot get here");
      }

      private int lineCountForWidth(int width) {
         return lineWidthHistogram.getLinesForWidth(width).size();
      }

      private Integer[] getSortedWidths() {
         Set<Integer> widths = lineWidthHistogram.getWidths();
         Integer[] sortedWidths = (widths.toArray(new Integer[0]));
         Arrays.sort(sortedWidths);
         return sortedWidths;
      }
   }
   ```