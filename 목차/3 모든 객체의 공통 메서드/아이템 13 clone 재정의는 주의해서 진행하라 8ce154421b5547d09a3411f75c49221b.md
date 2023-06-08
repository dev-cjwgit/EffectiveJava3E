# 아이템 13 clone 재정의는 주의해서 진행하라

**Cloneable 인터페이스 : Object의 protected 메서드인 clone의 동작 방식을 결정**

⇒ 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

**실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.**

## Object 명세 - Clone 메서드의 일반 규약

- **x.clone() ≠ x**
true
- **x.clone().getClass() == x.getClass()**
일반적으론 true, 필수는 아니다.
- **x.clone().equals(x)**
관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다.
Object를 제외한 모든 상위 클래스가 이 관례를 따른다면
true
- **x.clone().getClass() == x.getClass()**
관례상, 반환된 객체와 원본 객체는 독립적이어야 한다.
이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.