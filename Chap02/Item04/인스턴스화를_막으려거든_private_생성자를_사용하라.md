## Item 04

"인스턴스화를 막으려거든 private 생성자를 사용하라"

정적 메소드와 **정적 필드만을 포함하는 유틸리티 클래스**는 종종 유용하게 사용된다. 이런 클래스들은 인스턴스화할 필요가 없으며, 실제로 인스턴스화를 방지하는 것이 좋다. 주로 다음과 같은 경우에 사용된다:

1. **기본 타입 값이나 배열 관련 메소드를 모은 클래스**

예시로 `java.lang.Math`나 `java.util.Arrays`가 있다. java.util.Arrays의 코드를 보면:

```java
public class Arrays {
    public static boolean isArray(Object o) { ... }
    public static Object[] asObjectArray(Object array) { ... }
    public static List<Object> asList(Object array) { ... }
    public static <T> boolean isNullOrEmpty(T[] array) { ... }
    // ...
    private Arrays() {
    }
}
```

이 클래스의 특징:
- 모든 메소드가 **static으로 선언**되어 있다.
- **생성자가 private로 선언**되어 있다.

사용 방식:
- `Arrays arrays = new Arrays()`와 같이 인스턴스를 생성하지 않는다.
- Arrays.asList(배열)처럼 직접 static 메소드를 호출한다.

**private 생성자를 사용함으로써 얻는 이점**:
1. **인스턴스화 방지**: 클래스 외부에서 new 키워드로 인스턴스를 생성할 수 없다.
2. **상속 방지**: 하위 클래스를 만들 수 없어 의도치 않은 확장을 막을 수 있다.
3. **명확성**: 이 클래스가 인스턴스화를 위한 것이 아님을 명확히 한다.

이러한 방식은 유틸리티 클래스의 설계 의도를 명확히 하고, 잘못된 사용을 방지하는 데 도움을 준다.

2. **인터페이스 구현 객체를 생성하는 정적 메소드(팩토리)를 모은 클래스**

이런 유형의 클래스는 특정 인터페이스를 구현하는 객체를 생성하는 정적 메소드(팩토리 메소드)들을 모아놓은 것이다. 대표적인 예로 `java.util.Collections`가 있다.

`java.util.Collections`의 특징:
- 모든 메소드가 static으로 선언되어 있다.
- private 생성자를 가지고 있어 인스턴스화를 방지한다.

예시 코드 구조:
```java
public class Collections {
    // 정적 팩토리 메소드들
    public static <T> List<T> emptyList() { ... }
    public static <T> Set<T> emptySet() { ... }
    public static <K,V> Map<K,V> emptyMap() { ... }
    public static <T> List<T> singletonList(T o) { ... }
    // ...

    // private 생성자
    private Collections() {
    }
}
```

**이 클래스의 사용 방식**:
- `new Collections()`로 인스턴스를 생성하지 않는다.
- `Collections.emptyList()`, `Collections.singletonList(element)` 등과 같이 직접 static 메소드를 호출한다.


3. **final 클래스와 관련된 메소드들을 모아놓을 때**

final 클래스와 관련된 메소드를 한 곳에 모아두는 경우가 있다. final 클래스는 **보안상 상속을 금지**한다. 이는 final 클래스를 상속해 하위 클래스를 만드는 것이 시스템에 위험을 줄 수 있기 때문이다.

대표적인 final 클래스의 예로 `String 클래스`가 있다. String 클래스는 다음과 같이 선언된다:

```
public final class String implements java.io.Serializable, Comparable<String>, CharSequence { ... }
```

final 클래스와 관련된 메소드를 모아둔 클래스의 예시로 junit의 `StringUtils`를 볼 수 있다:

```
@API(status = INTERNAL, since = "1.0")
public final class StringUtils {
    private static final Pattern ISO_CONTROL_PATTERN = compileIsoControlPattern();
    private static final Pattern WHITESPACE_PATTERN = Pattern.compile("\\s");

    static Pattern compileIsoControlPattern() { ... }

    private StringUtils() {
        /* no-op */
    }

    public static boolean isBlank(String str) { ... }

    public static boolean isNotBlank(String str) { ... }

    public static boolean containsWhitespace(String str) { ... }

    // ...
}
```

이 클래스의 메소드들도 static으로 선언되어 있어 **객체 생성 없이 바로 호출**할 수 있다. 이런 구조는 유틸리티 클래스에서 자주 볼 수 있는 패턴이다.

`유틸리티 클래스`는 기본적으로 인스턴스로 만들어 쓰려고 설계한 것이 아니다!

___

### 생성자를 명시하지 않으면 되지 않을까?
생성자를 명시하지 않으면 컴파일러가 자동으로 `public`으로 기본 생성자를 생성해준다. 사용자가 코드만 보고 생성자가 없다고 생각하더라도, 컴파일 시 자동으로 생성되는 것이다.

### 추상 클래스로 만들어서 인스턴스화를 막기?
`abstract` 클래스는 인스턴스로 생성하는 것이 불가능하다. 따라서 `private` 생성자 대신에 사용할 수 있을지에 대한 의문이 생길 수 있다.

하지만 하위 클래스를 생성하면 인스턴스화가 가능해진다. `abstract` 클래스는 보통 클래스들의 공통 필드와 메소드를 정의하는 목적으로 만들어지기 때문에, 상속해서 사용하라는 의미로 오해할 수 있는 것이다.



