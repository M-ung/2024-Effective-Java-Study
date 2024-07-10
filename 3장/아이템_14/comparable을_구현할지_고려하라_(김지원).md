# Comparable을 구현할지 고려하라

</br>

## Comparable?
- Comparable 인터페이스는 객체를 정렬하는데 사용되는 메서드인 compareTo 를 정의하고 있다.
- Comparable 인터페이스를 구현한 클래스는 반드시 compareTo 를 정의해야 한다.

### 특징
- 자바에서 같은 타입의 인스턴스를 비교해야만 하는 클래스들은 모두 `Comparable` 인터페이스를 구현하고 있다.
- Boolean 타입을 제외한 래퍼 클래스와 알파벳 , 연대같이 순서가 명확한 클래스들은 모두 정렬이 가능하다.
- 기본 정렬 순서는 작은 값에서 큰 값으로 정렬되는 `오름차순`이다.
  
</br>

### 구현

```java
public class Car implements Comparable<Car> {
    private static final String SPACE = " ";
    private static final String MODEL_YEAR = " 식 ";

    private String modelName;
    private int modelYear;
    private String color;

    public Car(final String modelName, final int modelYear, final String color) {
        this.modelName = modelName;
        this.modelYear = modelYear;
        this.color = color;
    }

    @Override
    public String toString() {
        return this.modelYear + MODEL_YEAR + this.modelName + SPACE + this.color;
    }

    @Override
    public int compareTo(Car car) {
        return Integer.compare(this.modelYear, car.modelYear);
    }
}

public class Main {
    public static void main(String[] args) {
        Car car1 = new Car("그렌저", 2016, "검정색");
        Car car2 = new Car("쏘나타", 2020, "흰색");

        System.out.println(car1.compareTo(car2));
    }
}
```
- Comparable 을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 의미한다.

</br>

## Comparable 의 compareTo 메서드
> compareTo 는 해당 객체와 전달된 객체의 순서를 비교한다.
- compareTo 는 Object 의 equals 와 두가지 차이점이 있다.
  - compareTo 는 equals 와 달리 단순 동치성에 더해 순서까지 비교할 수 있으며, 제네릭하다.
- Comparable 을 구현한 객체들의 배열은 Arrays.sort(a) 로 쉽게 정렬이 가능하다.

</br>

### CompreTo 메서드 일반 규약
> 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 `음의 정수` 를 같으면 `0` 을 크면 `양의 정수` 를 반환한다. 비교할 수 없는 타입이 주어지면 `ClasscastExcpetion` 을 던진다.
- sgn 은 부호 함수를 뜻하며, 표현식의 값이 음수 , 0 , 양수 일 때 -1 , 0 , 1 을 반환하도록 정의
- 만약 기존 클래스를 확장한 구체 클래스가 값 필드를 추가했다면 compareTo 규약을 준수할 방법이 없기 떄문에 이런 경우 조합을 이용해 우회적으로 규약을 준수한다.
  - `Parent` 클래스에 name 필드만 있고 이를 비교하는 `compareTo` 메서드가 있다고 가정
  - `Child` 클래스가 `Parent` 클래스를 상속 받고 age 필드를 추가로 가지게 되면 name 뿐만 아니라 age 도 비교해야 하기 때문에 compareTo 규약을 준수하기 어렵다.
  - 이럴 경우에는 `상속` 을 사용하지 않고 기존 클래스의 객체를 새로운 클래스의 필드로 포함하게 되면 compareTo 규약을 준수하면서 새로운 값을 비교할 수 있다.

</br>

1. 대칭성
- 두 객체참조의 순서를 바꿔 비교해도 예상한 결과가 나와야한다.
- sgn(x.compareTo(y)) == -sgn(y.comapreTo(x)) 여야 한다.
- x.compareTo(y) 는 y.compareTo(x) 가 예외를 던질때에 한해 예외를 던져야 한다.


2. 추이성
- 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면 첫 번째는 세 번째보다 커야 한다.

3. 반사성
- 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같다.
- Comparable 을 구현한 클래스는 모든 z 에 대해 x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 이다.

4. equals
- compreTo 메서드로 수행한 동치성테스트 결과가 equals 와 같아야 한다.
- (x.compareTo(y) == 0) == (x.equals(y)) 여야 한다.
- Comparable 을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다.

</br>

## equals 와 compareTo 의 차이

