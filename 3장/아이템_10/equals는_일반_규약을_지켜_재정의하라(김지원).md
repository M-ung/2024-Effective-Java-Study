# equals는 일반 규약을 지켜 재정의하라.

</br> 
 

## equlas 를 재정의 하면 안 되는 경우

1. 각 인스턴스가 본질적으로 고유할 때
  - 값 클래스 (Integer , String ...) 가 아닌 동작하는 개체를 표현하는 클래스
  - ex) `Thread`
</br> 

2. 인스턴스의 논리적 동치성를 검사할 일이 없을 때
  - 정규표현식을 비교하는 java.util.regex.Pattern 과 같은 논리적 동치성을 검사하는 것이 아니면 `Object`의 기본 `equals`만으로 해결

</br> 

3. 상위 클래스에서 재정의한 `equals`가 하위 클래스에도 딱 들어맞을 때
  - ex) Set 은 AbstractSet 이 구현한 equals 를 상속 , List 는 AbstractList , Map 은 AbstarctMap 

</br> 

4. 클래스가 `private`이거나 `package-private`이고 `equals` 메서드를 호출할 일이 없을 때
  - 아래와 같이 구현해 `equals` 가 실수라도 호출되는 걸 막을 수 있다.

```java
@Override public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```

</br> 

## equals 를 재정의 해야 하는 경우

</br> 

- 객체 식별성(두 객체가 물리적으로 같은가?)이 아닌 `논리적 동치성` 을 확인해야하는데, 상위 클래스의 `equals` 가 논리적 동치성을 비교하도록 재정의 되지 않았을때
  - 두 값 객체를 `equals` 로 비교하는 경우, 객체가 같은지가 아니라 값이 같은지를 알고 싶을 것
  - `equals` 가 논리적 동치성을 확인하도록 재정의하면, 값 비교는 물론 Map 의 키와 Set 의 원소로 사용할 수 있다.
  - 하지만, 값 클래스여도 같은 인스턴스가 둘 이상 만들어지지 않는 인스턴스 통제 클래스라면 재정의하지 않아도 된다.
 
</br> 

## equals 메서드 재정의 일반 규약 : 동치 관계
- 동치 클래스 : 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산
  - `equals` 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 교환이 가능해야한다.
  - `equals` 메서드가 제대로 정의되어 있으면, 동치 클래스 내의 모든 객체들은 서로 동일한 것으로 취급될 수 있으며, 프로그램 내의 로직에서 이 객체들을 교환해도 문제가 없어야 한다는 것

</br> 

1. 반사성 : null 이 아닌 모든 참조 값 x 에 대해 x.equals(x) 는 true 이다.

</br> 


2. 대칭성 : null 이 아닌 모든 참조 값 x,y 에 대해 x.equals(y) 가 true 이면 y.equals(x) 도 true 이다.
```java
// 대칭성을 위반한 클래스
public final class CaseInsensitiveString{
  private final String s;

  public CaseInsensitiveString(String s){
    this.s = Obejcts.requireNonNull(s);
  }

  // 대칭성 위배!
  @Override public boolean equals(Object o){
    if(o instanceof CaseInsensitiveString)
      return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    if(o instanceof String) // 한방향으로만 작동한다.
      return s.equalsIgnoreCase((String) o);
    return false;
  }
}

// 대칭성을 만족하게 수정
@Override public boolean equals(Object o){
  return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s); 
}
```
</br> 

- 대칭성을 위반한 이유?
- CaseInsensitiveString의 equals는 String을 알고 있지만, String의 equals는 CaseInsensitiveString의 존재를 모른다는 데 있다.
- 대칭성을 명백히 위반

</br> 


3. 추이성 : null 이 아닌 모든 참조 값 x,y,z 에 대해 x.equals(y) 가 true 이고, y.equals(z) 도 true 면 x.equals(z) 도 true 이다.
   - 아래 코드는 상위 클래스에 없는 새로운 필드를 하위 클래스에 추가하여 equals 를 재정의할 때 자주 발생하는 문제

