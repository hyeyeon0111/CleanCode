# Chapter 11. 시스템

높은 추상화 수준, 시스템 수준에서도 깨끗함을 유지하는 방법을 살펴보자.

## 시스템 제작과 시스템 사용을 분리하라

- **제작**(construction)은 **사용**(use)와 아주 다르다.
- 소프트웨어 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 ‘연결'하는) **준비과정**과 (준비과정 이후에 이어지는) **런타임 로직**을 분리해야 한다.

**관심사 분리**

ex. 관심사를 분리하지 않은 코드

```java
// bad
public Service getService() {
	if(service == null) {
		service = new MyServiceImpl(...);  // 모든 상황에 적합한 기본값일까?
	}
	return service;
}
```

→ 초기화 지연(Lazy Initialization) 또는 계산 지연(Lazy Evalution) 기법

- 장점
    - 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않는다. 애플리케이션을 시작하는 시간이 빨라진다.
    - 어떤 경우에도 null 포인터를 반환하지 않는다.
- 단점
    - getService 메서드가 MyServiceImpl에 명시적으로 의존한다. 런타임 로직에서 MyServiceImpl 객체를 전혀 사용하지 않더라도 의존성을 해결하지 않으면 컴파일이 안 된다.
    - 테스트
        - getService 메서드를 호출하기 전에 테스트 전용 객체(TEST DOUBLE, MOCK OBJECT)를 service 필드에 할당해야 한다.
        - 런타임 로직에 객체 생성 로직을 섞어놨기 때문에 service가 null인 경로, null이 아닌 경로 모두를 테스트해야 한다.
        - 메서드가 두 가지 이상의 작업을 수행하기에 단일 책임 원칙(SRP)을 깬다.
    - MyServiceImpl이 모든 상황에 적합한 객체인지 모른다.

### 결론

- 모듈성을 깨서는 안된다. (객체를 생성하거나 의존성을 연결할 때에도)
- 설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다.
- 의존성을 해소하기 위한 일반적인 방식이 필요하다.

### Main 분리

- 생성과 관련한 코드는 모두 main이나 main이 호출하는 모듈로 옮기고, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.
1. main 함수에서 시스템에 필요한 객체를 생성한 후 애플리케이션에 넘긴다.
2. 애플리케이션은 객체를 사용한다.
- 모든 의존성이 main → 애플리케이션 쪽을 향한다.
    - 애플리케이션은 main이나 객체가 생성되는 과정을 전혀 모른다.

### 팩토리

- 객체가 생성되는 시점을 애플리케이션이 결정할 때도 있다.

ex. 주문처리 시스템

1. 애플리케이션은 LineItem 인스턴스를 생성해 Order에 추가한다.  // ABSTRACT FACTORY 패턴
2. LineItem을 생성하는 시점은 애플리케이션이 결정하지만 LineItem을 생성하는 코드는 모른다.
- 모든 의존성이 main → OrderProcessing 애플리케이션으로 향한다.
    - OrderProcessing은 LineItem이 생성되는 구체적인 방법을 모른다.
- 하지만 OrderProcessing은 LineItem 인스턴스가 생성되는 시점을 완벽하게 통제한다.

### 의존성 주입 ⭐

- 제어 역전(Inversion of Control, IoC) 기법을 의존성 관리에 적용한 매커니즘
    - 제어 역전
        - 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다.
        - 새로운 객체는 넘겨받은 책임만 맡는다.(SRP)
    - 의존성 관리
        - 객체는 의존성 자체를 인스턴스로 만드는 책임은 지지 않는다.
        - 책임을 다른 ‘전담' 메커니즘에 넘긴다. → 제어를 역전
- 초기 설정은 ‘책임질' 메커니즘으로 ‘main’ 루틴이나 특수 ‘컨테이너’를 사용한다.

ex. ‘부분적' 의존성 주입 - JNDI 검색

```java
MyService myService = (MyService) (jndiContext.lookup("NameOfMyService"));
```

