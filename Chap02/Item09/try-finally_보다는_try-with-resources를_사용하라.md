## Item09

"try-finally 보다는 try-with-resources를 사용하라"

## 1. try-finally의 개념과 문제점

### 자원 관리의 필요성

Java에는 InputStream, OutputStream, java.sql.Connection 등 사용 후 **명시적으로 close를 호출해야 하는 자원**들이 있다.

### 단순 close 호출의 문제
   ```java
   public static String inputString() throws IOException {
       BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
       String result = br.readLine();
       br.close();
       return result;
   }
   ```
예외 발생 시 close가 호출되지 않아 자원 누수가 발생할 수 있다.
즉, 쉽게 말해 BufferedReader는 사용 중 IOException이 발생할 수 있는데, 만약 br.readLine() **메서드에서 IOException이 발생하게 되면 메서드가 종료되므로 close가 호출되지 않고 스트림이 메모리에 남아있게** 된다.

### try-finally 사용
   ```java
   public static String inputString() throws IOException {
       BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
       try {
           return br.readLine();
       } finally {
           br.close();
       }
   }
   ```
예외 발생 여부와 관계없이 자원을 해제할 수 있다.

### 복잡한 자원 관리의 문제
   ```java
   public static void inputAndWriteString() throws IOException {
       BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
       try {
           BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
           try {
                String line = br.readLine();
               bw.write(line);
           } finally {
               bw.close();
           }
       } finally {
           br.close();
       }
   }
   ```
여러 자원을 관리할 경우 **코드가 복잡해지고 가독성이 떨어진다.**

## 2. try-finally의 숨겨진 문제점

try-finally의 가장 큰 문제점은 예외 처리에 있다. inputString 메서드를 예로 들어 설명하면, try 블록 실행 중 기기에 문제가 생겨 readLine 메서드가 예외를 던지는 상황을 가정해 볼 수 있다. 이 경우 같은 이유로 finally 블록의 close 메서드도 예외를 던지게 된다.

이러한 상황에서 상위 메서드에서 예외 정보를 확인하면, **finally 블록에서 발생한 예외가 try 블록에서 발생한 예외를 덮어쓰게 된다.** 결과적으로 finally 블록의 예외만 확인하게 되어, **try 블록에서 발생한 원인 예외를 파악하지 못하게 된다.** 이는 문제의 근본 원인을 파악하기 어렵게 만든다.

물론 적절한 코드를 통해 최초 원인 예외를 확인할 수는 있지만, 이는 코드를 복잡하게 만들어 가독성을 크게 해치므로 추천하는 방법이 아니다.

이를 실제로 확인하기 위해 다음과 같은 코드를 작성하고 실행해 볼 수 있다:

```java
public class Application {

    public static void main(String[] args) {
        try {
            check();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void check() {
        try {
            throw new IllegalArgumentException();
        } finally {
            throw new NullPointerException();
        }
    }
}
```

이 코드를 실행하면 다음과 같은 결과가 나온다:

```
java.lang.NullPointerException 
    at Application.check(Application.java:14)  
    at Application.main(Application.java:4
```

main 메서드의 catch 블록에서 printStackTrace를 호출했을 때, finally 블록에서 던진 `NullPointerException`만 잡히고 try 블록의 `IllegalArgumentException`은 무시된 것을 볼 수 있다.

결론적으로 `try-finally` 구문은 **가독성을 해칠 가능성**이 높으며, **예외 처리 로직을 작성하는 데 있어 미묘한 결점**이 존재한다. 이러한 문제점들로 인해 try-finally 대신 더 나은 대안을 찾아야 할 필요성이 대두된다.


## 3. 해결책: try-with-resources

`try-with-resources`는 Java 7부터 도입된 기능으로, try-finally 방식의 단점을 보완하기 위해 만들어졌다. 이 방식을 사용하기 위해서는 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.

AutoCloseable 인터페이스는 다음과 같이 단순하게 구성되어 있다:

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

이 인터페이스는 단지 **close 메서드 하나만을 정의해 둔 간단한 구조**다. Java 라이브러리와 많은 서드파티 라이브러리의 클래스와 인터페이스들은 이미 AutoCloseable을 구현하거나 확장해 두었다. 따라서 **close가 필요한 자원 클래스를 직접 만들 때(커스텀)는 AutoCloseable을 반드시 구현**하는 것이 좋다.

try-with-resources의 사용 예시를 살펴보면 다음과 같다:

