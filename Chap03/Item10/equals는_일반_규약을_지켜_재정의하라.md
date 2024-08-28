## Item10

## equals 메서드의 재정의

equals 메서드는 겉보기에 단순해 보이지만(Intellij에서 쉽게 생성 가능), 실제로는 **많은 주의가 필요한 메서드**이다. 잘못 구현하면 예기치 않은 문제를 일으킬 수 있으므로 신중히 접근해야 한다.

### equals 재정의가 불필요한 경우들

1. **본질적으로 고유한 인스턴스를 다루는 클래스**
   예를 들어, `Thread 클래스`는 각 인스턴스가 고유한 동작을 표현한다. 이런 경우 equals를 재정의할 필요가 없다.

1-1. 더 자세히
```java
@Override
public boolean equals(Object obj) {
    if (obj == this)
        return true; // 같은 인스턴스인 경우

    if (obj instanceof WeakClassKey) {
        Object referent = get(); // 현재 인스턴스의 참조 가져오기
        return (referent != null) && (referent == ((WeakClassKey) obj).get()); // 참조 비교
    } else {
        return false; // 서로 다른 클래스인 경우
    }
}
```

- **인스턴스 비교**:
obj == this로 같은 인스턴스인지 확인합니다.
- **참조 비교**:
두 WeakClassKey 인스턴스의 get() 메서드를 호출하여 실제 참조를 비교합니다. 이 경우, 값이 아닌 참조의 `동등성`을 확인합니다.

- **인스턴스 비교**: 객체의 내용(값)을 비교 
- **참조 비교**: 객체의 메모리 주소(참조)를 비교

- 그래서 왜 재정의를 하는 경우가 더 좋지 않은데?

- **고유한 동작 개체**
각 인스턴스의 고유성:
   **Thread 객체는 각각 독립적으로 실행되는 스레드**이다. 즉, 두 개의 Thread 인스턴스는 서로 다른 실행 흐름을 가지며, 그 자체로 의미가 있다.

- **동작의 중요성**
동작이 우선:
   Thread 객체는 스레드의 실행 상태나 동작을 나타내기 때문에, **두 스레드가 동일한 작업을 하는지 여부는 그 자체로 중요하지 않다.** 따라서, 동작이 아닌 인스턴스 자체가 중요하다.

2. **'논리적 동치성' 검사가 불필요한 경우**
   Pattern 클래스의 경우, 같은 정규표현식을 나타내는지 검사할 수 있지만 실제로 그럴 일이 거의 없다. 따라서 equals 재정의가 불필요하다.
```java
void patternTest() {
    final Pattern P1 = Pattern.compile("//.*");
    final Pattern P2 = Pattern.compile("//.*");

    System.out.println(P1.equals(P1)); // true
    System.out.println(P1.equals(P2)); // false
    System.out.println(P1.pattern()); // //.*
    System.out.println(P1.pattern().equals(P1.pattern())); // true
    System.out.println(P1.pattern().equals(P2.pattern())); // true
}
```

3. **상위 클래스의 equals가 하위 클래스에 적합한 경우**
   Set 인터페이스의 구현체들은 AbstractSet의 equals를 그대로 사용한다. 이는 Set의 동등성 비교가 크기와 내부 요소의 비교만으로 충분하기 때문이다.

```java
public boolean equals(Object o) {
    if (o == this) {
        return true;
    }

    if (!(o instanceof Set)) {
        return false;
    }
    
    Collection<?> c = (Collection<?>) o;
    if (c.size() != size()) { // 사이즈 비교
        return false;
    }
    try {
        return containsAll(c); // 내부 인스턴스 비교
    } catch (ClassCastException | NullPointerException unused) {
        return false;
    }
}
```

```java
void setTest() {
    Set<String> hash = new HashSet<>();
    Set<String> tree = new TreeSet<>();
    hash.add("Set");
    tree.add("Set");

    System.out.println(hash.equals(tree)) // true;
}
```

`Set`의 equals 메서드는 두 Set이 서로 다른 구현체라도, **크기와 요소를 기반으로 동치성을 판단**한다. 이로 인해 `HashSet`과 `TreeSet`과 같이 **서로 다른 구현체 간에도 같은 요소를 가지고 있다면** equals 메서드가 `true`를 반환할 수 있다.

4. **private 또는 package-private 클래스에서 equals 호출이 필요 없는 경우**
   이런 경우, equals 메서드를 오버라이드하여 호출을 금지할 수 있다.

