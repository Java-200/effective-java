## finalizer와 cleaner 사용을 피하라

(언젠가 다시 공부 필요 ... ☠️)

### **문단 1**

- 자바에서는 **finalizer와 cleaner 라는 객체 소멸자**를 사용한다.
- finalizer는 예측할 수 없고 상황에 따라 위험할 수 있어 일반적으로 불필요하고 , 쓰지말아야 한다.
- cleaner는 finalizer 보다 덜 위험하지만, 여전히 에측할 수 없고 느리고 일반적으로 불필요하다.

### **문단2**

- C++의 파괴자와 자바의 finalizer와 cleaner는 다른 역할이다.
- C++의 파괴자는 객체와 관련된 자원을 회수하는 보편적인 역할이다.
    - 비메모리 자원을 회수하는 용도로 쓰인다.
    - 비메모리 자원이란 단순히 메모리 자원이 아닌 외부 시스템과 연관된 자원을 의미.
        - 파일 (파일 핸들, 파일 스트림), 네트워크 연결(소켓), 데이터베이스 연결 (DB 커넥션) …
- 자바에서는 try-with-resources와 try-finally를 사용하여 해결한다.

### **문단3**

- finalizer와 cleaner는 즉시 수행된다는 보장이 없다. → 제 때 실행되어야 하는 작업은 할 수 없다.
    - 예) 파일 닫기
    - 시스템이 동시에 열 수 있는 파일의 개수는 제한이 있는데, finalizer와 cleaner 에 ‘파일 닫기’를 시키면 파일이 제 때 닫히지 않아 프로그램 오류를 발생시킬 수 있다.

### **문단4**

- finalizer와 cleaner의 실행 시점을 예측할 수 없다.
    - GC의 알고리즘에 따라 다를 수 있다.
    - 특정 환경에서는 잘 동작해도 다른 환경에서는 예측하지 못한 문제가 발생할 수 있다.
        - ex. 내 테스트 JVM에서는 잘되나, 고객 시스템에서는 오류 발생할 수 있음.

### **문단5**

- 클래스에 finalizer를 달아두면 그 인스턴스의 자원 회수가 제멋대로 지연될 수 있다.

```java
class Resource {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("리소스 정리 완료!");
    }
}

public class Main {
    public static void main(String[] args) {
        new Resource(); // 객체 생성 후 바로 참조 제거
        System.gc(); // GC 실행 요청
        System.out.println("프로그램 종료!");
    }
}
```

- finalizer() 메서드를 오버라이드한 Resource 클래스
- GC 실행 요청을 명시적으로 하여도, GC가 언제 실행될 지는 알 수 없다. → finalize() 언제 실행될 지 알 수 없음.
- **GC와 finalizer**
    - GC가 객체 메모리 회수를 시작할 때, 더 이상 사용되지 않는 객체를 식별한다.
    - 이 객체들이 finalizer() 메서드를 가지고 있다면 finalize()를 실행하기 위해 finalizer 대기열에 추가함.
    - GC가 이 대기열을 처리하며 finalize() 메서드를 실행하고, 이 때 실행되는 스레드는 finalizer 스레드라고 부른다.

** 책에서 설명한 일화 > finalizer로 인한 불필요한 지연으로 OOM 발생

1. 프로그램 OOM 발생하여 디버깅
2. 객체 수천 개가 finalizer 대기열에서 회수되기만을 기다리고 있음.
    1. 즉, 객체 수천 개가 finalizer가 호출되기만을 기다리고 있었다는 것.
    2. finalizer는 GC가 객체를 회수하기 전에 자원 정리하는 작업을 하지만, 이게 언제 실행될 지 알지 못함.
3. finalizer 스레드는 다른 애플리케이션 스레드보다 우선 순위가 낮아 실행될 기회를 제대로 얻지 못하였다.
    1. finalizer 스레드는 우선순위가 낮은 스레드로, 다른 애플리케이션의 스레드가 CPU를 많이 차지하고 있으면 finalizer 스레드는 지연될 수 있다. 
    2. finalizer가 실행되기 까지 시간이 오래 걸림.
    3. 객체 수천 개는 메모리를 차지하고 있지만 계속 finalizer가 실행되길 기다렸고, 결국 메모리 해제가 이뤄지지 않아 시스템 메모리 부족 상태가 됨.

