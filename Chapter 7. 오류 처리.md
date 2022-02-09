# Chapter 7. 오류 처리

오류 처리는 프로그램에서 중요하며 오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워지지 않도록 깨끗한 코드를 만들어야 한다.

## 오류 코드보다 예외를 사용하라

- 예외를 지원하지 않는 프로그래밍 언어 → 오류 처리 방법이 제한적, 호출자 코드가 복잡해짐
    - 오류 플래그 설정 또는 호출자에게 오류코드 반환
    - 논리와 오류 처리 코드가 뒤섞임

```java
public class DeviceController {
	public void sendShutDown() {
		DeviceHandle handle = getHandle(DEV1);
		// 디바이스 상태 점검
		if(handle != DeviceHandle.INVALID) {
			// 레코드 필드에 디바이스 상태 저장
			retrieveDeviceRecord(handle);
			// 디바이스가 일시정지 상태가 아니라면 종료
			if(record.getStatus() != DEVICE_SUSPENDED) {
				pauseDevice(handle);
				clearDeviceWorkQueue(handle);
				closeDevice(handle);
			} else {
				logger.log("Device suspended. Unable to shut down");
			}
		} else {
			logger.log("Invalid handle for: " + DEV1.toString());
		}
	}
}
```

- 예외를 지원하는 프로그래밍 언어 → 오류가 발생하면 예외를 던짐, 호출자 코드가 깔끔해짐
    - 디바이스를 종료하는 알고리즘과 오류 처리 알고리즘이 분리됨

```java
public class DeviceController {
	public void sendShutDown() {
		try {
			tryToShutDown();
		} catch(DeviceShutDownError e) {
			logger.log(e);
		}
	}

	private void tryToShutDown() throws DeviceShutDownError {
		DeviceHandle handle = getHandle(DEV1);
		DeviceRecord record = retrieveDeviceRecord(handle);

		pauseDevice(handle);
		clearDeviceWorkQueue(handle);
		closeDevice(handle);
	}

	private DeviceHandle getHandle(DeviceID id) {
		// ...
		throw new DeviceShutDownError("Invalid handle for: " + id.toString());
		// ...
	}

	private DeviceRecord retrieveDeviceRecord(DeviceHandle handle) {
		// ...
		throw new DeviceShutDownError("Device suspended. Unable to shut down");
		// ...
	}
}
```

## Try-Catch-Finally 문부터 작성하라

- 예외에서 프로그램 안에다 **범위를 정의**한다는 것에 주목하자.
- try-catch-finally 문
    - try 블록에서 어느 시점에서든 실행이 중단된 후 catch 블록으로 넘어갈 수 있음
    - try 블록은 트랜잭션과 비슷
        - try 블록에서 무슨 일이 생기든지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 하기 때문
    - 예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하자.

ex. 파일을 열어 객체 몇 개를 읽어들이는 코드

- 예외를 던지지 않는 코드 → 단위 테스트 실패

```java
// 파일이 없으면 예외를 던지는지 알아보는 단위 테스트
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
	sectionStore.retrieveSection("invalid - file");
}

public List<RecordedGrip> retrieceSection(String sectionName) {
	// 실제로 구현할 때까지 비어 있는 더미를 반환
	return new ArrayList<RecordedGrip>();
}
```

- 예외를 던지는 코드 → 단위 테스트 성공
- 먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법을 권장

```java
public List<RecordedGrip> retrieceSection(String sectionName) {
	try {
			FileInputStream stream = new FileInputStream(sectionName);
			stream.close();
		} catch(FileNotFoundException e) {
			throw new StorageException("retrieval error", e);
		}
	return new ArrayList<RecordedGrip>();
}
```

## 미확인 예외를 사용하라