</br> 


```java
public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override public boolean equals(Object o) {
		if(!o instanceof Point)
			return false;
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}

public class ColorPoint extends Point {
	private final Color color;
	
	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}

	...
}

// 색상 정보를 무시한채 비교하기 떄문에 equlas 수정
   @Override public boolean equals(Object o) {
    	if(!o instanceof ColorPoint)
    		return false;
    	return super.equals(o) && ((ColorPoint) o).color == color;
    }

public static void main(){
      Point p = new Point(1,2);
      ColorPoint cp = new ColorPoint(1,2, Color.RED);
      p.equals(cp);    // true
      cp.equals(p);    // false
    }
```
</br> 

- 처음의 equlas 는 색상 정보를 무시하고 비교하기 때문에 equlas 를 다시 수정했다.
- 하지만 수정한 코드도 문제가 있다.
- ColorPoint 의 equals 는 입력 매개변수의 클래스 종류가 다르다면 매번 false 만 반환한다. (대칭성 위배)

</br> 


```java
    @Override public boolean equals(Obejct o){
      if(!(o instanceof Point))
        return false;
      if(!(o instanceof ColorPoint))
        return o.equals(this);
      return super.equals(o) && ((ColorPoint) o).color == color;
    }

    public static void main(){
      ColorPoint p1 = new ColorPoint(1,2, Color.RED);
      Point p2 = new Point(1,2);
      ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
      p1.equals(p2);    // true 
      p2.equals(p3);    // true 
      p1.equals(p3);    // false
    }
```
- o 가 일반 Point 라면 색상을 무시하고 비교하고, o 가 ColorPoint 라면 색상까지 비교하도록 한다.
- 하지만 이 코드도 추이성 위배라는 문제가 발생한다.
- p1 - p3 는 색상까지 비교하기 때문에 false 가 나오게 된다
- 이러한 방식은 `무한 재귀` 에 빠질 위험도 있다.

</br> 


```java
    @Override public boolean equals(Object o){
      if(o == null || o.getClass() != getClass())
        return false;
      Point p = (Point) o;
      return p.x == x && p.y == y;
    }
```
</br> 

- 위와 같이 instanceof 가 아닌 getClass 로 검사를 하는 것도 `리스코프 치환 원칙`에 위배한다.
- `리스코프 치환 원칙`이란 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다는 것
  - Point 의 하위 클래스는 여전히 Point 이므로 어디서든 Point 로써 활용될 수 있어야 한다.
 
  
</br> 

### 해결 방법 
- 상속 대신 컴포지션을 사용하라 (equals 규약을 지키면서 값 추가하기)
- 컴포지션 : 기존 클래스가 새로운 클래스의 `구성 요소` 로 쓰인다.

</br> 


```java
public class ColorPoint{
  private final Point point;
  private final Color color;

	public ColorPoint(int x, int y, Color color) {
		point = new Point(x, y);
		this.color = Objects.requireNonNull(color);
	}

	/* 이 ColorPoint의 Point 뷰를 반환한다. */
  public Point asPoint(){ // view 메서드 패턴
    return point;
  }

  @Override public boolean equals(Object o){
    if(!(o instanceof ColorPoint)){
      return false;
    }
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```
</br> 

- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 했다.
- 컴포지션을 통해 새 클래스의 인스턴스 메서드들은 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환한다.

</br> 


### 해결 방법 정리
- Point 를 상속하는 대신 Point 를 ColorPoint 의 private 필드로 두고, ColorPoint 와 같은 위치의 일반 Point 를 반환하는 뷰를 public Method 로 추가했다.
- `ColorPoint` 와 `ColorPoint` 비교 : ColorPoint 의 equals 를 이용하여 color 값까지 모두 비교
- `ColorPoint` 와 `Point` 비교 : ColorPoint 의 asPoint 를 이용하여 Point 로 바꿔, Point 의 equals 를 이용해 비교
- `Point` 와 `Point` 비교 : Point 의 equals 를 이용하여 x,y 값 비교
  