- 자바 언어 명세(Java Language Specification
    - 자바 프로그래밍 언어의 문법, 규칙, 기능, 동작 등을 정의한 공식 문서이다.
    - 즉, 자바 언어가 어떻게 동작하는지에 대한 표준화된 규칙을 제공한다.

→ 하지만, 자바 언어 명세에서 finalizer 스레드가 언제? 얼마나? 수행될 지는 알 수 없기 때문에 해당 문제를 해결할 수 있는 방법은 … ‘**finalizer 를 사용하지 않는다.**’ 뿐이다.

- cleaner는 스레드를 제어할 수 있지만, GC의 통제하에 있어 즉각 수행되리라는 보장은 없다.

### 문단 6

- 자바 언어 명세는 finalizer와 cleaner의 수행 여부를 보장하지 않는다.
- 일부 객체의 종료 작업을 전혀 수행하지 못한 채 프로그램이 종료될 수 있다.
- 이 때문에 상태를 영구적으로 수정하는 작업에서는 절대 finalizer와 cleaner를 의존하지 마라.
    - ex. DB와 같은 공유 자원의 영구 락 해제를 finalizer와 cleaner에게 맡기면 분산 시스템 전체가 서서히 멈춘다.

### 문단 7

### 문단 8: finalizer의 문제점2 - 예외 발생 무시

- finalizer 실행 중 예외가 발생해도 무시되고 그 즉시 종료된다.
- 보통 예외가 스레드를 중단시키고 스택 추적 내역을 출력하겠지만, 같은 일이 finalizer에서 일어난다면 경고조차 출력하지 않는다.
- cleaner는 스레드 통제 가능하여 해당 문제 발생 X

### 문단 9: finalizer와 cleaner 문제점 3 - 성능 문제

- AutoCloseable 객체 생성 후 GC 수거까지 12ns 걸림. (try-with-resources)
- 반면에 finalizer를 사용하면 550ns가 걸림. (50배) / cleaner도 마찬가지.
- 하지만 안전망 형태로 finalizer/cleaner를 사용하면 66ns 정도 걸림.

### 문단 10: finalizer 문제점 4 - 보안 문제

- finalizer 를 사용하는 클래스는 공격에 노출되어 심각한 보안 문제 발생될 수 있다.

- finalizer 공격 원리
    1. 생성자, 직렬화 과정에서 예외 발생 시, 생성되다 만 객체에서 악위적 하위 클래스의 finalizer 가 수행될 수 있음. 
        1. finalizer()가 존재하면, 객체가 완전히 생성되지 않더라도 finalize() 메서드가 실행되게 됨. (객체가 비정상적으로 생성된다.)
    2. finalizer는 정적 필드에 자신의 참조를 할당하여 GC가 수집하지 못하도록 막음.
    3. 이러한 객체가 생성되면, 허용되지 않았을 작업이 수행 가능해진다. 
    
    → **객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 그렇지 않다.** 
    

- 대처 방법
    - final 클래스들은 하위 클래스를 만들 수 없으니 공격에서 안전하다.
        - final 클래스는 상속을 허용하지 않아 해당 클래스를 상속한 후에 finalize() 메서드를 오버라이드하여 악성 코드 삽입을 막을 수 있다.
    - final이 아닌 클래스를 finalizer 공격으로부터 방어하려면, 아무 일도 하지 않는 finalize 메서드를 만들고 Final로 선언하라.

### 문단 11: finalizer와 cleaner의 묘안

- AutoCloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰면 close 메서드 호출
    - `AutoCloseable` : 자원을 닫는 작업을 자동으로 수행할 수 있도록 도와주는 인터페이스
    - `close()` : 자원을 해제하는 메서드

### 문단 12, 13: finalizer와 cleaner의 적절한 쓰임새

1. close 메서드 미호출에 대한 안전망 역할 
2. 네이티브 피어와 연결된 객체에서 네이티브 객체까지 회수할 때
    1. `네이티브 피어` : 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
        1. `네이티브 코드` : 자바 외의 다른 언어로 작성된 코드 (C, C++, ..)
        2. 네이티브 코드와 자바는 JNI(Java Native Interface)를 통해 서로 상호작용 가능
            1. JNI는 코드 간 데이터를 전달하거나, 네이티브 메서드를 호출할 수 있도록 해주는 인터페이스
        3. 자바 객체가 네이티브 메서드를 통해 네이티브 객체에 기능을 위임하는 구조이다. 자바 객체는 네이티브 객체를 직접적으로 관리하고 참조하진 않지만, 네이티브 메서드를 통해 네이티브 객체와 상호작용을 한다. ? 
        
        ⇒ **자바 객체와 연결된 네이티브 코드에서 처리되는 객체**
        
    2. 네이티브 피어는 자바 객체가 아니라 **GC는 그 존재를 알 수 없다.** 
    → 자바 객체 회수 시 네이티브 객체까지 회수하지 못함. 
    → cleaner나 finalizer가 필요하다. (단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때만 … / 아니라면 close 메서드 사용)

### 문단 14~끝: cleaner의 까다로운 사용법 (잘 모르겠어요 …)

```java
public class Room implements AutoCloseable{
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable{
        int numJunkPiles; // 방(Room) 안의 쓰레기 ㅅ

        public State(final int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

				// close 메서드나 cleaner가 호출한다. 
        @Override
        public void run() {
            System.out.println("방청소");
            numJunkPiles = 0;
        }
    }

		// 방의 상태. cleanable과 공유한다.
    private final State state;
	
		// cleanable 객체. 수거 대상이 되면 방을 청소한다. 
    private final Cleaner.Cleanable cleanable;

    public Room(final int numJunkFiles) {
        this.state = new State(numJunkFiles);
        cleanable = cleaner.register(this, state); // Runnable 객체를 등록
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```

- 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할이다.
- Room의 close 메서드를 호출하거나, GC가 Room을 회수할 때까지 close가 호출되지 않는다면,
cleaner가 State의 run 메서드를 호출해줄 것이다.