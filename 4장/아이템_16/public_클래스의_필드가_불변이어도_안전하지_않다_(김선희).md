# public 클래스에서 가변 필드의 접근 제한자를 public으로 사용하는 경우
: 결론은 안좋다!
```java
class Point {
    public double x;
    public double y;
}
```

**WHY?**
* 캡슐화의 이점을 제공하지 못하고
* API를 수정하지 않고는 내부 표현을 변경할 수 없고
* 불변식을 보장할 수 없다

</br>

## 해결방법
필드를 `private` 으로 변경하고 `public 접근자(메서드)`를 추가한다.
```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

패키지 바깥에서 접근할 수 있는 클래스라면 **접근자를 제공함으로써** 클래스 내부 표현 방식을 언제든 바꿀 수 있다.

</br>

## 🙋‍♀️ 예외가 있다

`package-private` 클래스 혹은 `private` 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.

### 1️⃣ package-private 클래스 경우

```java
package mypackage;

class MyPackageClass {
    String data;

    MyPackageClass(String data) {
        this.data = data;
    }

    public void displayData() {
        MyPackageClass myPackageClass = new MyPackageClass("Hello World");
        System.out.println(myPackageClass.data);  // 데이터 필드에 접근 가능
    }
}
```

</br>

### 2️⃣ private 중첩 클래스 경우
```java
public class OuterClass {
    private class InnerClass {
        String data;

        InnerClass(String data) {
            this.data = data;
        }
    }

    public void displayInnerData() {
        InnerClass inner = new InnerClass("InnerClass");
        System.out.println(inner.data);  // 데이터 필드에 접근 가능
    }
}
```

해당 클래스 안에서 동작하는 코드이므로, **패키지 바깥 코드에서는 손대지 않고도 데이터 표현 방식을 변경할 수 있다**

</br>

## public 클래스의 필드가 불변이라면
: 별로 좋지 않은 생각이다

* API를 변경하지 않고는 표현 방식을 바꿀 수 없다
* 필드를 읽을 때 부수 작업을 수행할 수 없다

</br>

## 🎯 정리
```
1. public 클래스는 절대 가변 필드를 직접 노출하면 안된다
2. package-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 수 있다
```