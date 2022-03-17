# Chapter 17. 냄새와 휴리스틱

## 주석

- C1: 부적절한 정보
    - 다른 시스템(소스 코드 관리 시스템, 버그 추적 시스템, 이슈 추적 시스템, 기타 기록 관리 시스템)에 저장할 정보는 주석으로 적절하지 못하다.
    - 일반적으로 작성자, 최종 수정일, SPR 번호 등 메타 정보만 주석으로 넣는다.
- C2: 쓸모 없는 주석
    - 오래된 주석, 엉뚱한 주석, 잘못된 주석은 쓸모가 없다.
    - 빨리 삭제하는게 좋으며, 아예 달지 않는 편이 가장 좋다.
- C3: 중복된 주석
    - 코드만으로 충분한데 구구절절 설명하는 주석이다.
    
    ```swift
    i++; // i 증가
    
    /*
    * @param sellRequest
    * @return
    * @throws ManagedComponentException
    */
    public SellResponse beginSellItem(SellRequest sellRequest) throws ManagedComponentException
    ```
    
- C4: 성의 없는 주석
    - 주석을 달 것이라면 시간을 들여 최대한 멋지게 작성한다.
- C5: 주석 처리된 코드
    - 주석으로 처리된 코드를 발견하면 즉각 지워라.

## 환경

- E1: 여러 단계로 빌드해야 한다
    - 빌드는 간단히 한 단계로 끝나야 한다.
    - 한 명령으로 전체를 체크아웃해서 한 명령으로 빌드할 수 있어야 한다.
    
    ```c
    svn get mySystem
    cd mySystem
    ant all
    ```
    
- E2: 여러 단계로 테스트해야 한다
    - 모든 단위 테스트는 한 명령으로 돌려야 한다.
    - IDE에서 버튼 하나로 모든 테스트를 돌린다면 가장 이상적이다.

## 함수

- F1: 너무 많은 인수
    - 함수에서 인수 개수는 작을수록 좋으며, 아예 없으면 가장 좋다.
    - 넷 이상은 최대한 피한다. (p. 50 “함수 인수” 참조)
