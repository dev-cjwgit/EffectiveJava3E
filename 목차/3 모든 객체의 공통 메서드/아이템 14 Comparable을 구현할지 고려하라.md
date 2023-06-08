# 아이템 14 Comparable을 구현할지 고려하라

Comparable 인터페이스의 compareTo을 알아보자.

compareTo는 Object의 메서드가 아니다.

- 단순 동치성 비교
- 제네릭

하다는 특성을 가지고 있다.

**알바펫, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable를 구현하자.**

## compareTo 일반 규약

Comparable을 구현한 클래스는

- 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다.
**∴** x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때에 한해 예외를 던져야 한다.
- 추이성을 보장해야한다.
즉, (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0 이다.
- 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))다

compareTo 규악을 지키지 못하면 비교를 활용하는 클래스를 사용하기 힘들다.

- TreeSet, TreeMap, Collections, Arrays 등

## compareTo 규약

- 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.
- 첫 번째가 두 번째 보다 크고, 두 번째는 첫 번째보다 작아야 한다.
- 크기가 같은 객체들 끼리 어떤 객체와 비교하더라도 항상 같아야 한다

이 세 규약은 equals 규약과 동일하게 **반사성, 대칭성, 추이성**을 충족해야한다.

기존 클래스를 확장한 구체 클래스에서 값을 추가했다면 compareTo 규약을 지킬 방법이 없다.(equals와 동일)

세 번째 규약은 필수는 아니지만 꼭 지키길 권장한다.

- compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.

compareTo의 순서와 equals의 결과가 일관되지 않은 클래스를 Collections, Set, Map에 넣으면 예상 된 값이 안나올 수 있다.
∵ 정렬된 컬렉션들은 동치성을 비교할 때 equals 대신 compareTo를 사용하기 떄문이다.

ex) new BigDecimal(”1.0”), new BigDecimal(”1.00”)

- HashSet은 equals로 비교하여 2개를 갖게 된다.
- TreeSet은 하나의 원소만 생성된다.

## compareTo 메서드 작성 요령

- 입력 인수 타입을 확인하거나 형변환할 필요가 없다.
- null을 인수로 넣어 호출하면 NullPointerException을 던져야 한다.
- 객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다.

**compareTo 메서드에서 관계 연산자 ‘<’,  ‘>’를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 이제는 추천하지 않는다.**

```java
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode);
	if (result == 0) {
		result = Short.compare(prefix, pn.prefix);
		if (result == 0) {
			result = Short.compare(lineNum, pn.lineNum);
		}
	}
	return result;
}
```

```java
private static final Comparator<PhoneNumber> COMPARATOR = 
				comparingInt((PhoneNumber pn) -> pn.areaCode)
					.thenComparingInt(pn -> pn.prefix)
					.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```