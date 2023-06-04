# 아이템 8 finalizer와 cleaner 사용을 피하라

자바는 두 가지 객체 소멸자를 제공한다.
**finailzer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.**

**cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.**

자바의 finlizer와 cleaner은 C++의 파괴자와는 다르다.

C++은 파괴자에서 자원을 회수하는 보편적인 방법이다. 이는 비메모리 자원을 회수하는 용도로도 쓰이지만, 자바에서는 try-with-resources와 try-finally를 사용해 해결한다.

**자바에서 cleaner와 finalizer는 제때 실행되어야 하는 작업은 절대 할 수 없다.**

파일 닫기를 소멸자에다 맡기면 큰 오류를 일으킬 수 있다.

→ 시스템이 파일을 열 수 있는 한계가 있기 때문

자바 언어 명세는 finalizer나 cleaner의 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다.

따라서 프로그램 **생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.**

System.gc나 System.runFinalization 은 실행될 가능성을 높여줄 뿐, 보장해주진 않는다.

**finalizer와 cleaner는 심각한 성능 문제도 동반한다.**

- AutoCloseable 객체를 생성 후 GC가 수거하기까지 12ns 소요
- finalizer는 550ns가 소요

## 방(Room) 자원을 수거하기 전 반드시 청소(Clean)해야 한다고 가정한 예시

```java
import java.lang.ref.Cleaner;

// 코드 8-1 cleaner를 안전망으로 활용하는 AutoCloseable 클래스 (44쪽)
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
		// 방을 청소할 때, 수거할 자원들을 담고 있다.
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

State 인스턴스가 Room의 인스턴스를 참조하게 된다면, 순환 참조가 발생하여 GC가 Room 인스턴스를 회수 해갈 기회가 오지 않는다.