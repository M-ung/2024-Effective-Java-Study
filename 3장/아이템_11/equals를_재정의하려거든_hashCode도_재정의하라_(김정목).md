# equals를 재정의하려거든 hashCode도 재정의하라. 
</br>
equals를 재정의한 클래스 모두에게 hashCode도 재정의해야 한다. 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

<br>

### Object 명세에서 발췌한 규약
1. equals 비교에 사용되는 정보가 변경되지 않는다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 사관없다.
2. equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
3. equals(Object)가 두 객체를 다르다고 판단했다라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

<br>

hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째이다. -> 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다. <br>

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

m.get(new PhoneNumber(707, 867, 5309));
```

위 코드를 실행하면 "제니"를 반환하는 것이 아닌, null을 반환하게 된다. <br>
이는 PhoneNumber 클래스가 hashCode를 재정의하지 않았기 떄문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못 한 것이다. 그 결과 get 메서드는 엉뚱한 해시 버킷에 가서 객체를 찾으려 한 것이다. <br>
위 문제는 PhoneNumber에 적절한 hashCode 메서드만 작성해주면 해결된다. <ㅠr> 

```java
@Override public int hashCode() {return 42;}
```
위 코드는 적법한 코드지만, 최악의 hashCode를 구현한 코드이다. <br> 
이 코드는 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적합하다. 하지만 끔찍하게도 모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결 리스트처럼 동작한다. <br>
-> 그 결과 평균 수행 시간이 O(1)인 해시테이블이 O(n)으로 느려져서, 객체가 많아지면 도저히 쓸 수 없게 된다. <br>
</br>

### 좋은 hashCode를 작성하는 간단한 요령
1. int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫 번쨰 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다.<br>
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.<br>
&nbsp;a. 해당 필드의 해시코드 c를 계산한다.<br>
&nbsp;&nbsp;&nbsp;- 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.<br>
&nbsp;&nbsp;&nbsp;- 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다. <br>
&nbsp;&nbsp;&nbsp;- 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 귳픽을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수를 사용한다. 모든 원소가 핵심 원소라면 Array.hashCode를 사용한다. <br>
&nbsp;b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다. "result = 31 * result + c;"<br>
3. result를 반환한다.<br>
<br>
<br>

### 전형적인 hashCode 메서드

```java
    @Override public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
```
</br>

이 메서드는 PhoneNumber 인스턴스의 핵심 필드 3개만을 사용해 간단한 계산만 수행한다. 그 과정에 비결정적 요소는 전혀 없으므로 동치인 PhoneNumber 인스턴스들은 같은 해시코드를 가질 것이 확실하다.<br>

"단, 해시 충돌이 더욱 적은 방법을 꼭 써야 한다면 구아바의 `com.google.common.hash.Hasing`을 참고하자!"
</br>
</br>

### 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다.
```java
    @Override public int hashCode() {
        return Objects.hash(lineNum, prefix, areaCode);
    }
```
</br>

위 코드는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공한다. 이 메서드를 활용하면 앞서의 요령대로 구현한 코드와 비슷한 수준의 hashCode 함수를 단 한 줄로 작성할 수 있다. <br>
하지만, 아쉽게도 속도가 더 느리다.

<br>

클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기보다는 캐싱하는 방식을 고려해야 한다. <br>

### 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다.
```java
    private int hashCode; // 자동으로 0으로 초기화된다.

    @Override public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
```
</br>

성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다. 속도야 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다. <br>
hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말아야 한다. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다. <br>
<br>
## ⭐️ 핵심 정리
equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다. 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다. <br>
AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어준다.
