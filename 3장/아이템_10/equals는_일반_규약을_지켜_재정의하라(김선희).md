# equals 메서드 재정의를 피해야 할 경우
equals 메서드는 재정의하기 쉬워보이지만 함정이 많아 자칫 위험한 결과를 불러일으킬 수 있다.

재정의를 피해야 하는 경우를 정리해보자

## 1️⃣ 각 인스턴스가 본질적으로 고유하다
인스턴스가 고유할 때

```java
public static void main(String[] args) {
    Thread thread1 = new Thread(new Task("Task1"));
    Thread thread2 = new Thread(new Task("Task2"));

    // 두 개의 서로 다른 스레드를 비교합니다.
    System.out.println(thread1.equals(thread2)); // false 출력 -> 값이 다르므로

    // 각 스레드는 고유하므로 equals 메서드를 재정의하지 않습니다.
}
```

Thread 객체인 `thread1`과 `thread2`는 고유한 작업을 수행하는 스레드 이므로 독립적으로 실행된다. 따라서 `두 스레드를 동일한 것으로 간주하는 것은 의미가 없다`.

## 2️⃣ 인스턴스의 '논리적 동치성'을 검사할 일이 었을 때
해당 클래스의 객체들이 `논리적으로 동일한지` 비교할 필요가 없다는 것을 의미합니다.

```java
public static void main(String[] args) {
    Logger logger1 = new Logger("Logger1");
    Logger logger2 = new Logger("Logger2");

    // 각 Logger 객체는 고유한 로깅 인스턴스를 나타내며,
    // 논리적 동치성을 비교할 필요가 없습니다.
    logger1.log("This is a log message from logger1");
    logger2.log("This is a log message from logger2");

    // equals 메서드를 재정의하지 않으므로,
    // logger1과 logger2는 기본적으로 동일하지 않다고 간주됩니다.
    System.out.println(logger1.equals(logger2)); // false 출력
}
```
Logger 클래스는 로그 메시지를 기록하는 기능이 중요하지 `객체가 논리적으로 동일한지`는 중요하지 않습니다.

## 3️⃣ 상위 클래스에서 재정의한 equals 가 하위 클래스에서도 적절할 때
대부분의 **Set** 구현체는 `AbstractSet 이 구현한 equals 를 상속받아` 사용하고, **List** 구현체들은 `AbstractList 로부터`, **Map** 구현체들은 `AbstractMap 으로부터 상속받아` 사용한다.

## 4️⃣ 클래스가 private 이거나 package-private 이고 equals 메서드를 호출할 일이 없을 때
아래와 같이 예외처리를 던지자

```java
@Override
pubilc boolean equals(Object o) {
    throw new AssertionError();
}
```

# equals 를 재정의 해야하는 경우
## 1️⃣ 상위 클래스에서 equals가 논리적 동치성을 비교하도록 재정의 되어있지 않는 경우
주로 값 클래스들이 해당한다(String, Integer)

여기서 예외적인 경우는, 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스이거나 Enum 클래스 라면 equals 를 재정의하지 않아도 된다.

```
Why? 어차피 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으므로
```

## equals 재정의시 지켜야 하는 컨벤션
* `반사성` : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true 이다
* `대칭성` : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true 이면 y.equals(x)도 true 이다

```java
// 대칭성 위배
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) 
            return s.equalsIgnoreCase((CaseInsensitiveString) o).s;

        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
        
        return false;
    }
}
```

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

여기서 cis.equals(s)는 true 를 반환한다.
`CaseInsensitiveString 의 equals` 는 `String 의 존재를 알고 있지만`

여기서 s.equals(cis)는 false 를 반환한다.
`String 의 equals`는 `CaseInsensitiveString 의 존재를 모르고 있으므로`

따라서 대칭성의 규칙을 명백히 위반한다.

또한 **equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응하는지 알 수 없다.**

그래서 equals 는 아래와 같이 변경할 수 있다.

```java
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

* `추이성` : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true 이고 y.equals(z)도 true 면 x.equals(z)도 true 이다.

추이성은 `첫 번째 객체 == 두 번째 객체`, `두 번째 객체 == 세 번째 객체` 라면 `첫 번째 객체 == 세 번째 객체`이다.

```java
public class Point {
    private final int x;
    private final int y;

    public Point(final int x, final int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(final int x, final int y, final Color color) {
        super(x, y);
        this.color = color;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) { // 비교하는 객체가 ColorPoint의 인스턴스가 아니라면 false를 반환한다. 대칭성 위배!!
            return false;
        }
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
    
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) { // o가 Point의 인스턴스가 아닐 때
            return false;
        }
        if (!(o instanceof ColorPoint)) { // o가 Point의 인스턴스지만 ColorPoint의 인스턴스는 아닐 때
            return o.equals(this);
        }
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

위 코드의 문제점은 `equals 규약을 어긴 것은 아니지만 중요한 정보(color)를 놓치게 되니` 올바르지 않은 equals 이다.

그럼 색상을 무시하고 비교하면 어떻게 될까?

```java
@Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) { // o가 Point의 인스턴스가 아닐 때
            return false;
        }
        if (!(o instanceof ColorPoint)) { // o가 Point의 인스턴스지만 ColorPoint의 인스턴스는 아닐 때
            return o.equals(this);
        }
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

이렇게 되면 `무한 재귀`에 빠질 위험이 있다.

### 애초에 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

```java
@Override
public boolean equals(Object o) {
    if (o == null || o.getClass != getClass()) {
        return false;
    }
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

위 코드의 문제점은 `리스코프 치환 원칙에 위배`된다는 점이다.

* `일관성` : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true 를 반환하거나 false를 반환한다
* `null-아님` : null이 아닌 참조 값 x에 대해, x.equals(null)은 false 이다.