</br> 

4. 일관성 : null 이 아닌 모든 참조 값 x,y 에 대해 x.equals(y) 를 반복해서 호출하면 항상 true 이거나 false 이다.
- 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다.
- `가변 객체`의 경우 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있고, `불변 객체`는 한 번 다르면 끝까지 달라야한다.
- 클래스가 불변이든 가변이든 equals 의 파단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.
  - java.net.URL 의 equals 는 주어진 URL 과 매핑된 호스트의 IP 주소를 이용해 비교하는데, 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하므로 그 결과가 항상 같다고 보장할 수 없다.
  - 즉 equals 는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야한다.

</br> 
  
6. null - 아님 : null 이 아닌 모든 참조 값 x 에 대해, x.equals(null) 은 false 다.
- 모든 객체가 `null` 과 같지 않아야 한다.

```java
// 잘못된 명시적 null 검사
@Override
public boolean equals(Object o) {
  if(o == null) { 
      return false;
  }
}

// 올바른 묵시적 null 검사
@Override
public boolean equals(Object o) {
  if(!(o instanceof MyType)) { 
      return false;
  }
  MyType myType = (MyType) o;
}
```

</br> 

- 형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야 한다.
- 입력이 null 이면 타입 확인 단계에서 false 를 반환하므로 null 검사를 명시적으로 하지 않아도 된다.

</br> 

## 정리 : 양질의 equals 메서드 구현 방법
1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
- 자기 자신이면 true 를 반환
- 단순한 성능 최적화용으로 비교 작업이 복잡한 상황일 때 좋다.

2. `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
- 가끔 해당 클래스가 구현한 특정 인터페이스를 비교할 수도 있다.
- 이런 인터페이스를 구현한 클래스라면 equals 에서 (클래스가 아닌) 해당 인터페이스를 사용해야한다.

3. 입력을 올바른 타입으로 형변환 한다.
- 2번에서 `instanceof` 연산자로 입력이 올바른 타입인지 검사 했기 떄문에 이 단계는 100% 성공한다.

4. 입력 객체와 자기 자신의 대응되는 `핵심` 필드들이 모두 일치하는지 하나씩 검사한다.
- 모두 일치해야 true 를 반환

</br> 

## equals 구현시 주의할 추가 사항
- 기본타입 (`==` 연산자 비교)
- 참조타입 (`equals` 메서드로 비교)
- float , dobule 필드 (정적 메서드 `Float.compare` , `Double.compare` 로 비교)
- 배열 필드 (원소 각각을 지침대로 비교하고, 모두가 핵심 필드라면 `Arrays.equals()` 사용)
- null 정상값 취급 방지 (`Objects.equals()` 로 비교하여 `NPE` 발생 예방)
- 비교하기 복잡한 필드를 가진 클래스 (필드의 표준형을 저장한 후 표준형끼리 비교)
- 필드의 비교 순서는 `equals` 성능을 좌우 (다를 가능성이 크거나 비교하는 비용이 싼 필드부터 비교)
- `equals` 를 재정의할 땐 `hashCode` 도 반드시 재정의하자
- 너무 복잡하게 해결하려고 하지 말자
- `Object` 외의 타입을 매개변수로 받는 `equals` 메서드는 선언하지 말자
  - public boolean equals(MyClass o) : 입력 타입이 Object 가 아니므로 `오버로딩` 한 것

 </br> 
 
## 결론
- 꼭 필요한 경우가 아니라면 `equals` 를 재정의 하지 말자
- 많은 경우에 `Object` 의 `equals` 가 원하는 비교를 정확히 수행해준다.
- 재정의 해야할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 5가지의 규약을 확실히 지켜가며 비교해야한다.
- `IDE` 의 도움을 받도록 하자.

