### 클래스 / 객체 / 인스턴스

- 클래스: 객체를 정의해 놓은 설계도 (문법, 자료형)
- 객체: 클래스에 정의된 내용을 바탕으로 생성된 것 (개념, 구조체, 형식, 단위)
- 인스턴스화: 클래스를 기반으로 객체를 실제로 생성하는 것 (객체를 메모리에 할당)
- 인스턴스: 클래스를 통해 생성된 객체 (실체)

<br>

## 유틸리티 클래스

- 특정 목적을 위해 여러 메서드를 제공하는 클래스로 다른 클래스를 보조하는 역할
- 주로 정적 메서드와 정적 필드로 구성되어 있음

<br>

### 예시

- 기본 타입 값이나 배열 관련 메서드들을 모아놓은 경우
    - java.lang.Math와 java.util.Arrays
    - 3^2==9와 같이 동일한 입력에 동일한 결과를 리턴하면 메서드를 재사용하는 게 더욱 효율적
- 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓은 경우
    - java.util.Collections
- final 클래스와 관련한 메서드들을 모아놓은 경우

**⇒ 인스턴스 없이 기능을 수행할 수 있음**

<br>

### 그러나..

- 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만듦
- 매개변수를 받지 않는 pubilc 생성자가 만들어지고, 자동 생성된 건지 구분할 수 없음
    
    ⇒ 의도치 않게 인스턴스화할 수 있게 된 것
    

<br>

## 인스턴스 생성을 막는 방법

### 1. 추상 클래스 만들기

```java
public **abstract** class UtilityClass{
	public static void util(){
		System.out.println("Util");
	}
}
```

<br>

- 하지만 하위 클래스를 만들어 인스턴스화할 수 있다.

```java
public class ChildUtilityClass extends UtilityClass{
}
```

```java
public static void util(){
	ChildUtilityClass util = new ChildUtilityClass();
}
```

<br>

### 2. private 생성자 만들기

- 명시된 생성자를 만들어 컴파일러가 기본 생성자를 만들지 않도록

```java
public class UtilityClass{
  //기본 생성자 생성 방지용
	private UtilityClass(){
		throw new AssertionError();
	}
}
```

- private는 동일 클래스 내의 멤버만 접근 가능
    - 하위 클래스가 상위 클래스의 생성자에 접근할 수 없어 자연스럽게 상속을 불가능하게 만듦
- 내부에는 Error 처리를 통해 실수로라도 생성되지 않게 어떤 환경에서든 클래스의 인스턴스화를 막음
- 그러나, 생성자를 실행하지 않으려고 생성자를 구현하는 / 생성자가 있는데 호출할 수 없는 상황.. ;;
    
    →  주석을 꼭 달아서 개발자에게 알려주도록
