# private 생성자나 열거 타입으로 싱글턴임을 보증하라. <br>

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.  <br>

## 📌 싱글턴의 한계 <br>
1️⃣ private 생성자를 가지고 있기 때문에 상속할 수 없다. <br>
2️⃣ 싱글턴은 테스트하기가 어렵다. <br>
3️⃣ 서버환경에서는 싱글턴이 하나만 만들어지는 것을 보장하지 못한다. <br>
4️⃣ 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다. <br>

## 📌 싱글턴 사용 이유 <br>
한 번의 객체 생성으로 재사용이 가능하기 때문에 메모리 낭비를 방지할 수 있다. <br>

## 📌 싱글턴을 만드는 방식 <br>
1️⃣ public static 멤버가 final 필드인 방식 <br>
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }

	public void leaveTheBuilding()
}
```

### 위와 같이 작성하면 장점 <br>
1. 해당 클래스가 싱글턴임이 API에 명백히 드러난다. <br>
2. 간결하다. <br>

2️⃣ 정적 팩터리 메서드를 public static 멤버로 제공하는 방식 <br>
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding()
}
```

### 위와 같이 작성하면 장점 <br>
1. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다. <br>
2. 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다. <br>
3. 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다는 점이다. <br>
<br>

둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언한은 것만으로는 부족하다. <br>
모든 인스턴스 필드를 일시적이라고 선언하고 readResolve 메서드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때 마다 새로운 인스턴스가 만들어진다. <br>
2️⃣의 코드를 보면 가짜 Elvis가 탄생한다는 뜻이다. 이를 예방하고 싶다면 Elvis 클래스에 readResolve 메서드를 추가해야 한다. <br>

<br>

```java
private Object readResolve() {
	return INSTANCE;
}
```

<br>

3️⃣ 열거 타입 방식의 싱글턴 - 바람직한 방법 <br>
public 필드 방식과 비슷하지만, 더 간결하고 직렬화를 쉽게 할 수 있다. 또 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽히 막아준다. <br>
대부분 상황에서는 원소가 하나 뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다. <br>

```java
public enum Elvis {
	INSTANCE;
	public void leaveTheBuilding() { ... }
}
```
