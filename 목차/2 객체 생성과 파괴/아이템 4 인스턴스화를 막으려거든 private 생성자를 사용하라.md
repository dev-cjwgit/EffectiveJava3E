# 아이템 4 인스턴스화를 막으려거든 private 생성자를 사용하라

**추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.**

→ 하위 클래스를 만들어 인스턴스화 하면 그만이다.

private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

```java
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
	private UtilityClass(){
		throw new AssertionError();
	}

	... // 나머지 코드 생략
}
```