- 객체는 디렉터리 서버에 이름을 제공하고 그 이름에 일치하는 서비스를 요청
- 호출하는 객체는 반환되는 객체의 유형을 제어하지 않음

**진정한 의존성 주입**

- 클래스가 의존성을 해결하려 시도하지 않는다. 수동적이다.
- 의존성을 주입하는 방법
    - 설정자(setter)
    - 생성자 인수
- DI 컨테이너는 필요한 객체의 인스턴스를 만든 후 생성자 인수나 설정자 메서드를 사용해 의존성을 설정한다.
- 생성되는 객체 유형은 설정 파일에서 지정하거나 특수 생성 모듈에서 코드로 명시한다.
- 스프링 프레임워크
    - 자바 DI 컨테이너 제공
    - 객체 사이의 의존성은 XML 파일에 정의
    - 자바 코드에서는 이름으로 특정 객체 요청
- DI 컨테이너는 필요할 때까지는 객체를 생성하지 않고 계산 지연, 팩토리 호출, 프록시 생성 방법을 제공한다.

## 확장

- 테스트 주도 개발(TDD), 리팩터링, 깨끗한 코드는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.
- 관심사를 적절히 분리해 관리 한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다.

ex. 관심사를 적절히 분리하지 못한 아키텍처

```java
// 11-1. Bank EJB용 EJB2 지역 인터페이스
public interface BankLocal extends java.ejb.EJBLocalObject {
	String getStreetAddr1() throws EJBException;
	String getStreetAddr2() throws EJBException;
	String getCity() throws EJBException;
	String getState() throws EJBException;
	String getZipCode() throws EJBException;
	void setStreetAddr1(String street1) throws EJBException;
	void setStreetAddr2(String street2) throws EJBException;
	void setCity(String city) throws EJBException;
	void setState(String state) throws EJBException;
	void setZipCode(String zip) throws EJBException;
	Collection getAccounts() throws EJBException;
	void setAccounts(Collection accounts) throws EJBException;
	void addAccounts(AccountDTO accountDTO) throws EJBException;
}
```

```java
// 11-2. 상응하는 EJB2 엔티티 빈 구현
public abstract class Bank implements javax.ejb.EntityBean {
	// 비즈니스 논리...
	public abstract String getStreetAddr1();
	public abstract String getStreetAddr2();
	public abstract String getCity();
	public abstract String getState();
	public abstract String getZipCode();
	// ...
	public void addAccount(AccountDTO accountDTO) {
		InitialContext context = new InitialContext();
		AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
		AccountLoacl account = accountHome.create(accountDTO);
		Collection accounts = getAccounts();
		accounts.add(account);
	}
	// ...
}
```

```java
// + xml 배포 기술자 작성
// 영구 저장소에서 객체와 관계형 자료가 매핑되는 방식, 원하는 트랜잭션 동작 방식, 보안 제약조건 등이 들어간다.
```

- 비즈니스 논리는 EJB2 애플리케이션 ‘컨테이너'에 강하게 결합된다.
    - 클래스를 생성할 때는 컨테이너에서 파생해야 하며 컨테이너가 요구하는 다양한 생명주기 메서드도 제공해야 한다.
- 독자적인 단위 테스트가 어렵다.

### 횡단(cross-cutting) 관심사

- 영속성과 같은 관심사는 애플리케이션 객체 경계를 넘나드는 경향이 있다.
    - 모든 객체가 전반적으로 동일한 방식을 이용하게 만들자.
    - 특정 DBMS나 독자적인 파일을 사용, 테이블과 열은 같은 명명 관례 따르기, 트랜잭션 의미 일관적으로..
- 현실적으로 영속성 방식을 구현한 코드가 온갖 객체로 흩어진다.  // 횡단 관심사
- 하지만, 영속성 프레임워크와 도메인 논리를 모듈화할 수 있다.

**관점 지향 프로그래밍(AOP)**

