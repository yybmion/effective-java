## Item 3

### 싱글턴 패턴의 특징과 한계

`싱글턴 패턴`은 객체의 인스턴스가 **오직 하나만 생성되는 것을 보장**하는 디자인 패턴이다.
주로 생성자를 `private`으로 선언하고, 객체 접근을 위한 static 메서드를 제공함으로써 구현한다.
Spring Framework에서는 기본적으로 Bean들이 ApplicationContext 범위 내에서 싱글턴으로 관리된다. 이는 애플리케이션의 성능과 리소스 관리에 도움이 될 수 있다.

그러나 `싱글턴 패턴`에는 몇 가지 중요한 **한계**가 있다:

1. **상속의 제한**: private 생성자로 인해 상속이 불가능하다. 이는 객체 지향 프로그래밍의 주요 특징인 상속과 다형성을 활용할 수 없게 만든다.
   - `상속`: 서브클래스는 슈퍼클래스의 생성자를 호출하여 인스턴스를 생성해야 한다. 그러나 private 생성자는 서브클래스에서도 접근할 수 없기 때문에, 서브클래스의 인스턴스를 생성할 수 없다.
2. **테스트의 어려움**: 싱글턴 객체는 그 생성 방식이 제한적이어서 단위 테스트 시 mock 객체로 대체하기 어렵다. 이는 테스트 주도 개발(TDD)을 어렵게 만들 수 있다.
3. **다중 환경에서의 불확실성**: 서버 환경, 특히 여러 JVM에 분산된 환경에서는 싱글턴 객체가 하나만 생성된다는 보장이 없다. 클래스 로더의 구성에 따라 여러 인스턴스가 생성될 수 있다.
4. **전역 상태의 위험성**: 싱글턴은 사실상 전역 객체로 작용할 수 있어, 객체 지향 설계 원칙에 위배된다. 이는 코드의 결합도를 높이고 유지보수를 어렵게 만들 수 있다.
   - `전역 상태`: 싱글톤 패턴은 클래스의 인스턴스를 오직 하나만 생성하여 애플리케이션 전역에서 사용하도록 한다. 이렇게 되면 해당 인스턴스가 애플리케이션의 여러 부분에서 공유되기 때문에, 전역 상태가 형성된다.
   - `문제점`: 전역 상태는 프로그램의 여러 부분에서 접근할 수 있는 데이터나 객체를 의미한다. 이는 예기치 않은 부작용을 초래할 수 있으며, 코드의 가독성과 유지보수성을 떨어뜨릴 수 있다.

이러한 한계들로 인해, 많은 개발자들은 싱글턴 패턴 대신 `의존성 주입(Dependency Injection)`이나 static 유틸리티 클래스 등의 대안을 선호한다. 이러한 방식들은 싱글턴의 장점을 유지하면서도 그 한계를 극복할 수 있게 해준다.

### 싱글턴 사용 이유와 구현 방식

**싱글턴 사용 이유**:
1. **메모리 효율성**: 객체를 한 번만 생성하여 재사용함으로써 메모리 낭비를 방지한다.
2. **전역 접근성**: 싱글턴 객체는 전역적으로 접근 가능하여 다른 객체와의 공유가 용이하다.

**싱글턴 구현 방식**:
두 가지 주요 방식이 있으며, 둘 다 **private 생성자와 public static 멤버**를 사용한다.

1. **public static final 필드 방식**:

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```
(참고) Java에서 클래스가 처음 사용될 때, JVM은 해당 클래스를 로드하고 초기화한다. **이 과정에서 static 필드가 초기화**된다.

이 방식의 특징:
- Elvis.INSTANCE 초기화 시 생성자가 한 번만 호출된다.
- 시스템 전체에서 인스턴스가 하나만 존재함을 보장한다.

**장점**:
1. 싱글턴임이 API에 명확히 드러난다.
2. 구현이 간결하다.

사용 예시:
```java
@Test
public void singletonTest() {
    Elvis elvis1 = Elvis.INSTANCE;
    Elvis elvis2 = Elvis.INSTANCE;
    assertSame(elvis1, elvis2); // 성공
}
```

예외 사항:
리플렉션 API를 사용하면 private 생성자에 접근할 수 있어 싱글턴 속성을 깨뜨릴 수 있다.

```java
import java.lang.reflect.Constructor;

