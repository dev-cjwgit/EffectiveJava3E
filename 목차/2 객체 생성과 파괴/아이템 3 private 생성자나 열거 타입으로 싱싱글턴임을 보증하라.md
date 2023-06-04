# 아이템 3 private 생성자나 열거 타입으로 싱글턴임을 보증하라

**클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.**

- 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문
1. public static final 필드 방식의 싱글턴
    
    ```java
    public class Elvis {
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis() { ... }
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    Elvis.INSTANCE를 초기화할 때 딱 한번만 호출한다.
    다만, 예외로는 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다.
    
    (생성자를 수정하여 두번째 객체가 생성되려 할 때 예외를 던지게 해야함)
    
2. 정적 팩터리 방식의 싱글턴
    
    ```java
    public class Elvis {
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis() { ... }
    	public static Elvis getInstance() { return INSTANCE; }
    
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    제 2의 Elvis 인스턴스는 만들어지지 않는다 (리플렉션 제외)
    
3. **열거 타입 방식의 싱글턴(권장)**
    
    ```java
    public enum Elvis {
    	INSTANCE;
    
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    더 간결하고 직렬화가 가능하다.
    또한, 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽하게 막아준다.