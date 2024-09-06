# Item13

## clone 재정의는 주의해서 진행하라

### Cloneable 인터페이스의 문제점

`Cloneable`은 메서드가 없는 **마커 인터페이스**이다. clone 메서드는 Object 클래스에 `protected`로 선언되어 있어, `Cloneable`을 구현하는 것만으로는 외부 객체에서 **clone 메서드를 호출할 수 없다.**

### clone 메서드 재정의 시 주의사항

- `public`으로 **접근 제어자를 변경**해야한다.
- `반환 타입`을 **자신의 클래스 타입**으로 변경한다.
- `super.clone()`을 호출하고 **예외를 처리**해야한다.
- 깊은 복사(deep copy)가 필요한 경우 추가 작업이 필요하다.

### 코드 예시

```java
public class DeepCopyExample implements Cloneable {
    private int[] data;

    public DeepCopyExample(int[] values) {
        data = values;
    }

    @Override
    public DeepCopyExample clone() {
        try {
            DeepCopyExample result = (DeepCopyExample) super.clone();
            result.data = data.clone(); // 깊은 복사 수행
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // 일어날 수 없는 일이므로
        }
    }

    public void setDataAt(int index, int value) {
        data[index] = value;
    }

    public int getDataAt(int index) {
        return data[index];
    }

    public static void main(String[] args) {
        DeepCopyExample original = new DeepCopyExample(new int[]{1, 2, 3});
        DeepCopyExample cloned = original.clone();

        System.out.println("Original: " + original.getDataAt(0));
        System.out.println("Cloned: " + cloned.getDataAt(0));

        cloned.setDataAt(0, 100);

        System.out.println("After modification:");
        System.out.println("Original: " + original.getDataAt(0));
        System.out.println("Cloned: " + cloned.getDataAt(0));
    }
}

```

이 예시 코드에서 주목할 점들은 다음과 같다.

1. `DeepCopyExample` 클래스는 **Cloneable 인터페이스**를 구현한다.

2. `clone()` 메서드를 **public으로 오버라이드**하고, 반환 타입을 `DeepCopyExample`로 변경한다.

3. `super.clone()`을 호출하고 `CloneNotSupportedException`을 처리한다.

4. data 배열에 대해 **깊은 복사**를 수행한다. 이는 **원본과 복제본이 서로 다른 배열 인스턴스를 갖도록**한다.

5. main 메서드에서 clone의 동작을 테스트합한다. **복제본의 데이터를 변경해도 원본에 영향을 주지 않는 것**을 확인할 수 있다.

### clone 메서드 재정의의 대안

- 복사 생성자나 **정적 팩터리 메서드**를 사용하는 것이 더 좋은 방법일 수 있다.
- 이들은 더 유연하고, 안전하며, Cloneable 아키텍처의 문제를 피할 수 있다.

### 복사 생성자와 정적 팩터리 메서드

`복사 생성자`와 `정적 팩터리 메서드`를 통해서 `clone()` **메서드의 대안으로 자주 사용**되며, 여러 장점을 가지고 있다.

1. **복사 생성자 (Copy Constructor)**:

복사 생성자는 **같은 클래스의 다른 객체를 매개변수로 받아** 새로운 객체를 생성하는 `생성자`다.

```java
public class Person {
    private String name;
    private int age;
    private List<String> hobbies;

    // 일반 생성자
    public Person(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        this.hobbies = new ArrayList<>(hobbies);
    }

    // 복사 생성자
    public Person(Person other) {
        this.name = other.name;
        this.age = other.age;
        this.hobbies = new ArrayList<>(other.hobbies);
    }

    // getter와 setter 메서드들...

    public static void main(String[] args) {
        Person original = new Person("Alice", 30, Arrays.asList("reading", "swimming"));
        Person copy = new Person(original);

        System.out.println("Original: " + original.getName() + ", " + original.getAge());
        System.out.println("Copy: " + copy.getName() + ", " + copy.getAge());

        copy.setAge(31);
        copy.getHobbies().add("running");

        System.out.println("After modification:");
        System.out.println("Original: " + original.getName() + ", " + original.getAge() + ", " + original.getHobbies());
        System.out.println("Copy: " + copy.getName() + ", " + copy.getAge() + ", " + copy.getHobbies());
    }
}

```

**장점**
- 새로운 객체를 생성하므로 원본 객체의 **불변성을 보장**한다.
- 복사 과정을 명시적으로 제어할 수 있다.
- 다른 클래스의 객체로부터도 복사할 수 있도록 확장 가능하다.


2. **정적 팩터리 메서드 (Static Factory Method)**:

정적 팩터리 메서드는 객체 생성을 **캡슐화하는 정적 메서드**입니다. 복사 기능을 구현할 때도 사용할 수 있다.

```java
public class Employee {
    private String name;
    private int id;
    private Department department;

    public Employee(String name, int id, Department department) {
        this.name = name;
        this.id = id;
        this.department = department;
    }

    // 정적 팩터리 메서드를 이용한 복사
    public static Employee copyOf(Employee original) {
        Employee copy = new Employee(original.name, original.id, new Department(original.department.getName()));
        return copy;
    }

    // getter와 setter 메서드들...

    public static void main(String[] args) {
        Employee original = new Employee("Bob", 1001, new Department("IT"));
        Employee copy = Employee.copyOf(original);

        System.out.println("Original: " + original.getName() + ", " + original.getDepartment().getName());
        System.out.println("Copy: " + copy.getName() + ", " + copy.getDepartment().getName());

        copy.getDepartment().setName("HR");

        System.out.println("After modification:");
        System.out.println("Original: " + original.getName() + ", " + original.getDepartment().getName());
        System.out.println("Copy: " + copy.getName() + ", " + copy.getDepartment().getName());
    }
}

class Department {
    private String name;

    public Department(String name) {
        this.name = name;
    }

    // getter와 setter 메서드들...
}

```

**장점**:
- 메서드 이름을 통해 의도를 명확히 표현할 수 있다 (예: copyOf, newInstance 등).
- 반환 타입의 **하위 타입 객체를 반환할 수 있어 유연**하다.
- 매개변수화 타입 인스턴스를 만들 때 편리하다.

### **clone() 메서드 대신** 이러한 방법들을 사용하는 이유는 다음과 같다.

1. **안전성**: clone() 메서드는 **오버라이드**하기 쉽지만, 잘못 구현하면 **원본 객체의 상태를 훼손**할 수 있다.

2. **설계 유연성**: `복사 생성자`와 `정적 팩터리 메서드`는 **원하는 방식으로 객체를 복사할 수 있는** 더 많은 자유를 제공한다.

3. **명확성**: clone() 메서드의 규약은 모호한 부분이 있지만, 복사 생성자와 정적 팩터리 메서드는 그 의도가 명확하다.

4. **일관성**: 모든 클래스에서 일관된 방식으로 객체를 복사할 수 있다.

5. **예외 처리**: clone() 메서드는 `CloneNotSupportedException`을 던질 수 있어 사용이 불편할 수 있다.

6. **상속 문제 해결**: 복사 생성자나 정적 팩터리 메서드는 상속 관계에서 발생할 수 있는 문제를 더 쉽게 해결할 수 있다.

결론적으로 `복사 생성자`나 `정적 팩터리 메서드`를 사용하여 객체를 복사하면, 원본 객체와 복사된 객체는 **완전히 다른 인스턴스**가 된다.