public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { /* 생성자 내용 */ }

    public void leaveTheBuilding() {
        System.out.println("Elvis has left the building.");
    }
}

// 리플렉션을 사용하여 싱글톤 속성을 깨뜨리는 코드
public class Main {
    public static void main(String[] args) throws Exception {
        // 기존 싱글톤 인스턴스
        Elvis instance1 = Elvis.INSTANCE;

        // 리플렉션을 사용하여 private 생성자에 접근
        Constructor<Elvis> constructor = Elvis.class.getDeclaredConstructor();
        constructor.setAccessible(true); // 접근 제한을 무시하고 접근 가능하게 설정

        // 새로운 인스턴스 생성
        Elvis instance2 = constructor.newInstance();

        // 두 인스턴스 비교
        System.out.println(instance1 == instance2); // false 출력
    }
}

```

이 방식은 간단하고 명확하지만, `리플렉션`을 통한 공격에 취약할 수 있다.
따라서 보안이 중요한 상황에서는 추가적인 방어 로직이 필요할 수 있다.

**해결 방법**
생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.

```java
private Elvis() {
if(INSTANCE != null){
throw new RuntimeException("생성자를 호출할 수 없습니다!");
}
}
```

2. 정적 팩터리 메서드를 사용한 싱글턴 방식

이 방식은 public static 멤버로 **정적 팩터리 메서드**를 제공한다.

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
    public void leaveTheBuilding() { ... }
}
```

**특징**:
- `Elvis.getInstance()`는 항상 동일한 객체 참조를 반환한다.
- 두 번째 Elvis 인스턴스는 생성되지 않는다.

사용 예시:
```java
@Test
public void getInstance() {
    Elvis elvis1 = Elvis.getInstance();
    Elvis elvis2 = Elvis.getInstance();
    assertSame(elvis1, elvis2); // 성공
}
```

**장점**:
1. **유연성**: API 변경 없이 싱글턴이 아닌 방식으로 변경 가능하다. 내부 구현만 수정하면 된다.
2. **다양한 인스턴스**: 필요시 스레드별로 다른 인스턴스를 반환하도록 수정할 수 있다.
3. **제네릭 활용**: 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
4. **메서드 참조**: 정적 팩터리 메서드를 `Supplier`로 사용할 수 있다.

Supplier 활용 예시:
```java
Supplier<Elvis> elvisSupplier = Elvis::getInstance;
Elvis elvis = elvisSupplier.get();
```

이 방식은 메서드 참조를 Supplier 타입으로 만들어 사용할 수 있다. **Supplier는 get 메서드 하나만 가진 인터페이스**로, 어떤 타입이든 반환할 수 있다.

정적 팩터리 방식은 더 유연하고 다양한 상황에 적용할 수 있는 장점이 있다. 하지만 구현이 조금 더 복잡하고, 정적 팩터리 메서드를 통해 접근해야 한다는 점을 고려해야 한다.

### 두 싱글턴 방식의 공통적인 문제점과 해결책

**문제**:
**직렬화 후 역직렬화 과정**에서 새로운 인스턴스가 생성될 수 있다. 이는 싱글턴 속성을 깨뜨린다.

**원인**:
**역직렬화는 기본적으로 새로운 인스턴스를 생성**하며, 이 과정에서 생성자를 호출하지 않고 값을 복사한다.

- 문제 예시 코드