- 횡단 관심사에 대처해 모듈성을 확보하는 일반적인 방법론
- 관점 aspect
    - 특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야한다.
    - ex. 영속성
    1. 영속적으로 저장할 객체와 속성을 선언한 후 영속성 책임을 영속성 프레임워크에 위임한다.
    2. AOP 프레임워크는 대상 코드에 영향을 미치지 않는 상태로 동작 방식을 변경한다.

## 자바 프록시

- 단순한 상황에 적합하다.
- 개별 객체나 클래스에서 메서드 호출을 감싸는 경우에 좋다.

```java
// 11-3. JDK 프록시 예제
// Bank.java
public interface Bank {
	Collection<Account> getAccounts();
	void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
// 추상화를 위한 POJO(Plain Old Java Object) 구현
public class BankImpl implements Bank {
	private List<Account> accounts;

	public Collection<Account> getAccounts() {
		return accounts;
	}
	public void setAccounts(Collection<Account> accounts) {
		this.accounts = new ArrayList<Account>();
		for (Account account: accounts) {
			this.accounts.add(account);
		}
	}
}

// BankProxyHandler.java
// 프록시 API가 필요한 "InvocationHandler"
public class BankHandler implements InvocationHandler {
	private Bank bank;

	public BankProxyHandler(Bank bank) {
		this.bank = bank;
	}

	// InvocationHandler에 정의된 메서드
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		String methodName = method,getName();
		if(methodName.equals("getAccounts")) {
			bank.setAccounts(getAccountsFromDataBase());
			return bank.getAccounts();
		} else if // ...
	}
}

// 다른 곳에 위치하는 코드
Bank bank = (Bank) Proxy.newProxyInstance(
	Bank.class.getClassLoader(),
	new Class[] { Bank.class },
	new BankProxyHandler(new BankImpl()));
```

- 프록시로 감쌀 인터페이스 Bank와 비즈니스 논리를 구현하는 POJO BankImpl 정의
- 프록시 API에는 InvocationHandler를 넘겨줌
- 넘긴 InvocationHandler는 프록시에 호출되는 Bank 메서드를 구현하는 데 사용됨
- BankProxyHandler는 자바 리플렉션 API를 사용해 제네릭스 메소드를 상응하는 BankImpl 메서드로 매핑함

→ 복잡..

→ 프록시를 사용하면 깨끗한 코드를 작성하기 어렵다. 

→ 또한 시스템 단위로 실행 ‘지점'을 명시하는 메커니즘도 제공하지 않는다.

## 순수 자바 AOP 프레임워크

- 프록시 코드는 도구로 자동화할 수 있다.
- 스프링 AOP, JBoss AOP 등 여러 자바 프레임워크는 내부적으로 프록시를 사용한다.
- 스프링
    - 비즈니스 논리를 POJO로 구현한다.
    - POJO는 순수하게 도메인에 초점을 맞춘다.
    - POJO는 엔터프라이즈 프레임워크에 의존하지 않는다.
    - 따라서 테스트가 개념적으로 더 쉽고 간단하다.
- 프로그래머는 설정 파일이나 API를 사용해 필수적인 애플리케이션 기반 구조를 구현한다.
    - 영속성, 트랜잭션, 보안, 캐시, 장애조치 등 횡단 관심사도 포함
    - 프레임워크는 사용자 모르게 프록시나 바이트코드 라이브러리를 사용해 이를 구현한다.
- 이런 선언들이 요청에 따라 주요 객체를 생성하고 연결하는 등 DI 컨테이너의 구체적인 동작을 제어한다.

// 11-4. 스프링 2.x 설정파일

Bank 도메인 객체는 자료 접근자 객체(Data Accessor Object)로 프록시되었음

자료 접근자 객체는 JDBC 드라이버 자료 소스로 프록시되었음

- 클라이언트는 Bank 객체에서 getAccounts()를 호출한다고 믿지만 실제로는 Bank POJO의 기본 동작을 확장한 중첩 DECORATOR 객체 집합의 가장 외곽과 통신한다.

