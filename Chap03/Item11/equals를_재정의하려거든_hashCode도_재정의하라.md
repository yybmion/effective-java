## Item11

```java

public class Alcohol {

    private String category;

    public Alcohol(String category) {
        this.category = category;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }

        Alcohol alcohol = (Alcohol) o;

        return Objects.equals(category, alcohol.category);
    }
}
```

```java
public class Main {

    public static void main(String[] args) {

        Alcohol 참이슬 = new Alcohol("소주");
        Alcohol 처음처럼 = new Alcohol("소주");

        System.out.println(참이슬.hashCode());
        System.out.println(처음처럼.hashCode());
        System.out.println(참이슬.eqauls(처음처럼));
    }
}

//    1456208737 
//    288665596
//    true
```

```java
public class Main {
    
    public static void main(String[] args) {
        Map<Alcohol, Integer> 알콜_도수 = new HashMap<>();
        알콜_도수.put(참이슬, 16);
        System.out.println(알콜_도수.get(참이슬));
        System.out.println(알콜_도수.get(처음처럼));
    }
}

// 16
// null
```

이 코드를 보여주는 취지는 무엇일까?

Alcohol 클래스의 equals 메서드는 **객체의 동등성을 비교**하기 위해 작성된 코드다.
하지만 `hashCode` 메서드가 재정의되지 않았기 때문에, 두 개의 Alcohol **객체가 동일하다고 판단되더라도** **동일한 해시 코드를 반환하지 않을 수** 있다. 
이를 통해 발생할 수 있는 문제를 이제 알아보려한다.

- **동일 객체의 해시 코드 불일치**: equals 메서드가 두 객체의 category가 같다고 판단하더라도, 각 객체의 hashCode는 기본 구현에 따라 다르게 계산된다.
- 이는 해시 기반 컬렉션(예: HashMap, HashSet)에서 문제를 일으킬 수 있다.

### Object 클래스의 명세에서 hashCode와 관련된 중요한 원칙

1. **불변성**: equals 비교에 사용되는 정보가 변경되지 않았다면, 객체의 **hashCode 값은 항상 동일**해야 한다. 즉, 객체의 상태가 변하지 않으면 해시 코드는 변하지 않아야 한다.

2. **동등성 유지**: 두 객체가 equals 메서드로 동일하다고 판단되면, 두 객체의 hashCode도 반드시 동일한 값을 반환해야 한다. 이는 해시 기반 컬렉션에서의 일관성을 보장한다.

3. **비유일성**: 두 객체가 equals 메서드로 다르다고 판단되더라도, 두 객체의 **hashCode가 서로 다를 필요는 없다.** 즉, 서로 다른 두 객체가 동일한 해시 코드를 가질 수 있다. 이를 `"해시 충돌"`이라고 하며, 해시 테이블에서는 일반적인 상황이다.

___

> 그냥 아무 값이나 리턴하면 안돼?

```java
@Override
public int hashCode() {
    return 32;
    }
```

모든 인스턴스가 **동일한 해시 코드를 반환**하면, 해시 기반 컬렉션에서 모든 객체가 같은 버킷에 저장된다.
이로 인해 `해시 충돌`이 발생하게 되고, 충돌 처리 방식에 따라 성능이 크게 저하될 수 있다.

예를 들어, `HashSet`에서 객체를 탐색할 때, `O(1)`의 성능을 기대할 수 있지만, 모든 객체가 동일한 해시 코드를 가지면 `O(N)`의 성능으로 느려질 수 있다.

### O(1),O(N)???
- **O(1) 성능**: 해시 코드를 통해 객체의 위치를 직접 계산할 수 있기 때문에, 일반적으로 객체를 찾는 데 걸리는 시간은 상수 시간 `O(1)`이다.
- **O(N) 성능**: 만약 모든 객체가 동일한 해시 코드를 가진다면, 모든 객체가 하나의 버킷에 저장된다. 이 경우, 객체를 찾기 위해 이 **버킷 내의 모든 객체를 순회**해야 하므로 시간 복잡도가 `O(N)`으로 증가한다. 여기서 N은 버킷에 저장된 객체의 수다.

```java
intellij default
@Override
    public int hashCode() {
        return category != null ? category.hashCode() : 0;
    }
    
java 7+
@Override
    public int hashCode() {
        return Objects.hash(category);
    }
```

이 구현은 category 값에 기반하여 해시 코드를 생성하므로, **서로 다른 카테고리를 가진 객체들은 서로 다른 해시 코드를 가진다.**
이는 효율적인 데이터 검색을 가능하게 한다.

Java 7 이상의 구현은 다음과 같이 `Objects.hash` 메서드를 사용하여 여러 필드를 기반으로 해시 코드를 쉽게 생성할 수 있다

___

### 좋은 해시코드를 작성하기 위한 요령

1. int 변수 result 를 선언한 후 값 c로 초기화한다. 이 때 c는  해당 객체의 첫 번째 핵심필드를 단계 2.1 방식으로 계산한 해시코드이다.

2. 해당 객체의 나머지 핵심필드 f 각각에 대해 다음 작업을 수행한다 
   2-1. 해당 필드의 해시코드 c를 계산한다
     2-1-1. 기본타입 필드라면 Type.hashCode(f)를 수행한다
     2-1-2. 참조타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals 를 재귀적으로 호출한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 필드의 값이 null 이면 0을 사용한다.
     2-1-3.필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 배열에 핵심 원소가 하나도 없다면 상수 0을 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
   2-2. 단계 2.1 에서 계산한 해시코드 c로 result를 갱신한다.
   ex) result = 31 * result + c;

3. result를 반환한다.

```java

public class Address {
    
    private String 도시;
    private String 구;
    private String 동;
    
    @Override
    public int hashCode() {
        int result = String.hashCode(도시);
        result = 31 * result + String.hashCode(구);
        result = 31 * result + String.hashCode(동);
        return result;
    }
}
```

```java
java 7+
@Override
public int hashCode() {
    return Objects.hash(도시, 구, 동);
```

### 불변 클래스와 해시 코드

`불변 클래스`는 한 번 생성된 객체의 상태를 변경할 수 없는 클래스를 의미한다.
이러한 클래스에서는 해시 코드를 한 번만 계산하고, 이후에는 계산된 값을 **재사용하는 방식이 효율적**이다.

```java
private int hashCode;

@Override 
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        // 해시코드 계산
        hashCode = result;
    }
    return result;
}

```
**private int hashCode;**는 클래스의 해시 코드를 저장할 필드다. `초기값은 0`이다.
hashCode가 0인 경우(객체 처음 생성될때), 해시 코드를 계산하지만, 이후에 hashCode() 메서드가 호출될 때마다 **이미 계산된 값을 반환**한다.

### 주의할 점
- 성능을 위해 해시코드 계산할 때 핵심 필드를 생략해서는 안된다.
- hashcode 값의 **생성 규칙을 API 사용자에게 자세히 공표해서는 안된다.**
