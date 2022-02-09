# Chapter 8. 경계

시스템에 들어가는 모든 소프트웨어를 직접 개발하는 겨우는 드물며, 외부 코드를 우리 코드에 깔끔하게 통합해야 한다.

## 외부 코드 사용하기

- 인터페이스 제공자: 적용성을 최대한 넓히려 애쓴다.
- 인터페이스 사용자: 자신의 요구에 집중하는 인터페이스를 바란다.
- 이러한 긴장으로 인해 **시스템 경계**에서 문제가 생긴다.

ex. java.util.Map

- 다양한 인터페이스로 수많은 기능을 제공하지만 그만큼 위험도 크다.
    - clear(), containsKey(), containsValue(), entrySet(), equals(), get(), getClass(), ...
- Map을 인수로 여기저기 넘길 경우
    - Map 사용자라면 누구나 Map 내용을 지울 권한이 있음  // claer()
- 설계 시 Map에 특정 객체 유형만 저장하기로 결정한 경우
    - Map은 객체 유형을 제한하지 않기 때문에 언제든 다른 객체 유형을 추가할 수 있음

```java
// Sensor객체를 담는 Map
Map sensors = new HashMap();

Sensor s = (Sensor)sensors.get(sensorId);
```

→ Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있음

```java
Map<String, Sensor> sensors = new HashMap<Sensor>();

Sensor s = sensors.get(sensorId);
```

→ 제네릭스(Generics)를 사용하면 코드 가독성이 높아짐

❗Map<String, Sensor>는 사용자에게 필요하지 않은 기능까지 제공함

```java
public class Sensors {
	private Map sensors = new HashMap();

	public Sensor getById(String id) {
		return (Sensor) sensors.get(id);
	}
}
```

→ 경계 인터페이스인 Map을 Sensors 안으로 숨긴다.

- Map 인터페이스가 변하더라도 나머지 프로그램에는 영향을 미치지 않는다.
- Sensors 사용자는 제네릭스가 사용되었는지 여부에 신경 쓸 필요가 없다.
- Sensors 클래스 안에서 객체 유형을 관리하고 변환한다.
- Sensors 클래스는 프로그램에 필요한 인터페이스만 제공한다.
- 코드는 이해하기 쉽지만 오용하기는 어렵다.

❗Map을 여기저기 넘기지 말라. Map 인스턴스를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않는다.

## 경계 살피고 익히기

### 학습 테스트

- 외부 코드를 사용할 경우 곧바로 우리쪽 코드에서 외부 코드를 호출하는 대신 **간단한 테스트 케이스를 작성**해 외부 코드를 익히는 것이 좋다.
- 테스트 케이스 작성 시 프로그램에서 API를 사용하려는 방식대로 외부 API를 호출한다.  // 사용 목적에 초점 맞추기

## log4j 익히기

- 로깅 기능을 직접 구현하는 대신 아파치의 log4j 패키지를 사용한다고 가정

→ 테스트 케이스 작성

```java
public class LogTest {
	private Logger logger;

	@Before
	public void initialize() {
		logger = Logger.getLogger("logger");
		logger.removeAllAppenders();
		Logger.getRootLogger().removeAllAppenders();
	}

	@Test
	public void basicLogger() {
		BasicConfigurator.configure();
		logger.info("basicLogger");
	}
	
	@Test
	public void addAppenderWithStream() {
		logger.addAppender(new ConsoleAppender(
				new PatternLayout("%p %t %m%n"),
				ConsoleAppender.SYSTEM_OUT));
		logger.info("addAppenderWithStream");
	}
	
	@Test
	public void addAppenderWithoutStream() {
		logger.addAppender(new ConsoleAppender(
				new PatternLayout("%p %t %m%n")));
		logger.info("addAppenderWithStream");
	}
}
```

→ 간단한 콘솔 로거를 초기화하는 방법을 익혔음~!

→ 이후 독자적인 로거 클래스로 캡슐화하기 → 나머지 프로그램은 log4j 경계 인터페이스를 몰라도 됨

## 학습 테스트는 공짜 이상이다

- 학습 테스트는 필요한 지식만 확보하는 쉬운 방법이며, 투자하는 노력보다 얻는 성과가 더 크다.
- 패키지 새 버전이 나온다면 학습 테스트를 돌려 우리 코드와 호환되는지 확인한다.
- 경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬워진다.

## 아직 존재하지 않는 코드를 사용하기

- **아는 코드와 모르는 코드를 분리하는 경계**

ex. 지정한 주파수를 이용해 이 스트림에서 들어오는 자료를 아날로그 신호로 전송하는 모듈

1. 자체 인터페이스 정의  // transmit(frequency, stream) → 인터페이스를 전적으로 통제
2. 송신기 API에서 필요한 인터페이스만 CommunicationsController로 분리
3. TransmitterAdapter 구현
4. Adapter 패턴으로 API 사용을 캡슐화해 API가 바뀔 때 수정할 코드를 한곳으로 모음
- Adapter 패턴
    
    [https://yaboong.github.io/design-pattern/2018/10/15/adapter-pattern/](https://yaboong.github.io/design-pattern/2018/10/15/adapter-pattern/)
    

## 깨끗한 경계

- 변경
    - 통제하지 못하는 코드를 사용할 때는 너무 많은 투자를 하거나 향후 변경 비용이 지나치게 커지지 않도록 주의해야 한다.
- 경계에 위치하는 코드는 깔끔히 분리하며, 테스트 케이스도 작성한다.
    - 통제가 불가능한 외부 패키지에 의존하는 대신 통제가 가능한 우리 코드에 의존하는 편이 훨씬 좋다.
- 외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하자.
    - 새로운 클래스로 경계를 감싸거나 Adapter 패턴을 사용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환하자.
    - 코드 가독성, 경계 인터페이스를 사용하는 일관성이 높아지며, 외부 패키지가 변했을 때 변경할 코드도 줄어든다.
