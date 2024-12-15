### 싱글턴이란 ?

인스턴스를 오직 하나만 생성할 수 있는 클래스 <br>
ex) 무상태 객체(함수) || 시스템 컴포넌트(설계상 유일)

**클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트가 테스트하기 어려워질 수 있다.**

🧐 why 🧐
- 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면, <br>
**싱글턴 패턴을 가짜(mock) 구현으로 대체할 수 있기 때문**

(1) 단점의 싱글턴 클래스

```java
public class Concert {
   private boolean lightsOn;

   private Elvis elvis;

   public Concert(Elvis elvis) {
      this.elvis = elvis;
   }

   public void perform() {
      lightsOn = true;
      elvis.sing();
   }

   public boolean isLightsOn() {
      return lightsOn;
  }

  public boolean isMainStateOpen() {
    return mainStateOpen; 
  } 
}
```

```java
class ConcertTest {

   @Test
   void perform() {
      Concert concert = new Concert(Elvis.INSTANCE);
      concert.perform();

      Assertions.assertTrue(concert.isLightsOn());
      Assertions.assertTrue(concert.isMainStateOpen());
   }
```

- Concert 라는 클래스가 있고, 이 클래스에서 Elvis를 사용하고 있다.
- 즉, Concert라는 클래스가 Elvis 클래스의 클라이언트 코드이다.
    - Elvis 클래스를 직접 사용하기 때문에 테스트 하기 어려워진다.

(2) 인터페이스로 개선한 싱글턴 클래스

```java
public interface IElvis {
   void leaveTheBuilding();
   void sing();
}
```

```java
// Evlis가 IElvis를 구현한다.
public class Elvis implements IElvis{

   // 싱글톤 오브젝트
   public static final Elvis INSTANCE = new Elvis();

   private Elvis() {}

   public void leaveTheBuilding() {
      System.out.println("start song !");
   }

   public void sing() {
      System.out.println("sing sang song !");
   }

   public static void main(String[] args) {
      Elvis elvis = Elvis.INSTANCE;
      elvis.leaveTheBuilding();
   }
}
```

```java
public class Concert {
   private boolean lightsOn;

   private boolean mainStateOpen;

   private IElvis elvis;

   public Concert(IElvis elvis) {
      this.elvis = elvis;
   }

   public void perform() {
      mainStateOpen = true;
      lightsOn = true;
      elvis.sing();
   }

   public boolean isLightsOn() {
      return lightsOn;
   }

   public boolean isMainStateOpen() {
      return mainStateOpen;
   }
}
```
```java
class ConcertTest {

   @Test
   void perform() {
      Concert concert = new Concert(new MockElvis());
      concert.perform();

      Assertions.assertTrue(concert.isLightsOn());
      Assertions.assertTrue(concert.isMainStateOpen());
   }
}
```
- IElvis 인터페이스를 구현한 클래스를 통해 테스트하기가 수월해진다.

<br>

### [싱글턴 패턴을 만드는 방식]

### 1. public static final 필드 방식의 싱글턴

```java
public class Elvis{
  public static final Elvis instance = new Elvis();
  private Elvis(){...}
  
  public void leaveTheBuilding(){...}
}
```

- private 생성자는 Elvis.Instace를 초기화할 때 한 번만 호출
    - public, protected 생성자가 없으므로 해당 내용 보장
- 예외) AccessibleObject.setAccessible을 사용해 private 호출 → 예외 던지자

### 2. 정적 팩터리 메서드 방식의 싱글턴

```java
public class Elvis{
  private static final Elvis INSTANCE = new Elvis();
  private Elvis(){...}
  public static Elvis getInstance {return INSTANCE;}
  
  public void leaveTheBuilding() {...}
}
```

- Elvis.getInstance는 항상 같은 객체의 참조를 반환하므로 제 2의 Elvis 인스턴스 만들어지지 X
- 장점 1) API를 바꾸지 않고도 싱글턴 아니게 변경할 수 있음
- 장점 2) 제네릭 싱글턴 팩터리로 만들 수 있음
- 장점 3) 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있음

### 3. 열거 타입 방식의 싱글턴 - 바람직한 방법

```java
private enum Elvis{
  INSTANCE;
  
  public void leaveTheBuilding() {...}
}
```

- public 필드 방식과 유사하지만, 더 간결하고, 추가 노력 없이 직렬화 가능