```java
@Override 
public boolean equals(Object o){
    throw new AssertionError(); // 호출 금지
}
```

- **호출 금지**: 클래스가 `private`이거나 `package-private`인 경우, 외부 클래스에서 해당 클래스의 인스턴스를 생성하거나 사용할 수 없기 때문에 equals 메서드가 호출될 일이 없다.
- **의도 표현**: equals 메서드를 오버라이드하면서 `AssertionError`를 던지는 것은 개발자가 의도적으로 **해당 메서드가 호출되지 않도록 하려는 것**이다. 이는 이 클래스의 인스턴스가 비교되는 상황이 발생하지 않도록 보장하는 방법이다.

📌 결론적으로 equals는 논리적인 동치성을 확인하고 싶을 때 재정의 한다.

## equals의 일반 규약

equals 메서드는 동등성 비교를 위해 사용되며, 다음의 규약을 반드시 준수해야 한다. 이 규약들은 모든 객체에 대해 적용되며, null이 아닌 참조 값 x, y, z에 대해 다음과 같은 특성을 가져야 한다:

1. **반사성**
   객체는 자기 자신과 반드시 같아야 한다. 즉, `x.equals(x)`는 항상 true여야 한다.

2. **대칭성**
   두 객체의 equals 비교 결과는 서로 같아야 한다. `x.equals(y)`가 true라면, `y.equals(x)`도 true여야 한다.

3. **추이성**
   equals 관계는 추이적이어야 한다. `x.equals(y)`가 true이고 `y.equals(z)`가 true라면, `x.equals(z)`도 true여야 한다.

4. **일관성**
   객체가 수정되지 않는 한, equals 메서드는 항상 같은 결과를 반환해야 한다.

5. **null 비교**
   모든 객체는 null과 같지 않아야 한다. `x.equals(null)`은 항상 false를 반환해야 한다.

이러한 규약들은 복잡해 보일 수 있지만, **객체 간의 일관된 동등성 비교를 위해 반드시 지켜져야 한다**. 존 던의 말처럼, 어떤 클래스도 독립적으로 존재하지 않기 때문에 이러한 규약의 준수는 매우 중요하다.

### 1. 반사성
   null이 아닌 모든 참조 값 x에 대해 x.equals(x)를 만족해야 한다. 즉, 자기 자신과 같아야 한다.
   
### 2. 대칭성
대칭성 위반의 예시
```java
public final class CaseInsensitiveString {

    private final String str;
    
    public CaseInsensitiveString(String str) {
        this.str = Objects.requireNonNull(str);
    }
    
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return str.equalsIgnoreCase(((CaseInsensitiveString) o).str);
        }
    
        if (o instanceof String) { // 한 방향으로만 작동한다.
            return str.equalsIgnoreCase((String) o);
        }
        return false;
    }
}

void symmetryTest() {
    CaseInsensitiveString caseInsensitiveString = new CaseInsensitiveString("Test");
    String test = "test";
    System.out.println(caseInsensitiveString.equals(test)); // true
    System.out.println(test.equals(caseInsensitiveString)); // false
}
```
`caseInsensitiveString.equals(test)` 호출 시, **caseInsensitiveString 객체가 String과 비교되므로 대소문자 구분 없이 true를 반환**한다.
반면, `test.equals(caseInsensitiveString)` 호출 시, test는 String이므로 CaseInsensitiveString 인스턴스와 비교를 시도하지만, **equals 메서드에서 String을 CaseInsensitiveString으로 처리하는 경우가 없기 때문에 false를 반환**합니다.

- 예를 들어
```java
if (o instanceof CaseInsensitiveString) {
// o는 CaseInsensitiveString의 인스턴스입니다.
}
```
- 이 조건이 참이면 o는 CaseInsensitiveString의 인스턴스이다. 반면에 test는 단순한 String 객체이기 때문에, CaseInsensitiveString의 인스턴스가 아니다.

### 3. 추이성

추이성 위반의 예

```java
public class Point {

    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return this.x == p.x && this.y == p.y;
    }
}
```

