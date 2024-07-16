# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.

</br>

우선 아래의 코드를 먼저 보자.

```java
// 책에는 public이 아니지만, 확실한 비교를 위해 public을 붙였다.
public class Point {
    public double x;
    public double y;
}
```

</br>

### 이 코드의 문제점이 뭘까?

</br>

1. 데이터 필드에 직접 접근 가능해 `캡슐화의 이점`을 제공하지 못한다.
2. API를 수정하지 않고는 `내부 표현`을 바꿀 수 없다.
3. `불변식`을 보장할 수 없다.
4. 외부에서 필드에 접근할 때 `부수 작업`을 수행할 수 없다.

</br>

</br>

```java
public class Point {
    private double x;
    private double y;
	
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
	
    public double getX() { return x; }
    public double getY() { return y; }
	
    public void setX(double x) { this.x = x; }
    public void sety(double y) { this.y = y; }
}
```

그러므로 우리는 위의 코드와 같이 필드를 `private`으로 변경하고, `getter`와 `setter`를 추가해야한다.

> 개발 시 `setter`를 열어두는 것은 좋지 않다.

</br>

</br>

## package-private 클래스나 private 중첩 클래스에서는 데이터 필드를 노출한다 해도 문제가 되지는 않는다.

</br>

```java
// package-private 클래스에서의 데이터 필드 노출
class PackagePrivateClass {
    public int exposedField;

    PackagePrivateClass(int value) {
        this.exposedField = value;
    }
}

// private 중첩 클래스에서의 데이터 필드 노출
public class OuterClass {
    private static class innerClass {
        public int exposedField;

        innerClass(int value) {
            this.exposedField = value;
        }
    }
}
```

왜 이러한 클래스에서는 안전할까?

</br>

- `package-private 클래스`는 **동일한 패키지** 내에서만 접근 가능하기 때문에 외부에서 해당 필드에 접근 불가
- `private 중첩 클래스`는 **Outer클래스** 내부에서만 접근이 가능하기 때문에 외부에서는 해당 필드에 접근 불가

</br>

</br>

## public 클래스의 필드가 불변이어도 안전하지 않다.

</br>

```java
public class Point {
    public final double x;
    public final double y;
	  
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
}
```

위의 코드와 같이 구현한다면 불변식을 보장할 수 있다는 점 외에는 필드를 `public`으로 두는 것과 동일한 단점을 가진다.

</br>

</br>

## 핵심 정리

</br>

`public` 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 
불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
하지만 `package-private 클래스`나 `private 중첩 클래스`에서는 종종 필드를 노출하는 편이 나을 때도 있다.