- F2: 출력 인수
    - 출력 인수는 직관을 정면으로 위배한다.
    - 인수를 출력이 아니라 입력으로 간주하는데, 상태를 변경해야 한다면 출력 인수를 쓰지 말고 함수가 속한 객체의 상태를 변경해라. (p. 56 “출력 인수" 참조)
- F3: 플래그 인수
    - boolean 인수는 함수가 여러 기능을 수행한다는 명백한 증거다.
    - 플래그 인수는 피해야 마땅하다. (p. 52 “플래그 인수" 참조)
- F4: 죽은 함수
    - 아무도 호출하지 않는 함수는 삭제한다.

## 일반

- G1: 한 소스 파일에 여러 언어를 사용한다
    - 오늘날 프로그래밍 환경은 한 소스 파일 내에서 다양한 언어를 지원한다.
    - 이상적으로는 소스 파일 하나에 언어 하나만 사용하는 방식이 가장 좋다.
    - 현실적으로는 여러 언어가 불가피하지만, 언어 수와 범위를 최대한 줄이도록 하자.
- G2: 당연한 동작을 구현하지 않는다
    - 최소 놀람의 원칙(The Principle Of Least Surprise)에 의거해 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다.
    
    ```java
    Day day = DayDate.StringToDay(String dayName);
    ```
    
    - 당연한 동작을 구현하지 않으면 코드를 읽거나 사용하는 사람이 더 이상 함수 이름만으로 함수 기능을 직관적으로 예상하기 어렵다.
- G3: 경계를 올바로 처리하지 않는다
    - 코드는 올바로 동작해야 하므로 모든 경계 조건을 찾아내고, 모든 경계 조건을 테스트하는 테스트 케이스를 작성하라.
    - 자신의 직관에 의존하여 경계와 구석진 곳을 간과하지 마라.
- G4: 안전 절차 무시
    - serialVersionUID를 직접 제어할 필요가 있을지도 모르지만 직접 제어는 언제나 위험하다.
    - 컴파일러 경고를 끄고 빌드하지 말자.
    - 실패하는 테스트 케이스를 나중으로 미루지 말자.
- G5: 중복 🖤
    - DRY(Don’t Repeat Yourself) 원칙: 데이비드 토머스, 앤디 헌트
    - 한 번, 단 한 번만(Once, and only once”): 켄트 벡
    - 모든 테스트를 통과한다는 규칙 다음으로 중요: 론 제프리스
    - 코드에서 중복을 발견할 때마다 추상화할 기회로 간주하라. 하위 루틴이나 다른 클래스로 분리하라.
        - 똑같은 코드가 여러 차례 나오는 중복 → 함수로 교체
        - 여러 모듈에서 일련의 switch/case 나 if/else 문으로 똒같은 조건을 거듭 확인하는 중복 → 다형성으로 대체
        - 알고리즘이 유사하나 코드가 서로 다른 중복 → Template Method 패턴이나 Strategy 패턴으로 중복 제거
    - 어디서든 중복을 발견하면 없애라!!
- G6: 추상화 수준이 올바르지 못하다
    - 추상화는 저차원 상세 개념에서 고차원 일반 개념을 분리한다.
        - ex. 추상 클래스(고차원 개념)와 파생 클래스(저차원 개념)
    - 모든 저차원 개념은 파생 클래스에 넣고, 모든 고차원 개념은 기초 클래스에 넣는다. → 철저하게 분리
        - ex. 세부 구현 관련 상수, 변수, 유틸리티 함수는 기초 클래스에 넣으면 안 됨
        - 기초 클래스는 구현 정보에 무지해야 함
    - 소스 파일, 컴포넌트, 모듈도 철저히 분리해 다른 컨테이너에 넣는다.
    - 고차원 개념과 저차원 개념을 섞어서는 안 된다!
    
    ```java
    public interface Stack {
    	Object pop() throws EmptyException;
    	void push(Object o) throws FullException;
    	double percentFull();
    	class EmptyException extends Exception {}
    	class FullException extends Exception {}
    }
    // percentFull(): 추상화 수준이 올바르지 못함 -> BoundedStack 같은 파생 인터페이스에 넣기
    
    // 크기가 무한한 스택은 0을 반환하면 되지 않는가? -> No!
    stack.percentFull() < 50.0;
    // OutOfMemoryException 예외가 절대 발생하지 않으리라 장담하지 못함
    ```
    
    - 잘못된 추상화 수준은 거짓말이나 꼼수로 해결하지 못한다.
    - 어려운 작업이므로 주의하자!
- G7: 기초 클래스가 파생 클래스에 의존한다
    - 기초 클래스는 파생 클래스를 아예 몰라야 한다.
    - 개념을 기초 클래스와 파생 클래스로 나누는 이유: 고차원 기초 클래스 개념을 저차원 파생 클래스 개념으로부터 분리해 독립성을 보장하기 위해서
        - 예외: 파생 클래스 개수가 확실히 고정되어있는 경우에 기초 클래스에 파생 클래스를 선택하는 코드가 들어감(FSM: Finite State Machine)
    - 기초 클래스와 파생 클래스를 다른 JAR 파일로 배포하면 변경이 시스템에 미치는 영향이 작아지므로 유지보수하기 수월해진다.
- G8: 과도한 정보
    - 잘 정의된 모듈은 인터페이스가 아주 작다.
    - 작은 인터페이스로도 많은 동작이 가능하다.
    - 잘 정의된 인터페이스는 많은 함수를 제공하지 않기 때문에 결합도가 낮다.
    - 클래스가 제공하는 함수, 변수 수는 작을수록 좋다.
    - 자료, 유틸리티 함수, 상수, 임시 변수를 숨겨라.
    - 인터페이스를 매우 작고 깐깐하게 만들어서 정보를 제한해 결합도를 낮춰라!
- G9: 죽은 코드
    - 실행되지 않는 코드를 가리킨다.
    - ex. 불가능한 조건을 확인하는 if 문, throw 문이 없는 try문에서 catch 블록, 불가능한 case 조건
    - 죽은 코드는 발견하면 제거하라.
- G10: 수직 분리
    - 변수와 함수는 사용되는 위치에 가깝게 정의한다.
    - 지역 변수는 처음으로 사용하기 직전에 선언하며 수직으로 가까운 곳에 위치한다.
    - 비공개 함수는 처음으로 호출한 직후에 정의한다.
- G11: 일관성 부족
    - 어떤 개념을 특정 방식으로 구현했다면 유사한 개념도 같은 방식으로 구현한다. (최소 놀람의 원칙에도 부합)
    - ex. 한 함수에서 response 변수에 HttpServletResponse 인스턴스를 저장 → 다른 함수에서도 일관성 있게 동일한 변수 이름 사용
    - 간단한 일관성만으로도 코드를 읽고 수정하기 쉬워진다.
- G12: 잡동사니
    - 비어있는 생성자, 아무도 사용하지 않는 변수, 함수, 주석 등은 제거해라.
- G13: 인위적 결합
    - 서로 무관한 개념을 인위적으로 결합하지 않는다.
    - ex. 일반적인 enum은 특정 클래스에 속할 이유가 없음, 범용 static 함수도 마찬가지
    - 함수, 상수, 변수를 선언할 때는 올바른 위치를 고민하라.
- G14: 기능 욕심
    - 마틴 파울러가 말하는 코드 냄새 중 하나
    - 클래스 메서드는 자기 클래스의 변수와 함수에 관심을 가져야지 다른 클래스의 변수와 함수에 관심을 가져서는 안 된다.
    - 메서드가 다른 객체의 참조자, 변경자를 사용해 그 객체 내용을 조작하는 행위는 그 객체 클래스의 범위를 욕심내는 것이다.
    
    ```java
    // calculateWeeklyPay 메서드는 HourlyEmployee 객체에서 온갖 정보를 가져옴
    // calculateWeeklyPay 메서드는 HourlyEmployee 클래스 범위를 욕심냄
    public class HourlyPayCalculator {
    	public Money calculateWeeklyPay(HourlyEmployee e) {
    		int tenthRate = e.getTenthRate().getPennies();
    		int tenthsWorked = e.getTenthsWorked();
    		int straightTime = Math.min(400, tenthsWorked);
    		int overTime = Math.max(0, tenthsWorked - straigntTime);
    		int straightPay = straightTime * tenthRate;
    		int overtimePay = (int)Math.round(overTime * tenthRate * 1.5);
    		return new Money(straightPay + overtimePay);
    	}
    }
    ```
    
    - 어쩔 수 없는 경우
    
    ```java
    // reportHours 메서드는 HourlyEmployee 클래스를 욕심냄
    // HourlyEmployee 클래스가 보고서 형식을 알 필요 없음
    // 하지만 함수를 HourlyEmployee로 옮기면 SRP, OCP, CCP 위반
    // 보고서 형식이 바뀌면 클래스도 바뀌기 때문
    public class HourlyEmployeeReport {
    	private HourlyEmployee employee;
    
    	public HourlyEmployeeReport(HourlyEmployee e) {
    		this.employee = e;
    	}
    
    	String reportHours() {
    		return String.format(
    			"Name: %s\tHours:%d.%1d\n",
    			employee.getName(),
    			employee.getTenthsWorked()/10,
    			employee.getTenthsWorked()%10);
    	}
    }
    ```
    
- G15: 선택자 인수
    - 선택자 인수는 목적을 기억하기 어려울 뿐 아니라 각 선택자 인수가 여러 함수를 하나로 조합한다.
    
    ```java
    // bad
    // 초과근무 수당을 1.5배로 지급하면 true, 아니면 false
    public int calculateWeeklyPay(boolean overtime) {
    		int tenthRate = getTenthRate();
    		int tenthsWorked = getTenthsWorked();
    		int straightTime = Math.min(400, tenthsWorked);
    		int overTime = Math.max(0, tenthsWorked - straigntTime);
    		int straightPay = straightTime * tenthRate;
    		double overtimeRate = overtime ? 1.5 : 1.0 * tenthRate;
    		int overtimePay = (int)Math.round(overTime * overtimeRate);
    		return straightPay + overtimePay;
    }
    ```
    
    ```java
    // good
    public int straightPay() {
    	return getTenthsWorked() * getTenthRate();
    }
    
    public int overTimePay() {
    	int overTimeTenths = Math.max(0, getTenthsWorked() - 400);
    	int overTimePay = overTimeBonus(overTimeTenths);
    	return straightPay() + overTimePay;
    }
    
    private int overTimeBonus(int overTimeTenths) {
    	double donus = 0.5 * getTenthRate() * overTimeTenths;
    	return (int)Math.round(bonus);
    }
    ```
    
    - boolean, enum, int 등 함수 동작을 제어하려는 인수는 바람직하지 않음
    - 인수를 넘겨 동작을 선택하는 대신 새로운 함수를 만드는 편이 좋다.
- G16: 모호한 의도
    - 코드를 짤 때는 의도를 최대한 분명히 밝힌다.
    
    ```java
    // bad
    // overTimePay 함수
    public int m_otCalc() {
    	return iThsWkd * iThsRte +
    		(int)Math.round(0.5 * iThsRte *
    		Math.max(0, iThsWkd - 400)
    		);
    }
    ```
    
- G17: 잘못 지운 책임
    - 코드를 배치하는 위치는 중요하다.
    - ex. PI 상수는 어디에 들어갈까? → 삼각함수를 선언한 클래스
    - ex. OVERTIME_RATE 상수 → HourlyPayCalculator 클래스에 선언
    - 최소 놀람의 원칙 (The Principle of Least Surprise) 적용
        - 코드는 독자가 자연스럽게 기대할 위치에 배치
    - 때로는 개발자에게 편한 함수에 배치
    - 또는 함수 이름을 기준으로 결정하기
- G18: 부적절한 static 함수
    - Math.max(double a, double b)는 좋은 static 메서드다. 특정 인스턴스와 관련된 기능이 아니며, 두 인수 정보만 사용함
    - Math.max 함수를 재정의 할 가능성은 전혀 없다.
    - 간혼 우리는 static으로 정의하면 안 되는 함수를 static으로 정의한다.
    
    ```java
    // bad
    // 재정의 할 가능성 존재 -> 수당을 계산하는 알고리즘이 여러 개일지도 모름
    // Employee 클래스에 속하는 인스턴스 함수여야 함
    HourlyPayCalculator.calculatePay(employee, overtimeRate);
    ```
    
    - 일반적으로 static 함수보다 인스턴스 함수가 더 좋음
- G19: 서술적 변수
    - 켄트 벡이 지적한 문제
    - 프로그램 가독성을 높이는 가장 효과적인 방법 중 하나는 계산을 여러 단계로 나누고 중간 값으로 서술적인 변수 이름을 사용하는 방법이다.
    
    ```java
    // good
    // FitNess
    Matcher match = headerpattern.matcher(line);
    if(match.find()) {
    	String key = match.group(1);
    	String value = match.group(2);
    	headers.put(key.toLowercase(), value);
    }
    ```
    
    - 서술적인 변수 이름은 많이 써도 괜찮다. 해독하기 어렵던 모듈이 읽기 쉬운 모듈로 바뀐다.
- G20: 이름과 기능이 일치하는 함수
    
    ```java
    // bad
    // 5일을 더하는 함수? 5주? 5시간? date 인스턴스를 변경하는 함수? 새로운 Date를 반환하는 함수?
    Date newDate = date.add(5);
    ```
    
    - date 인스턴스에 5일을 더해 인스턴스를 변경하는 함수라면 addDaysTo 또는 increaseByDays 가 적합
- G21: 알고리즘을 이해하라
    - 알고리즘이 올바르다는 사실을 확인하고 이해하려면 기능이 뻔히 보일 정도로 함수를 깔끔하고 명확하게 재구성하라.
- G22: 논리적 의존성은 물리적으로 드러내라
    - 한 모듈이 다른 모듈에 의존한다면 물리적인 의존성도 있어야 한다.
    - 의존하는 모든 정보를 명시적으로 요청하는게 좋다.
- G23: If/Else 혹은 Switch/Case 문보다 다형성을 사용하라
    - ‘switch 문 하나’ 규칙?
    - 선택 유형 하나에는 switch 문을 한 번만 사용, 같은 선택을 수행하는 다른 코드에서는 다형성 객체를 생성해 switch 문을 대신한다.
- G24: 표준 표기법을 따르라
    - 팀은 업계 표준에 기반한 구현 표준을 따라야 한다.
    - 팀이 정한 표준은 팀원들 모두가 따라야 한다.
    - p. 512 목록 B-7 에서 목록 B-14 참고
- G25: 매직 숫자는 명명된 상수로 교체하라
    - 일반적으로 코드에서 숫자를 사용하지 말라는 규칙
    - 숫자는 명명된 상수 뒤로 숨기라는 의미
    - 어떤 상수는 이해하기가 쉬우므로 코드 자체가 자명하다면 숨길 필요는 없다.
    - ‘매직 숫자'라는 용어는 숫자뿐만 아니라 의미가 분명하지 않은 토큰을 모두 가리킨다.
    
    ```java
    // 매직 숫자: 7777, "John Doe"
    assertEquals(7777, Employee.find("John Doe").employeeNumber());
    ```
    
    ```java
    // 실제 의미
    assertEquals(
    	HOURLY_EMPLOYEE_ID,
      Employee.find(HOURLY_EMPLOYEE_NAME).employeeNumber());
    ```
    
- G26: 정확하라
    - 검색 결과 중 첫 번째 결과만 유일한 결과로 간주하는 행동은 순진하다.
    - ex. 부동소수점으로 통화를 표현하는 행위
    - ex. 갱신할 가능성이 희박하다고 잠금과 트랜잭션 관리를 건너뛰는 행위
    - ex. List로 선언할 변수를 ArrayList로 선언하는 행동
    - 코드에서 뭔가를 결정할 때는 정확히 결정한다. 결정을 내리는 이유와 예외를 처리할 방법을 분명히 알아야 한다.
    - ex. null을 반환할지도 모르는 함수라면 null을 반드시 점검하기
- G27: 관례보다 구조를 사용하라
    - 설계 결정을 강제할 때는 규칙보다 관례를 사용한다. 명명 관례도 좋지만 구조 자체로 강제하면 더 좋다.
    - ex. enum 변수가 switch/case 문보다 추상 메서드가 있는 기초클래스가 더 좋다.?
    - switch/case 문을 매번 똑같이 구현하게 강제하기는 어렵지만, 파생 클래스는 추상 메서드를 모두 구현하지 않으면 안 되기 때문
- G28: 조건을 캡슐화하라
    - 부울 논리는 이해하기 어렵다. 조건의 의도를 분명히 밝히는 함수로 표현하라.
    
    ```java
    // good
    if(shouldBeDeleted(timer))
    
    // bad
    if(timer.hasExpired() && !timer.isRecurrent())
    ```
    
- G29: 부정 조건은 피하라
    - 부정 조건은 긍정 조건보다 이해하기 어렵다. 가능하면 긍정 조건으로 표현한다.
    
    ```java
    // good
    if(buffer.shouldCompact())
    
    // bad
    if(!buffer.shouldNotCompact())
    ```
    
- G30: 함수는 한 가지만 해야 한다
    - 한 함수 안에 여러 단락을 이어 일련의 작업을 수행하지 마라
    - 한 가지만 수행하는 좀 더 작은 함수 여럿으로 나눠야 마땅하다.
    
    ```java
    // bad
    // 1. 직원 목목을 루프로 돈다
    // 2. 각 직원의 월급일을 확인한다.
    // 3. 해당 직원에게 월급을 지급한다.
    public void pay() {
    	for(Employee e: employees) {
    		if(e.isPayDay()) {
    			Money pay = e.calculatePay();
    			e.deliverPay(pay);
    		}
    	}
    }
    ```
    
    ```java
    // good
    // 각 함수는 한 가지 임무만 수행, 3장 참고
    public void pay() {
    	for(Employee e: employees) {
    		payIfNecessary(e);
    	}
    }
    
    private void payIfNecessary() {
    	if(e.isPayDay()) {
    			calculateAndDeliverPay(e);
    	}
    }
    
    private void calculateAndDeliverPay() {
    	Money pay = e.calculatePay();
    	e.deliverPay(pay);
    }
    ```
    
- G31: 숨겨진 시각적인 결합
    - 때로는 시간적인 결합이 필요하다. 하지만 시간적인 결합을 숨겨서는 안 된다.
    - 함수를 짤 때는 함수 인수를 적절히 배치해 함수가 호출되는 순서를 명백히 드러낸다.
    
    ```java
    // bad
    // 시간적인 결합을 강제하지 않음
    public class MoogDiver {
    	Gradient gradient;
    	List<Spline> splines;
    
    	public void dive(String reason) {
    		saturateGradient();  // gradient 처리
    		reticulateSplines();  // splines 처리
    		diveForMoog(reason);
    	}
    	// ...
    }
    ```
    
    ```java
    // good
    // 연결 소자를 생성해 시간적인 결합을 노출함
    // 각 함수가 내놓는 결과는 다음 함수에 필요함
    public class MoogDiver {
    	Gradient gradient;
    	List<Spline> splines;
    
    	public void dive(String reason) {
    		Gradient gradient = saturateGradient();
    		List<Spline> splines = reticulateSplines(gradient);
    		diveForMoog(splines, reason);
    	}
    	// ...
    }
    ```
    
- G32: 일관성을 유지하라
    - 코드 구조를 잡을 때는 이유를 고민하고, 그 이유를 코드 구조로 명백히 표현하라.
    - 시스템 구조가 일관성이 있다면 남들도 일관성을 따르고 보존한다.
    
    ```java
    // bad
    // VariableExpandingWidgetRoot 클래스가 AliasLinkWidget 클래스 범위에 속할 필요가 전혀 없다
    // 다른 클래스의 유틸리티가 아닌 public 클래스는 자신이 아닌 클래스 범위 안에서 선언하면 안 된다. 패키지 최상위 수준에 public 클래스로 선언하는 관례가 일반적이다.
    public class AliasLinkWidget extends ParentWidget {
    	public static class VariableExpandingWidgetRoot {
    		// ...
    	}
    	// ...
    }
    ```
    
- G33: 경계 조건을 캡슐화하라
    - 경계 조건은 놓치기 쉬우므로 한 곳에서 별도로 처리한다. 코드 여기저기에서 처리하지 않는다.
    - 코드에 +1 이나 -1을 흩어놓지 않는다.
    
    ```java
    // bad
    // level + 1는 두 번 나옴. 이런 경계 조건은 변수로 캡슐화하는게 좋다.
    if(level + 1 < tags.length) {
    	parts = new Parse(body, tags, level + 1, offset + endTag);
    	body = null;
    }
    ```
    
    ```java
    // good
    // level + 1는 두 번 나옴. 이런 경계 조건은 변수로 캡슐화하는게 좋다.
    
    int nextLevel = level + 1;
    if(nextLevel < tags.length) {
    	parts = new Parse(body, tags, nextLevel, offset + endTag);
    	body = null;
    }
    ```
    
- G34: 함수는 추상화 수준을 한 단계만 내려가야 한다
    - 함수 내 모든 문장은 추상화 수준이 동일해야 한다.
    - 추상화 수준은 함수 이름이 의미하는 작업보다 한 단계 낮아야 한다.
    
    ```java
    // bad
    // 페이지를 가로질러 수평자를 만드는 HTML 태그 생성하는 함수
    // 수평자 높이는 size 변수로 지정
    // -- 추상화 수준 --
    // 1. 수평선에 크기가 있다는 개념
    // 2. HR 태그 자체의 문법
    // 네 개 이상의 대시(-)를 감지해 HR 태그로 변환, 대시 수가 많을수록 크기는 커짐
    public String render() throws Exceptioin {
    	StringBuffer html = new StringBuffer("<hr>");
    	if(size > 0) {
    		html.append(" size=\"").append(size+1).append("\"");
    	}
    	html.append(">");
    
    	return html.toString();
    }
    ```
    
    ```java
    // good
    // 1. render 함수는 HR 태그만 생성함, HR 태그 문법은 전혀 상관하지 않음
    // 2. HTML 문법은 HtmlTag 모듈이 알아서 치리함
    // 위 코드에서 <hr/>이 아니라 <hr>을 출력한다는 오류도 발견
    public String render() throws Exceptioin {
    	HtmlTag hr = new HtmlTag("<hr>");
    	if(extraDashes > 0) {
    		hr.addAttribute("size", hrSize(extraDashes);
    	}
    	return hr.html();
    }
    
    public String hrSize(int height) {
    	int hrSize = height + 1;
    	reutrn String.format("%d", hrSize);
    }
    ```
    
    - 추상화 수준 분리는 리팩터링을 수행하는 가장 중요한 이유 중 하나!
- G35: 설정 정보는 최상위 단계에 둬라
    - 추상화 최상위 단계에 둬야 할 기본값 상수나 설정 관련 상수를 저차원 함수에 숨겨서는 안 된다.
    - 대신 고차원 함수에서 저차원 함수를 호출할 때 인수로 넘긴다.
    
    ```java
    // good
    // 첫 행은 명령행 인수의 구문을 분석한다. 각 인수 기본값은 Argument 클래스 맨 처음에 나온다.
    // 시스템의 저수준을 뒤질 필요가 없다.
    public static void main(String[] args) throws Exception {
    	Arguments arguments = parseCommandLine(args);
    	// ...
    }
    
    public class Arguments {
    	public static final String DEFAULT_PATH = ".";
    	public static final String DEFAULT_ROOT = "FitNesseRoot";
    	public static final int DEFAULT_PORT = 80;
    	public static final int DEFAULT_VERSION_DAYS = 14;
    	// ...
    }
    ```
    
    - 설정 관련 상수는 최상위 단계에 둔다. 변경하기도 쉽다.
    - 설정 관련 변수는 나머지 코드에 인수로 넘긴다. 저차원 함수에 상수 값을 정의하면 안 된다.
- G36: 추이적 탐색을 피하라
    - 일반적으로 한 모듈은 주변 모듈을 모를수록 좋다.
    - A가 B를 사용하고 B가 C를 사용해도 A가 C를 알아야 할 필요는 없다.
        - ex. a.getB().getC().doSomething(); // bad
    - 디미터의 법칙(Law of Demeter)
    - 부끄럼 타는 코드 작성(Writing Shy Code)
    - 자신이 직접 사용하는 모듈만 알아야 한다는 뜻
    - a.getB().getC()를 → a.getB().getQ().getC()로 바꾸려면 a.getB().getC()를 찾아 모두 바꿔야 한다.
    - 내가 사용하는 모듈이 내게 필요한 서비스를 모두 제공해야 한다. 원하는 메서드를 찾느라 시스템을 탐색할 필요가 없어야 한다.
        - ex. myCollaborator.doSomething();  // good

## 자바

- J1: 긴 import 목록을 피하고 와일드카드를 사용하라
    - 패키지에서 클래스를 둘 이상 사용한다면 와일드카드를 사용해 패키지 전체를 가져오라.
        - ex. import package.*;
- J2: 상수는 상속하지 않는다
    - 대신 static import를 사용하라.
- J3: 상수 대 Enum
    - 자바 5는 enum을 제공한다. 마음껏 활용하라!
    - public static final int는 옛날 기교다.
    - enum은 메서드와 필드도 사용할 수 있다. int보다 훨씬 더 유연하고 서술적인 강력한 도구다.

## 이름

- N1: 서술적인 이름을 사용하라
    - 이름은 성급하게 정하지 않으며, 서술적인 이름을 신중하게 고른다.
    - 신중하게 선택한 이름을 보고 독자는 모듈 내 다른 함수가 하는 일을 짐작한다.
    
    ```java
    // bad
    public int x() {
    	int q = 0;
    	int z = 0;
    	for(int kk = 0; kk < 10; kk++) {
    		if(l[z] == 10) {
    			q += 10 + (l[z+1] + l[z+2]);
    			z += 1;
    		} else if(l[z] + l[z+1] == 10) {
    			q += 10 + l[z+2];
    			z += 2;
    		} else {
    			q += l[z] + l[z+1];
    			z += 2;
    		}
    	}
    	return q;
    }
    ```
    
    ```java
    // good
    // 의도가 금방 드러나며, 의도를 유추해 빠진 함수를 채워 넣기도 쉽다.
    public int score() {
    	int score = 0;
    	int frmae = 0;
    	for(int frameNumber = 0; frameNumber < 10; frameNumber++) {
    		if(isStrike(frmae)) {
    			score += 10 + nextTwoBallsForStrike(frame);
    			frmae += 1;
    		} else if(isSpare(frame)) {
    			score += 10 + nextBallForSquare(frame);
    			frmae += 2;
    		} else {
    			score += twoBallsInFrame(frame);
    			frmae += 2;
    		}
    	}
    	return score;
    }
    
    // isStrike() 함수가 하는 일을 짐작할 수 있음
    private boolean isStrike(int frame) {
    	return rolls[frame] = 10;
    }
    ```
    
- N2: 적절한 추상화 수준에서 이름을 선택하라
    - 구현을 드러내는 이름은 피하라.
    - 작업 대상 클래스나 함수가 위치하는 추상화 수준을 반영하는 이름을 선택하라.
    - 추상화 수준이 너무 낮은 변수 이름은 발견하면 바꿔라.
    
    ```java
    public interface Modem {
    	boolean dial(String phoneNumber);
    	boolean disconnect();
    	boolean send(char c);
    	char recv();
    	String getConnectedPhoneNumber();
    }
    ```
    
    ```java
    // good
    // 연결 대상의 이름을 더 이상 전화번호로 제한하지 않는다.
    // 전화번호는 물론이고 다른 연결 방식에도 사용 가능하다.
    public interface Modem {
    	boolean connect(String connectionLocator);
    	boolean disconnect();
    	boolean send(char c);
    	char recv();
    	String getConnectedLocator();
    }
    ```
    
- N3: 가능하다면 표준 명명법을 사용하라
    - ex. DECORATOR 패턴을 활용한다면 장식하는 클래스 이름에 Decorator 라는 단어를 사용해야 한다.
    - 팀마다 특정 프로젝트에 적용할 표준을 고안한다. 유비쿼터스 언어라 부른다.
    - 프로젝트에 유효한 의미가 담긴 이름을 많이 사용할수록 독자가 코드를 이해하기 쉬워진다.
- N4: 명확한 이름
    - 함수나 변수의 목적을 명확히 밝히는 이름을 선택한다.
    
    ```java
    // bad
    // 이름만 봐서는 함수가 하는 일이 분명하지 않다.
    // doRename, renamePage 함수의 차이점이 모호하다.
    // renamePage -> renamePageAndOptionallyAllReferences
    // 이름이 길지만 모듈에서 한 번만 호출되며, 길다는 단점은 서술성이 메꾼다.
    private String doRename() throws Exception {
    	if(refactorReferences) {
    		renameReferences();
    	}
    	renamePage();
    
    	pathToRename.removeNameFromEnd();
    	ptahToRename.addNameToEnd(newName);
    	return PathParse.render(pathToRename);
    }
    ```
    
- N5: 긴 범위는 긴 이름을 사용하라
    - 이름 길이는 범위 길이에 비례해야 한다.
    - 범위가 5줄 안팎이라면 i나 j와 같은 변수 이름도 괜찮다.
    
    ```java
    // 볼링 게임
    // good
    // 오히려 i를 rollCount라고 쓰면 더 헷갈린다.
    private void rollMany(int n, int pins) {
    	for(int i=0; i<n; i++) {
    		g.roll(pins);
    	}
    }
    ```
    
- N6: 인코딩을 피하라
    - 이름에 유형 정보나 범위 정보를 넣어서는 안 된다.
    - 이름 앞에 m_이나 f와 같은 접두어가 불필요하다.
    - 헝가리안 표기법의 오염에서 이름을 보호하라.
- N7: 이름으로 부수 효과를 설명하라
    - 함수, 변수, 클래스가 하는 일을 모두 기술하는 이름을 사용한다.
    - 이름에 부수 효과를 숨기지 않는다.
    - 여러 작업을 수행하는 함수에다 동사 하나만 사용하면 곤란하다.
    
    ```java
    // bad
    // 단순히 "oos"만 가져오지 않는다. 기존에 "oos"가 없으면 생성한다.
    // getOos -> createOrReturnOos
    public ObjectOutputStream getOos() throws IOException {
    	if(m_oos == null) {
    		m_oos = new ObjectOutputStream(m_socket.getOutputStream());
    	}
    	return m_oos;
    } 
    ```
    

## 테스트

- T1: 불충분한 테스트
    - 테스트 케이스는 잠재적으로 깨질 만한 부분을 모두 테스트해야 한다.
    - 테스트 케이스가 확인하지 않는 조건이나 검증하지 않는 계산이 있다면 그 테스트는 불완전하다.
- T2: 커버리지 도구를 사용하라!
    - 커버리지 도구는 테스트가 빠뜨리는 공백을 알려준다.
- T3: 사소한 테스트를 건너뛰지 마라
    - 사소한 테스트 케이스는 짜기 쉬우며, 사소한 테스트가 제공하는 문서적 가치는 구현에 드는 비용을 넘어선다.
- T4: 무시한 테스트는 모호함을 뜻한다
    - 불분명한 요구사항은 테스트 케이스를 주석으로 처리하거나 @Ignore를 붙여 표현하게 만든다.
- T5: 경계 조건을 테스트하라
    - 경계 조건에서 실수하는 경우가 흔하므로, 각별히 신경 써서 테스트한다.
- T6: 버그 주변은 철저히 테스트하라
    - 버그는 서로 모이는 경향이 있다.
    - 한 함수에서 버그를 발견했다면 그 함수를 철저히 테스트하라. 다른 버그도 발견할 것이다.
- T7: 실패 패턴을 살펴라
    - 때로는 테스트 케이스가 실패하는 패턴으로 문제를 진단할 수 있다.
    - ex. 입력이 5자를 넘기는 테스트 케이스 모두 실패
    - ex. 함수 둘째 인수로 음수를 넘기는 테스트케이스 실패
    - 테스트 보고서에서 빨간색/녹색 패턴만 보고도 깨달을 수 있음
    - p. 343 SerialDate 클래스 예제 참고
- T8: 테스트 커버리지 패턴을 살펴라
    - 통과하는 테스트가 실행하거나 실행하지 않는 코드를 살펴보면 실패하는 테스트 케이스의 실패 원인이 드러난다.
- T9: 테스트는 빨라야 한다
    - 느린 테스트 케이스는 실행하지 않게 되므로 빨리 돌아가게 최대한 노력한다.