```java
...
		final BigDecimal bigDecimal1 = new BigDecimal("1.0");
        final BigDecimal bigDecimal2 = new BigDecimal("1.00");

        final HashSet<BigDecimal> hashSet = new HashSet<>();
        hashSet.add(bigDecimal1);
        hashSet.add(bigDecimal2);

        System.out.println(hashSet.size());

        final TreeSet<BigDecimal> treeSet = new TreeSet<>();
        treeSet.add(bigDecimal1);
        treeSet.add(bigDecimal2);

        System.out.println(treeSet.size());
...
// 실행결과 
hashSet: 2
treeSet: 1
```
- `HashSet` 과 `TreeSet` 은 서로 다른 메서드로 객체의 동치성을 비교한다.
- `HashSet` 은 `equals` 를 기반으로 비교하기 때문에 추가된 두 BigDecimal 이 다른 값으로 인식되어 크기가 `2`가 된다.
- `Treeset` 은 `compareTo` 를 기반으로 객체에 대한 동치성을 비교하기 때문에 같은 값으로 인식되어 compareTo 가 `0` 으로 반환하기 떄문에 크기가 1 이 된다.

</br>

## CompareTo 메서드 작성 요령
- `Comparable` 은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 의 인수타입은 컴파일시에 정해지기 때문에 입력 인수 확인이나 형변환을 할 필요가 없다.
- `null` 을 인수로 넣으면 `NPE` 을 던져야 한다.
- `compareTo` 는 동치가 아닌 순서를 비교 한다.
- 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출한다.
- `Comparable` 을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 `Comparator` 을 대신 사용한다.

</br>

### compareTo 메서드에 관계연산자 (<,>) 를 사용하지 않는 것을 추천한다.
- 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드 `compare` 를 대신 이용한다.
- 관계연산자(<.>) 는 오류를 유발할 가능성이 있기 떄문에 추천하지 않는다.

</br>

클래스에 핵심 필드 여러개일 때 비교
> 클래스에 핵심 필드가 여러개라면 가장 핵심적인 필드부터 비교한다.
- 비교 결과가 `0` 이 아니라면, 즉 순서가 결정되면 바로 결과를 반환하면 된다.
- 똑같지 않은 필드를 찾을 때 까지 비교해나가도록 구현하면 된다.

</br>

## Comparator?
- 자바 8에서 `Comparator` 이넡페이스가 일련의 비교자 생성 메서드와 메서드 연쇄방식으로 비교자를 생성할 수 있게 됐다.
- 비교자들은 `comparTo` 메서드를 구현하는데 활용될 수 있다.
  - 이 방식은 간결하지만 약간의 성능 저하가 뒤따를 수 있다.
  - 자바의 static import 를 활용하면 코드가 훨씬 깔끔해진다.
- `Comparator` 는 `comparingInt` 와 `thenComparingInt` 등의 숫자용 기본 타입을 커버하는 보조 생성 메서드들을 가지고 있다.
- `comparing` 와 `thenComparing` 이란 객체 참조용 비교자 생성 메서드 또한 가지고 있다.

</br>

```java
private static final Comparator<PhoneNumber> COMPARATOR =
            Comparator.comparingInt((PhoneNumber phoneNumber) -> phoneNumber.areaCode)
                    .thenComparingInt(phoneNumber -> phoneNumber.prefix)
                    .thenComparingInt(phoneNumber -> phoneNumber.lineNum);

public int compareTo(PhoneNumber phoneNumber) {
	    return COMPARATOR.compare(this, phoneNumber);
}

```
- 위 코드는 비교자 생성 메서드 2개를 이용해 비교자를 생성한다.
  - 첫 번째는 `comparingInt`
  - 두 번째는 `thenCompaingInt`
- `comparingInt` 는 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메소드
- `thenComparingInt` 는 `Comparator` 의 인스턴스 메서드로, int 키 추출 함수를 입력 받아 다시 비교자를 반환한다.
- `comparingInt` 는 첫 번째 비교기를 생성하여 areaCode 를 비교하고 같다면 다음 로직을 넘어간다.
- `thenComparingInt` 는 두 번쨰 비교기와 세 번쨰 비교기를 생성하고 prefix 필드를 먼저 비교하고 같다면 lineNum 필드를 비교하도록 한다.
- 모든 필드가 같다면 `0` 을 반환한다.

</br>

### 추이성을 위반 (해시코드 값의 차를 기준으로 하는 비교자)

```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return o1.hashCode() - o2.hashCode();
        }
};
```
- 이 방식은 정수 오버플로를 일으키거나 부동소수점 계산방식에 따른 오류를 발생시킬 수 있어 사용하지 말아야 한다.
- 아래의 두 방식을 대신 사용하도록 한다.

</br>

### 해결 방법 1 (정적 compare 메서드 사용)

```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
        @Override
        public int compare(final Object o1, final Object o2) {
            return Integer.compare(o1.hashCode(), o2.hashCode());
        }
};
```

</br>

### 해결 방법 2 (Comparator 사용)

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

</br>

## 결론

</br>

순서를 고려해야하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션이 되도록 한다.

`comparTo` 메서드에서 필드의 값을 비교할 때 `<` 와 `>` 연산자는 지양해야 한다.
- 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 가 제공하는 비교자 생성 메서드를 이용한다.