```java
import java.io.*;

public class Main {
    public static void main(String[] args) {
        try {
            // 1. 인스턴스를 직렬화
            ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("elvis.ser"));
            out.writeObject(Elvis.INSTANCE);
            out.close();

            // 2. 직렬화된 인스턴스를 역직렬화
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("elvis.ser"));
            Elvis elvisFromFile = (Elvis) in.readObject();
            in.close();

            // 3. 두 인스턴스 비교
            System.out.println("Are both instances the same? " + (Elvis.INSTANCE == elvisFromFile)); // false 출력

            // 4. 가짜 인스턴스의 이름 출력
            System.out.println("Elvis from file name: " + elvisFromFile.name); // "Elvis Presley"
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

1. **직렬화**: Elvis.INSTANCE를 직렬화하여 elvis.ser 파일에 저장한다.
2. **역직렬화**: elvis.ser 파일에서 객체를 읽어와 새로운 elvisFromFile 인스턴스를 생성한다.
3. **비교**: Elvis.INSTANCE와 elvisFromFile을 비교하면 false가 출력된다. 이는 서로 다른 인스턴스라는 것을 의미한다.
4. **가짜 인스턴스**: elvisFromFile은 원래의 Elvis.INSTANCE와는 다른 새로운 인스턴스이므로, 가짜 인스턴스가 생성된 것이다.

**해결책**:
1. `readResolve()` 메서드 구현
2. 모든 인스턴스 필드에 `transient` 키워드 적용

**구체적인 방법**:
1. Serializable 인터페이스 구현
2. 모든 인스턴스 필드를 transient로 선언
3. readResolve() 메서드 추가

예시 코드:
```java
public class Elvis implements Serializable {
    private static final transient Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    // 역직렬화 시 호출되는 메서드
    private Object readResolve() {
        return INSTANCE;
    }
}
```

**readResolve() 메서드의 역할**:
- 역직렬화 과정에서 자동으로 호출된다.
- 이 메서드가 반환하는 객체로 역직렬화된 객체를 대체한다.
- 싱글턴의 경우, 항상 동일한 인스턴스(INSTANCE)를 반환하여 싱글턴 특성을 유지한다.

이 방식을 통해:
- '진짜' 싱글턴 인스턴스가 반환된다.
- 역직렬화 과정에서 생성된 '가짜' 인스턴스는 가비지 컬렉터에 의해 처리된다.

이러한 방법을 통해 직렬화 상황에서도 싱글턴 특성을 유지할 수 있다. 하지만 이는 **복잡성을 증가**시키므로, 직렬화가 필요한 상황에서만 사용하는 것이 좋다.


3. 열거 타입(Enum)을 이용한 싱글턴 방식

이 방식은 원소가 하나인 열거 타입을 선언하여 싱글턴을 구현한다. 대부분의 상황에서 이 방법이 싱글턴을 만드는 가장 좋은 방법으로 여겨진다.

코드 예시:
```java
public enum Elvis {
    INSTANCE;

    public String getName() {
        return "Elvis";
    }

    public void leaveTheBuilding() { ... }
}
```

**사용 방법**:
```java
String name = Elvis.INSTANCE.getName();
```

**특징**:
1. Elvis 타입의 인스턴스는 INSTANCE 하나뿐이며, 추가 인스턴스 생성이 불가능하다.
2. **직렬화와 리플렉션 공격에도 안전**하다.

**장점**:
1. **간결성**: 코드가 간결하고 이해하기 쉽다.
2. **안전성**: 복잡한 직렬화 상황이나 리플렉션 공격에서도 추가 인스턴스 생성을 완벽히 방지한다.
3. **직렬화 지원**: 별도의 처리 없이 직렬화가 자동으로 처리된다.

**제한사항**:
- **열거 타입이 다른 클래스를 상속할 수 없다.** 따라서 싱글턴이 특정 상위 클래스를 상속해야 하는 경우에는 이 방법을 사용할 수 없다.

**결론**:
열거 타입을 이용한 싱글턴 구현은 대부분의 상황에서 최선의 방법이다. 코드가 간결하고, 추가적인 노력 없이 직렬화와 리플렉션 공격에 대한 안전성을 보장받을 수 있다. 다만 상속이 필요한 경우에는 사용할 수 없다는 제한이 있으므로, 이런 경우에는 다른 방식을 고려해야 한다.

### Reference
https://velog.io/@lychee/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%95%84%EC%9D%B4%ED%85%9C-3.-private-%EC%83%9D%EC%84%B1%EC%9E%90%EB%82%98-%EC%97%B4%EA%B1%B0-%ED%83%80%EC%9E%85%EC%9C%BC%EB%A1%9C-%EC%8B%B1%EA%B8%80%ED%84%B4%EC%9E%84%EC%9D%84-%EB%B3%B4%EC%A6%9D%ED%95%98%EB%9D%BC