```java
public class ColorPoint extends Point {

    private final Color color;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof ColorPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.color == ((ColorPoint) o).color;
    }
}
```
- 먼저 o가 Point의 인스턴스인지 확인한다.
- o가 ColorPoint가 아닐 경우, o.equals(this)를 호출하여 Point의 equals 메서드를 사용한다. 즉, 색상은 무시하고 좌표만 비교한다.
- o가 ColorPoint인 경우, super.equals(o)를 호출하여 좌표를 비교하고, 색상도 비교한다.
```java
void transitivityTest() {
    ColorPoint a = new ColorPoint(2, 3, Color.RED);
    Point b = new Point(2, 3);
    ColorPoint c = new ColorPoint(2, 3, Color.BLUE);

    System.out.println(a.equals(b)); // true
    System.out.println(b.equals(c)); // true
    System.out.println(a.equals(c)); // false
}
```
1. **a.equals(b)는 true**: ColorPoint인 a와 Point인 b의 좌표가 동일하므로 `true`를 반환한다.
2. **b.equals(c)는 true**: Point인 b와 ColorPoint인 c의 좌표가 동일하므로 `true`를 반환한다.
3. **a.equals(c)는 false**: ColorPoint인 a와 ColorPoint인 c의 좌표는 동일하지만 **색상이 다르므로** `false`를 반환한다.

### 3-1. 추이성(무한 재귀)

```java
public class SmellPoint extends Point {

    private final Smell smell;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
        if (!(o instanceof SmellPoint)) {
            return o.equals(this);
        }

        // o가 ColorPoint이면 색상까지 비교한다.
        return super.equals(o) && this.smell == ((SmellPoint) o).smell;
    }
}

void infinityTest() {
    Point cp = new ColorPoint(2, 3, Color.RED);
    Point sp = new SmellPoint(2, 3, Smell.SWEET);

    System.out.println(cp.equals(sp));
}
```

equals 메서드에서 o가 SmellPoint가 아닐 때 `o.equals(this)`를 호출한다.
이때 o가 Point의 인스턴스인 경우, o가 ColorPoint일 수 있다. ColorPoint의 equals 메서드는 Point에 대한 비교를 수행하고, 다시 **o.equals(this)를 호출**하게 된다.

이렇게 되면 **SmellPoint의 equals 메서드가 계속해서 서로를 호출하게 되어 무한 재귀가 발생**한다.

### 3-2. 추이성(리스코프 치환 원칙)

만약 추이성을 지키기 위해서 Point의 equals를 각 클래스들을 `getClass`를 통해서 같**은 구체 클래스일 경우에만 비교**하도록 하면 어떨까?

**리스코프 치환 원칙 (Liskov Substitution Principle)**??

### 1. **리스코프 치환 원칙 (LSP)**

리스코프 치환 원칙은 객체 지향 설계에서 중요한 원칙 중 하나로, 서브타입은 언제나 그 상위 타입으로 교체할 수 있어야 한다는 것이다. 즉, 어떤 타입의 객체를 해당 타입의 서브타입으로 교체했을 때, 프로그램의 행동이 바뀌지 않아야한다.

### 2. **getClass()를 사용한 equals 메서드**

제안된 `equals` 메서드는 `getClass()`를 사용하여 두 객체가 같은 클래스인지 확인한다:

```java
@Override
public boolean equals(Object o) {
    if (o == null || o.getClass() != this.getClass()) {
        return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y == p.y;
}
```

이렇게 하면 두 객체가 같은 클래스일 때만 비교하도록 되어 있다. 하지만 이 방식은 `리스코프 치환 원칙`을 위반한다.

### 3. **왜 LSP를 위반하는가?**

- **서브타입의 특성**: 서브타입은 상위 타입의 행동을 확장하거나 수정할 수 있어야 한다. 하지만 `getClass()`를 사용하면 서브타입의 객체가 상위 타입의 객체와 비교할 수 없게 된다.
- 예를 들어, `Point` 클래스와 그 서브클래스인 `ColorPoint`가 있을 때, `ColorPoint`의 인스턴스는 `Point`로 간주되어야 한다. 그러나 `getClass()`를 사용하면 `ColorPoint`와 `Point`는 서로 다른 클래스로 인식되므로, `equals` 메서드에서 서로 비교할 수 없다. 
- 결과적으로, `ColorPoint` 객체를 `Point`로 교체했을 때 프로그램의 동작이 변경될 수 있다.

### 4. **객체 지향적 추상화의 이점 포기**

- `getClass()`를 사용하면 서브타입의 특성을 고려하지 않게 되므로, 객체 지향의 장점인 다형성을 활용할 수 없다.
- 객체 지향 설계에서의 동치관계는 클래스 계층 구조에서의 상속과 다형성을 바탕으로 해야 하며, `getClass()`를 사용하면 이를 포기하게 된다.