ex. 애플리케이션에서 DI 컨테이너에게 시스템 내 최상위 객체를 요청하는 코드

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

- 스프링과 독립적
    - 스프링 관련 자바 코드가 거의 필요 없음
    - EJB2 시스템의 강한 결합 문제가 모두 사라짐
- xml은 장황하고 읽기 어렵지만 설정 파일에 명시된 ‘정책'이 프록시나 관점 논리 보다는 단순하다.
- EJB3는 XML 설정 파일과 자바 5 애너테이션 기능을 사용해 횡단 관심사를 선언적으로 지원하는 스프링 모델을 따른다.

```java
// 11-5. EBJ3 Bank EJB
@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
	@Id @GeneratedValue(strategy=GenerationType.AUTO)
	private int id;

	@Embeddable  // Bank의 데이터베이스 행에 '인라인으로 포함된' 객체
	public class Address {
		protected String streetAddr1;
		protected String streetAddr2;
		protected String city;
		protected String state;
		protected String zipcode;
	}

	@Embedded
	private Address address;

	@OneToMany(cascade = CascadeType.All, fetch = FetchType.EAGER, mappedBy = "bank")
	private Collection<Account> accounts = new ArrayList<Account>();
	
	public int getId() {
		return id;
	}
	
	public void setId(int id) {
		this.id = id;
	}

	public void addAccount(Account account) {
		account.setBank(this);
		accounts.add(account);
	}

	public Collection<Account> getAccounts() {
		return accounts;
	}
	
	public void setAccounts(Collection<Account> accounts) {
		this.accounts = accounts;
	}
}
```

- EJB2 코드보다 훨씬 더 깨끗하다.
    - 모든 정보다 애너테이션 속에 있어서 코드 자체가 깔끔하다.
    - 코드를 테스트하고 개선하고 보수하기가 쉬워졌다.

## AspectJ 관점

- AspectJ 언어
    - 언어 차원에서 관점을 모듈화 구성으로 지원하는 자바 언어 확장
    - 새 도구를 사용하고 새 언어 문법과 사용법을 익혀야 한다는 단점이 있다.
    - 최근에 나온 AspectJ ‘애너테이션 폼'은 자바 코드에 자바 5 애너테이션을 사용해 관점을 정의하여 새로운 언어라는 부담을 완화한다.

## 테스트 주도 시스템 아키텍처 구축

- 최선의 시스템 구조는 각기 POJO 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다.
- 이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다.
- 이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.

## 의사 결정을 최적화하라

- 관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다.
- 이런 기민함 덕분에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다.
- 또한 결정의 복잡성도 줄어든다.

## 명백한 가치가 있을 때 표준을 현명하게 사용하라

- 표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 캡슐화하기 쉽고, 컴포넌트를 엮기 쉽다.
- 하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다.
- 어떤 표준은 원래 표준을 제정한 목적을 잊어버리기도 한다.

## 시스템은 도메인 특화 언어가 필요하다

- 최근 조명 받는 DSL은 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다.
- DSL로 짠 코드는 도메인 전문가가 작성한 구조적인 산문처럼 읽힌다.
- 좋은 DSL은 도메인 개념과 개념을 구현한 코드 사이에 존재하는 ‘의사소통 간극'을 줄여준다.
- 도메인 특화 언어를  사용하면 고차원 정책에서 저차원 세부사항에 이르기까지 모든 추상화 수준과 모든 도메인을 POJO로 표현할 수 있다.

## 결론

- 시스템은 깨끗해야 한다.
- 모든 추상화 단계에서 의도는 명확히 표현해야 한다.
    - 그러려면 POJO를 작성하고 관점 메커니즘을 사용해 각 구현 관심사를 분리해야한다.
- 시스템 설계든 개별 모듈 설계든 **실제로 돌아가는 가장 단순한 수단**을 사용해야 한다!
