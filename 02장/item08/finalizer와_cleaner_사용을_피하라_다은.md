## 자바의 객체 소멸자
- finailzer: 예측할 수 없고, 상황에 따라 위험 (오동작, 낮은 성능, 이식성 문제의 원인)
    - finailzer 스레드가 다른 스레드보다 우선 순위가 낮아 finailzer 처리만을 기다리다 OutOfMemoryError가 발생할 수 있음
    - finailzer 동작 중 발생한 예외는 무시되고 처리할 작업이 남았더라도 종료되어 객체가 훼손될 수 있음
- cleaner: 덜 위험하지만 여전히 예측할 수 없고 느림

⇒ 자바에서는 finailzer와 cleaner의 수행 시점과 수행 여부를 보장하지 않음

⇒ 근데 가비지 컬렉터의 효율을 떨어뜨리기까지 하니 성능 저하도 동반함

⇒ 거기다 finailzer를 사용한 클래스는 finailzer 공격을 당할 수 있음

<br>

### finailzer와 cleaner의 대안
- AutoCloseable을 구현한 후 클라이언트가 인스턴스를 다 쓰면 close 메서드를 호출
- close 메서드가 이 객체가 유효하지 않음을 필드에 기록하고, 다른 메서드가 필드를 검사해서 IllegalStateException을 던지는 방식

<br>

### finailzer와 cleaner의 사용
- 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망
    - 안 하는 것보다는 늦게라도 하는 게 낫기 때문
    - e.g., FileInputStream, FileOutputStream, ThreadPoolExecutor
- 네이티브 피어와 연결된 객체
    - 네이티브 피어(native peer): 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
    - 자바 객체가 아니라서 가비지 컬렉터가 이 존재를 모르기 때문에 finalizer나 cleaner가 처리하기 적당한 작업
    - 그러나 성능저하를 감당할 수 없거나 즉시 회수해야 한다면 close 메서드 사용