- 확인된 예외
    - 메서드를 선언할 때 메스드가 반환할 예외를 모두 열거함
    - OCP(Open Closed Principle) 위반
        - 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 함
    - 캡슐화가 깨진다.
        - 최하위 함수를 변경해 새로운 오류를 던지면 함수는 선언부에 throws 절을 추가해야 함
        - 변경된 함수를 호출하는 함수 모두 catch블록에서 예외를 처리하거나 선언부에 throw절을 추가해야함
        - 최하위 단계에서 최상위 단계까지 연쇄적인 수정이 일어남
        - throws 경로에 위치하는 모든 함수가 최하위 함수에서 던지는 예외를 알아야 하므로 캡슐화가 깨짐

## 예외에 의미를 제공하라

- 예외를 던질 때 전후 상황을 충분히 덧붙이면 오류가 발생한 원인과 위치를 찾기가 쉬워진다.
    - 오류 메시지에 정보(실패한 연산 이름, 실패 유형 등)를 담아 예외와 함께 던져라.
    - 로깅 기능을 사용한다면 catch 블록에서 오류를 기록하도록 충분한 정보를 넘겨라.

cf. 자바는 모든 예외에 호출 스택을 제공함, 그러나 실패한 코드의 의도를 찾기엔 부족함

## 호출자를 고려해 예외 클래스를 정의하라

- 오류를 정의할 때 프로그래머는 **오류를 잡아내는 방법**을 중요하게 생각해야 한다.
- 대다수 상황에서 오류를 처리하는 방식 → 오류를 일으킨 원인과 무관하게 일정함
    - 오류 기록
    - 프로그램을 계속 수행해도 좋은지 확인

```java
// 오류를 형편없이 분류한 사례
// 외부 라이브러리가 던질 예외를 모두 잡아내는 코드
ACMEPort port = new ACMEPort(12);

try {
	port.open();
} catch(DeviceResponseException e) {
	reportPortError(e);
	logger.log("Device response exception", e);
} catch(ATM1212UnlockedException e) {
	reportPortError(e);
	logger.log("Unlock exception", e);
} catch(GMXError e) {
	reportPortError(e);
	logger.log("Device response exception");
} finally {
	// ...
}
```

- **감싸기 기법**
    - LocalPort 클래스는 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 감싸기(wrapper) 클래스
    - 외부 API를 사용할 때는 감싸기 기법이 최선!
    - 감싸기 기법의 장점
        - 외부 API를 감싸면 외부 라이브러리와 프로그램 사이의 의존성이 크게 줄어든다.
        - 다른 라이브러리로 갈아타도 비용이 적다.
        - 감싸기 클래스에서 외부 API 호출 대신 테스트 코드를 넣어주는 방법으로 테스트할 수 있다.
        - 특정 업체가 API를 설계한 방식에 발목 잡히지 않는다. → 프로그램이 사용하기 편리한 API를 정의하면 되기 때문
            
            ex. PortDeviceFailure 예외 유형
            

```java
// 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환하기
LocalPort port = new LocalPort(12);
try {
	port.open();
} catch(PortDeviceFailure e) {
	reportError(e);
	logger.log(e.getMessage(), e);
} finally {
	// ...
}

public class LocalPort {
	private ACMEPort innerPort;
	
	public LocalPort(int portNumber) {
		innerPort = new ACMEPort(portNumber);
	}

	public void open() {
		try {
			innerPort.open();
		} catch(DeviceResponseException e) {
			throw new PortDeviceFailure(e);
		} catch(ATM1212UnlockedException e) {
			throw new PortDeviceFailure(e);
		} catch(GMXError e) {
			trhow new PortDeviceFailure(e);
		}
	}
}
```

- 예외 클래스가 하나만 있어도 충분한 코드가 많다.
    - 예외 클래스에 포함된 정보로 오류를 구분해도 괜찮은 경우
- 한 예외는 잡아내고 다른 예외는 무시해도 괜찮은 경우에는 여러 예외 클래스 사용

## 정상 흐름을 정의하라

- 외부 API를 감싸 독자적인 예외를 던지고, 코드 위에 처리기를 정의해 중단된 계산을 처리하는 방식

→ 때로는 중단이 적합하지 않은 경우도 있다.