```java
public static String inputString() throws IOException {
    try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in))) {
        return br.readLine();
    }
}
```
(try 키워드와 함께 괄호 ()를 사용하여 자원을 선언함으로써 이 구문이 try-with-resources임을 나타낸다. 이 괄호 안에 선언된 모든 자원은 try 블록이 끝날 때 자동으로 close() 메서드가 호출된다.)

놀랍게도 코드의 가독성이 대단히 좋아졌다! 또한 가독성 뿐만이 아니다. 예외가 발생했을 때 디버깅 하기에도 더 편리하다. 아까 가정한 상황처럼 inputString 메서드의 readLine과 close 모두에서 예외가 발생하는 경우, **close(물론 코드 상으로는 보이지 않지만) 호출 시 발생하는 예외는 숨겨지고 readLine의 예외가 기록**된다.


`try-with-resources` 방식에서 숨겨진 **예외는 완전히 무시되는 것이 아니다.** 대신 `suppressed` 상태가 되어 stackTrace 출력 시 '숨겨졌다'는 메시지와 함께 표시된다. 이러한 suppressed 상태의 예외는 Java 7부터 도입된 `getSuppressed` 메서드를 통해 접근하고 활용할 수 있다.

다음의 테스트 코드를 통해 이를 확인할 수 있다:

```java
public class Application {

    public static void main(String[] args) {
        try {
            check();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void check() throws Exception {
        try (IllegalArgumentExceptionThrower thrower = new IllegalArgumentExceptionThrower()) {
            throw new NullPointerException();
        }
    }

    static class IllegalArgumentExceptionThrower implements AutoCloseable {
        @Override
        public void close() throws Exception {
            throw new IllegalArgumentException();
        }
    }
}
```

이 코드를 실행하면 다음과 같은 결과가 출력된다:

```
java.lang.NullPointerException  
    at Application.check(Application.java:17)
    at Application.main(Application.java:9)
    Suppressed: java.lang.IllegalArgumentException
        at Application$IllegalArgumentExceptionThrower.close(Application.java:24)
        at Application.check(Application.java:16)
        ... 1 more
```

이 출력 결과를 보면, **직접 throw한 NullPointerException이 주요 예외로 catch되어 출력**되는 것을 볼 수 있다. 그리고 close 메서드에서 발생한 `IllegalArgumentException`은 'Suppressed:' 태그 뒤에 추가로 출력된다.

이러한 메커니즘은 예외 처리의 명확성을 높여준다. **주요 예외(이 경우 NullPointerException)가 가장 먼저 표시되어 문제의 근본 원인을 쉽게 파악**할 수 있게 해주며, 동시에 자원을 닫는 과정에서 발생한 예외(IllegalArgumentException)도 함께 확인할 수 있게 해준다.

따라서 `try-with-resources` 방식은 예외 정보를 더 체계적으로 관리하고 표현할 수 있게 해주며, 이는 코드의 디버깅과 문제 해결을 더욱 효과적으로 만들어준다.

## 4. try-with-resources와 catch 블록의 병용:
`try-with-resources` 구조는 기존의 `try-finally`처럼 **catch 블록과 함께 사용**할 수 있다. 이를 통해 자원의 자동 해제와 예외 처리를 동시에 수행할 수 있다.

예시는 다음과 같다:
```java
public static String inputString() {
    try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in))) {
        return br.readLine();
    } catch (IOException e) {
        return "IOException 발생";
    }
}
```

이 구조를 사용하면 자원의 해제는 자동으로 처리되면서 동시에 발생할 수 있는 예외에 대한 핸들링도 가능하다.

📌 정리:
1. close를 통해 회수해야 하는 자원을 다룰 때는 `try-finally` 대신 반드시 `try-with-resources`를 사용해야 한다.

2. try-with-resources의 장점:
    - 가독성이 좋다.
    - 자원 회수가 쉽고 정확하다.
    - 예외 처리가 명확하다.
    - 숨겨진 예외도 suppressed 형태로 확인할 수 있다.

3. 커스텀 자원을 회수해야 하는 경우, `AutoCloseable` 인터페이스를 구현해야 한다.

4. try-with-resources는 catch 블록과 함께 사용할 수 있어 자원 해제와 예외 처리를 동시에 할 수 있다.

5. Java 7 이상에서는 getSuppressed 메서드를 통해 suppressed된 예외도 확인할 수 있다.

`try-with-resources`를 사용함으로써 코드는 더 간결해지고, 자원 관리는 더 안전해지며, 예외 처리는 더 명확해진다. 따라서 자원을 다루는 코드를 작성할 때는 항상 try-with-resources 사용을 고려해야 한다.