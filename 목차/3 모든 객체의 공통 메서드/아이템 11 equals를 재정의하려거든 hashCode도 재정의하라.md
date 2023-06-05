# 아이템 11 equals를 재정의하려거든 hashCode도 재정의하라

**equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.**

그렇지 않으면 HashMap, HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

## Object 명세에서 발취한 규약

- equals 비교에 사용되는 정보가 변견되지 않았따면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없다
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

**hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다.
즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
```

실제로 “제니”가 출력 되어야 할 것 같지만, hashCode를 재정의하지 않았기 때문에 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번쨰 규악을 지키지 못한다.

⇒ null 반환

```java
@Override
public int hashCode() {
	return 42;
}
```

동치인 모든 객체에서 똑같은 해시 코드를 반환하니 적법하다.

하지만 해시테이블의 버킷 하나에만 담겨서 LinkedList 처럼 동작한다.

⇒ O(n)이 소요된다

## 좋은 hashCode를 작성하는 간단한 요령

1. int 변수 result를 선언 후 값 c로 초기화 한다.
이때 c는 해당 객체의 첫 번째 핵심 필드를 단계 2.a 방식으로 계산한 해시 코드다.
(핵심 필드란, equals 비교에 사용되는 필드)
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c를 계산한다.
        1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다.
        (Type은 해당 기본 타입의 박싱 클래스)
        2. 참조 타입 필드면서 이 클래스의 equals가 재귀적으로 호출한다면 이 필드의 hashCode를 재귀적으로 호출한다.
        3. 필드가 배열이라면, 핵심 원소를 각각의 별도 필드처럼 다룬다.
        (Arrays.hashCode 사용 가능), 핵심 필드가 없다면 0을 추천
    2. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다.
    result = 31 * result + c;
3. result를 반환한다.

**31을 곱하는 이유**

31이 홀수이면서 소수이다.

→ 이 숫자가 짝수이고 오버플로가 발생한다면 정보를 잃게 된다.
→ 2가 아닌 이유는 시프트 연산과 같은 결과를 내기 떄문이다.

```java
@Override
public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```

```java
private int hashCode;

@Override
public int hashCode() {
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

**성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.**

→ 속도야 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다.

**hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.**

→ 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수 있다.