# Comparable을 구현할지 고려하라.

</br>

이번 아이템에서는 `Comparable` 인터페이스의 `compareTo` 메서드를 다룬다. (`Object`의 메서드 아님)

`compareTo`는 `equals`와 비슷한데 약간의 차이가 있다.

</br>

`compareTo`는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.

`Comparable`을 구현했다는 것은 인스턴스들에 ***자연적인 순서*** 가 있음을 뜻한다. 그래서 다음과 같이 정렬할 수 있다.

```java
Arrays.sort(a);
```

</br>

이 외에도 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 역시 쉽게 할 수 있다.

> 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 `Comparable` 인터페이스를 구현하자.

</br>

</br>
 

## compareTo 메서드 규약

</br>

> 우선 `compareTo`는 타입이 타른 객체를 신경 쓰지 않아도 된다. 타입이 다르면 `ClassCastException`을 던지면 된다.
> `hashCode` 규약을 지키지 못하면 해시를 사용하는 클래스와 어울리지 못하듯, `compareTo` 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다.
> 비교를 활용하는 클래스의 예로는 정렬된 컬렉션인 `TreeSet`과 `TreeMap`, 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 `Collections`와 `Arrays`가 있다.

</br>
 
 1. 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
 2. 첫 번째가 두 번째 보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야한다.
 3. 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
    - `compareTo` 메서드로 수행한 동치성 테스트의 결과가 `equals`와 같아야 한다는 의미

</br>

세 규약은 ***반사성***, ***대칭성***, ***추이성*** 을 충족해야 함을 뜻한다.

</br>

</br>

## compareTo 메서드 작성 요령

</br>

`compareTo` 메서드 작성 요령은 `equals`와 비슷하나, 몇 가지 차이점을 주의해야 한다.

</br>

1. `Comparable`은 타입을 인수로 받는 제네릭 인터페이스이므로 `compareTo` 메서드의 인수 타입은 컴파일타임에 정해진다. (입력 인수의 타입을 확인할 필요 X)
2. `compareTo` 메서드는 각 필드가 동치인지를 비교하는 게 아니라 그 순서를 비교한다.
    - `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 `비교자(Comparator)`를 대신 사용한다.
3. 클래스에 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교해나가자.

</br>

> 자바 7부터 변경되어 `compareTo` 메서드에서 관계 연산자 `<`와 `>`를 사용하는 방식은 추천하지 않는다!

</br>


자바 8에서는 `Comparator` 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.

이 방식은 간결하나, 약간의 성능 저하가 뒤따른다.

자바의 `정적 임포트` 기능을 이용하면 정적 비교자 생성 메서드들은 그 이름만으로 사용할 수 있어 코드가 훨씬 깔끔해진다.

</br>

***‘값의 차’*** 를 기준으로  첫 번째 값이 두 번째 값보다 작으면 음수를, 같으면 0을, 첫 번쨰 값이 크면 양수를 반환하는 `compareTo`나 `compare` 메서드와 마주하게 될 것이다.

</br>

아래 코드를 보자.

```java
// 코드 14-4 해시코드의 값의 차를 기준으로 하는 비교자 - 추이성을 위배한다!
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return o1.hashCode() - o2.hashCode();
  }
};
```

결론부터 말하면, 이 방식은 사용하면 안 된다.

정수 오버플로를 일으키거나 IEEE 754 부동소수점 계산 방식에 따른 오류를 낼 수 있다.

</br>

대신 아래의 두 방식을 사용하자.

</br>

</br>

### 정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return Integer.compare(o1.hashCode(), o2.hashCode());
  }
};
```

</br>

</br>

### 비교자 생성 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = 
  Comparator.comparingInt(o -> o.hashCode());
```

</br>

</br>

## 핵심 정리

순서를 고려해야 하는 값 클래스를 작성한다면 꼭 `Comparable` 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다.

`compareTo` 메서드에서 필드의 값을 비교할 때 `<` 와 `>` 연산자는 쓰지 말아야 한다.

그 대신 박싱된 기본 타입 클래스가 제공하는 정적 `compare` 메서드나 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
