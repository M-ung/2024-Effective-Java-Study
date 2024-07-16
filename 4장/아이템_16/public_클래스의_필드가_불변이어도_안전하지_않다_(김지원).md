# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.

</br>

### 예시 코드
```java
class Point {
    public double x;
    public double y;
}
```

</br>

### public 클래스에 public 필드를 사용했을 때 문제점은?
- `불변식` 보장
- API를 수정하지 않고는 `내부 표현`을 바꿀 수 없다.
> 여기서 API를 수정하지 않고서 내부 표현을 바꿀 수 없다는 말이 무슨말일까.. (아래 코드 참고)

</br>


```java
public class A {
  public int value;
  public Person(int value) {
    this.value = value;
  }
}

public class B {
  public void doSomething(A a) {
    System.out.println(a.value);
  }
}
```
- 만약 여기서 A 필드가 private 로 바꾸게 된다면 어떻게 될까?
  - B.doSomething 에서 a.value 사용에서 컴파일 오류가 발생한다.
- 즉 이렇게 public 필드를 수정할 경우에 A , B 모두 수정을 해야하는 상황을 말하는 것

</br>

### 해결 방법
- 위에서 말한 문제점을 해결하기 위해서는 private 으로 바꾸고 public 접근자를 추가하기


```java
public class A {
  private int value;
  public Person(int value) {
    this.value = value;
  }

  public int getValue() { return value; }
  public void setValue() { this.value = value; }
}

public class B {
  public void doSomething(A a) {
    System.out.println(a.getValue());
  }
}
```
- 이렇게 작성하게 되면 A 의 내부 표현을 마음대로 바꿔도 상관 없다.
- 이전에는 클라이언트가 필드를 직접 참조하기 때문에 내부 표현을 바꾸기가 어렵지만 지금은 사용하지 않기 떄문에 쉬워진다.

</br>

### 불변 객체를 참조하는 public 필드?
- 직접 노추릿 단점은 조금 줄어들 수 있지만 API를 변경하지 않고는 표현 방식을 바꿀 수 없다는 단점은 계속 가지게 된다.
- 불변식만을 보장할 수 있다는 장점은 있지만 좋은 생각이 아니기 때문에 사용을 지양하자.

## 핵심 정리

</br>

`public` 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 
불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
하지만 `package-private 클래스`나 `private 중첩 클래스`에서는 종종 필드를 노출하는 편이 나을 때도 있다.