## 4. 일관성

null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

```java
URL url1 = new URL("www.example.com");
URL url2 = new URL("www.example.com");
System.out.println(url1.equals(url2)); // 결과가 일정하지 않을 수 있다
```

java.net.URL 클래스의 equals 메서드는 호스트의 IP 주소를 비교에 사용한다. **네트워크 상태에 따라 IP 주소가 변경될 수 있어** 같은 URL 객체라도 equals 결과가 달라질 수 있다.
이런 문제를 피하기 위해서는 **equals 메서드가 메모리에 존재하는 객체만을 사용한 결정적 계산을 수행**해야 한다. 외부 자원에 의존하는 비교는 피해야 한다.

## 5. null-아님

null이 아닌 모든 참조 값 x에 대해, **x.equals(null)은 false**
o.equals(null) == true인 경우는 상상하기 어렵지만, 실수로 `NullPointerException`을 던지는 코드 또한 허용하지 않는다.

```java
@Override 
public boolean equals(Object o) {
    // 우리가 흔하게 인텔리제이를 통해서 생성하는 equals는 다음과 같다.
    if (o == null || getClass() != o.getClass()) {
        return false;
    }
    
    // 책에서 추천하는 내용은 null 검사를 할 필요 없이 instanceof를 이용하라는 것이다.
    // instanceof는 두번째 피연산자(Point)와 무관하게 첫번째 피연산자(o)거 null이면 false를 반환하기 때문이다. 
    if (!(o instanceof Point)) {
        return false;
    }
}
```

## 좋은 재정의 방법
```java
@Override
public boolean equals(final Object o) {
    // 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    if (this == o) {
        return true;
    }

    // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    if (!(o instanceof Point)) {
        return false;
    }

    // 3. 입력을 올바른 타입으로 형변환 한다. (Point 타입으로 안전하게 형변환한다. instanceof를 통해 타입이 확인되었으므로 안전하다.
    final Piece piece = (Piece) o;

    // 4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
    
    // float와 double을 제외한 기본 타입 필드는 ==를 사용한다.
    return this.x == p.x && this.y == p.y;
    
    // 필드가 참조 차입이라면 equals를 사용한다.
    return str1.equals(str2);
    
    // null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
    return Objects.equals(Object, Object);
}
```

## 마무리

equals 메서드 재정의에 대한 주요 지침을 다음과 같이 요약할 수 있다.

1. **재정의 필요성 검토**

equals 메서드는 꼭 필요한 경우가 아니면 재정의하지 않는 것이 좋다. 기본 Object.equals()가 클래스의 의미에 맞는 동등성을 제공한다면 그대로 사용하는 것이 안전하다.

2. **규약 준수**

재정의가 필요한 경우, 반사성, 대칭성, 추이성, 일관성, null-아님의 다섯 가지 규약을 반드시 지켜야 한다. 이는 equals 메서드의 올바른 동작을 보장하기 위해 중요하다.

3. **성능 최적화**

equals 메서드의 성능은 필드 비교 순서에 따라 달라질 수 있다. 따라서:
- 다를 가능성이 높은 필드를 먼저 비교한다.
- 비교 비용이 저렴한 필드를 먼저 비교한다.

이렇게 하면 불필요한 비교를 줄여 성능을 향상시킬 수 있다.

4. **비교 대상 선별**

- 객체의 논리적 상태와 무관한 필드는 비교하지 않는다.
- 핵심 필드에서 파생되는 필드는 비교할 필요가 없다. 둘 중 하나만 비교하면 된다.

5. **hashCode 메서드 재정의**

equals를 재정의할 때는 반드시 hashCode 메서드도 함께 재정의해야 한다. 이는 HashMap, HashSet 등의 컬렉션에서 객체를 올바르게 처리하기 위해 필요하다.

6. **매개변수 타입**

equals 메서드의 매개변수는 반드시 Object 타입으로 선언해야 한다. 다른 타입을 사용하면 Object.equals를 재정의하는 것이 아니라 다른 메서드를 정의하게 되어 예상치 못한 문제가 발생할 수 있다.

예시
```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyClass)) return false;
    MyClass other = (MyClass) o;
    return this.field1 == other.field1 && 
           Objects.equals(this.field2, other.field2);
}
```

이러한 지침을 따르면 견고하고 효율적인 equals 메서드를 구현할 수 있다. 항상 주의 깊게 설계하고, 필요한 경우에만 재정의하는 것이 중요하다.