ex. 비용 청구 애플리케이션에서 총계를 계산하는 코드

- 특수 상황을 처리할 필요가 없음!

```java
try {
	MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
	// 식비를 비용으로 청구한 경우, 청구한 식비를 총계에 더하기
	m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
	// 식비를 비용으로 청구하지 않은 경우, 일일 기본 식비를 총계에 더하기
	m_total += getMealPerDiem();
}
```

- ExpenseReportDAO가 청구한 식비가 없다면 일일 기본 식비를 반환하는 MealExpense 객체를 반환하도록 수정!

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

public class PerDiemMealExpenses implements MealExpense {
	public int getTotal() {
		// 기본값으로 일일 기본 식비 반환
	}
}
```

→ **특수 사례 패턴**

- 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식
- 클래스나 객체가 예외적인 상황을 캡슐화해서 처리하므로 클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다.

## null을 반환하지 마라

- null을 반환하는 습관
    - 오류를 유발하는 행위
    - 나쁜 코드

```java
// null을 확인하는 코드로 가득한 애플리케이션
public void registerItem(Item item) {
	if(item != null) {
		ItemRegistry registry = persistentStore.getItemRegistry();
		if(registry != null) {
			Item existing = registry.getItem(item.getID());
			if(existing.getBillingPeriod().hasRetailOwner()) {
				existing.register(item);
			}
		}
	}
}
```

- null 반환 해결 방법
    - 예외를 던지거나 **특수 사례 객체**를 반환한다. (많은 경우 해결책이 됨)
    - 사용하려는 외부 API가 null을 반환한다면 감싸기 메서드를 구현해 예외를 던지거나 특수 사례 객체를 반환하는 방식을 고려한다.
- 특수 사례 객체 예시

```java
List<Employee> employees = getEmployees();
if(employees != null) {
	for(Employee e: employees) {
		totalPay += e.getPay();
	}
}
```

→ getEmployees()가 null 대신 빈 리스트를 반환한다면 코드가 훨씬 깔끔해짐

```java
List<Employee> employees = getEmployees();
for(Employee e: employees) {
	totalPay += e.getPay();
}

public List<Employee> getEmployees() {
	if(직원이 없다면){
		// 미리 정의된 읽기 전용 리스트 반환
		return Collections.emptyList();
	}
}
```

→ NullPointerException이 발생할 가능성도 줄어듦

## null을 전달하지 마라

- 메서드로 null을 전달하는 방식은 더 나쁘다.

ex. 두 지점 사이의 거리를 계산하는 메서드

```java
public class MetricsCalculator {
	public double xProjection(Point p1, Point p2) {
		return (p2.x - p1.x) * 1.5;
	}
}

// NullPointerException
calculator.xProjection(null, new Point(12, 13));
```

- 해결 방법
    - 새로운 예외 유형을 만들어 던지기 → InvalidArgumentException을 잡아내는 처리기가 필요함
    
    ```java
    public class MetricsCalculator {
    	public double xProjection(Point p1, Point p2) {
    		if(p1 == null || p2 == null) {
    			throw InvalidArgumentException(
    				"Invalid argument for MetricsCalculator.xProjection");
    		}
    		return (p2.x - p1.x) * 1.5;
    	}
    }
    ```
    
    - assert문 사용하기
    
    ```java
    public class MetricsCalculator {
    	public double xProjection(Point p1, Point p2) {
    		assert p1 != null : "p1 should not be null";
    		assert p2 != null : "p2 should not be null";
    		return (p2.x - p1.x) * 1.5;
    	}
    }
    ```
    

그러나!! 문제를 해결하지는 못함.. → 애초에 null을 넘기지 못하도록 금지해야함

인수로 null이 넘어오면 코드에 문제가 있다는 말..

## 결론

- 깨끗한 코드는 읽기도 좋아야 하지만 **안정성**도 높아야 한다.
- **오류 처리를 프로그램 논리와 분리**해야 한다. → 독립적인 추론 가능, 코드 유지보수성 높아짐
