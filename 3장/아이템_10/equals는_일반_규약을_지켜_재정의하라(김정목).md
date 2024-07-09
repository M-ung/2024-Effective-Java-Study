# equals는 일반 규약을 지켜 재정의하라.
equals 메서드는 재정의하기 쉬워 보이지만 자칫하면 끔찍한 결과를 초래한다. 그렇기 때문에 재정의하지 않는 것이 가장 좋은 방법이다. <br><br>

1️⃣ 각 인스턴스가 본질적으로 고유하다. <br>
-> 각 인스턴스가 본질적으로 고유하다. <br>
2️⃣ 인스턴스의 '논리적 동치성' 을 검사할 일이 없다. <br>
-> equals를 재정의해서 논리적 동치성을 검사하는 방법이 있다고 쳐도 설계자는 클라이언트가 이 방식을 원하지 않거나 애초에 필요하지 않다고 판단할 수 있다. <br>
3️⃣ 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다. <br>
-> 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현체들은 AbstractList로부터, Map 구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다. <br>
4️⃣ 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다. <br>
<br>

그렇다면 equals를 재정의해야 할 때는 언제일까? <br>
<br>
바로 "객체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때" 이다. <br>
두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것이다. 이럴 때 equals가 논리적 동치성을 확인하돌고 재정의해두면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응할 수 있다. <br>
<br>

equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다. 아래는 Object 명세에 적힌 규약이다. <br>
- 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
- 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
- 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

<br><br>

그렇다면 Object 명세에서 말하는 동치관계란 무엇일까? <br>
동치 관계를 만족시키기 위한 다섯 요건을 하나씩 살펴보자.<br>
- 반사성: 객체는 자기 자신과 같아야 한다는 뜻이다.
- 대칭성: 두 객체는 서로에 대한 동치 여부에 똑같이 대답해야 한다는 뜻이다.

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  
            return s.equalsIgnoreCase((String) o);
        return false;
    }
```

위 코드를 기반으로 아래 코드를 작성해 보면, 

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

cis.equals(s) 는 true를 반환한다. 하짐나 s.equals(cis)는 false를 반환하며 대칭성을 위반한다. <br>

위와 같은 문제를 해결하기 위해서는 CaseInsensitiveString의 equals를 String과도 연동하겠다는 허황한 꿈을 버려야 한다. 그렇다면 아래와 같은 코드가 나온다. <br>

```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```
<br>

- 추이성 : 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다. <br>
```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
```

위 코드를 확장해서 점에 색상을 더해보았다.

</br> 

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
```

이때 그대로 둔다면 색상 정보는 무시한 채 비교를 수행한다.  <br>

```java
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```

위 코드는 일반 Point를 ColorPoint에 비교한 결과와 그 둘을 바꿔 비교한 결과가 다를 수 있다. <br>
위 코드는 Point의 equals는 색상을 무시하고, ColorPoint의 equals는 입력 매개변수의 클래스 종류가 달라 매번 false만 반활할 것이다. <br>

<br>

그렇다면 계속 색상을 무시하도록 하면 해결이 될까? 라는 의문점이 든다. <br>

```java
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        if (!(o instanceof ColorPoint))
            return o.equals(this);

        return super.equals(o) && ((ColorPoint) o).color == color;
    }
```
위 코드는 대칭성은 지키지만, 추이성을 깨버린다. <br>
또 이 방식은 무한 재귀에 빠질 위험도 있다. <br>

그렇다면 해법은 무엇일까? <br>
구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다....<br>
-> 이 말은 얼핏, equals 안의 instanceof 검사를 getClass 검사로 바꾸면 규약도 지키고 값도 추가하면서 구체 클래스를 상속할 수 있다는 뜻으로 들린다. <br>
<br>

```java
    @Override public boolean equals(Object o) {
        if (o == null || o.getClass() != getClass())
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
```

위 코드 같은 경우는 같은 구현 클래스의 객체와 비교할 때만 true를 반환한다. 괜찮아 보이지만 실제로는 활용할 수 없다. <br>
이는 Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로써 활용될 수 있어야 하는데 이 방식에서는 그렇지 못 한다. <br>
<br>
구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 하나 있다. <br>
"상속 대신 컴포지션을 사용하는 것이다." <br>

```java
public class ColorPoint{
    private final Point point;

    private final Color color;

    public ColorPoint(int x, int y, Color color){
        if(color == null)
          throw new NullPointerException();
        point = new Point(x,y);
        this.color = color;
    }

    public Point asPoint(){ 
        return point;
    }

    @Override public boolean equals(Object o){
        if (!(o instanceof ColorPoint))
            return false; 
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color); 
    }
}
```

위 코드는 Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두고, ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰 메서드를 public으로 추가하는 식이다. <br>
<br>
- 일관성 : 두 객체가 같다면 앞으로도 영원히 같아야 한다는 뜻이다. <br>

클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다. <br>

- null-아님 : 모든 객체가 null과 같지 않아야 한다는 뜻이다.

<br>

<br>

### 지금까지 한 내용들을 모두 종합해서 양질의 equals 메서드 구현 방법을 단계별로 정리
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 "핵심" 필드들이 모두 일치하는지 하나씩 검사한다.

<br>
아래는 전형적인 equals 메서드의 예이다. <br>


```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    // 나머지 코드는 생략
}
```

### 마지막 주의 사항
1. equals를 재정의 할 땐 hashCode도 반드시 재정의를 해야 한다.
2. 너무 복잡하게 해결하려 들지 말아야 한다. -> 필드들의 동치성만 검사해도 equals 규약을 어렵지 않게 지킬 수 있다.
3. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말아야 한다.

### ⭐️ 핵심 정리
꼭 필요한 경우가 아니면 equals를 재정의하지 말아야 한다. 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다. 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.
