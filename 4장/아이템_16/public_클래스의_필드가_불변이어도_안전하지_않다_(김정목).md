# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.

<br>

```java
public class member {
  public String memberEmail;
  public String memberPassword;
}  
```

위 코드를 회원의 이메일과 비밀번호와 같은 회원 정보를 담은 `member` 라는 클래스이다. <br>
`member` 라는 클래스는 `public` 이며 `member` 클래스에서 담고 있는 `memberEmail` 과 `memberPassword` 또한 `public` 이다. <br>
<br>
위와 같이 코드를 작성하면 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다. <br>
<br>
그래서 아래와 같이 데이터 필드를 `public` 에서 `private` 로 변경하고, `public` 접근자(getter)를 추가하여야 한다. <br>

```java
public class member {
  private String memberEmail;
  private String memberPassword;

  public String getMemberEmail() { return memberEmail; }
  public String getMemberPassword() { return memberPassword; }
}  
```
위와 같이 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다. <br>

<br>
하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다. <br>

// 여기서부터 살짝 이해 이슈 
