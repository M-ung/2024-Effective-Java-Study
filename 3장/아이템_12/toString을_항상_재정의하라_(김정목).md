# toString을 항상 재정의하라.
Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다. <br>
toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다. <br>

<br>

toString을 재정의하지 않으면 Object의 기본 toString 메서드를 사용하게 된다. <br>
Object의 기본 toString은 `클래스이름@16진수로_표시한_해시코드` 가 출력된다. <br>

```java
System.out.println(phoneNumber); //PhoneNumber@adbbd
```
하지만 보통의 개발자들이 toString 메서드에게 기대하는 것은 `707-867-5309` 같은 그 클래스의 필드 값일 것이다. 따라서 toString은 모든 구체클래스에서 재정의되어야 한다.

아래와 같이 재정의하면 된다. <br>

```java
@Override public String toString() {
    return String.format("%03d-%03d-%04d",
            areaCode, prefix, lineNum);
}
```

### toString 생성 시 주의할 점
1. 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.
2. 포맷을 명시하든 아니든 여러분의 의도는 명확히 밝혀야 한다. <br>
값 클래스의 경우에는 더욱 포맷 명시, 명시한 포맷에 맞는 객체 생성 코드를 작성해야 한다.
3. toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공해야 한다.


## ⭐️ 핵심 정리
모든 구체 클래스에서 Object의 toString을 재정의해야 한다. 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다. toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다. toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.
