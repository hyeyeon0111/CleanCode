# Chapter 6. 객체와 자료 구조

변수를 private으로 정의하고 get/set 함수를 public으로 제공하여 많은 프로그래머가 private 변수를 외부에 노출하고 있다.

## 자료 추상화

- 구현을 감추기 위해서는 추상화가 필요하다.
- **추상 인터페이스**를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스
    - 인터페이스나 get/set 함수만으로는 추상화가 이뤄지지 않는다.
    - 아무 생각 없이 get/set 함수를 추가하지 말자..

```java
// 구현을 외부로 노출하는 클래스
// 직좌표계를 사용하며 개별적으로 좌표값을 읽고 설정하게 강제함
public class Point {
	public double x;
	public double y;
}

// 구현을 완전히 숨긴 클래스
// 직좌표계를 사용하는지 극좌표계를 사용하는지 알 수 없지만 자료구조를 명백하게 표현함
// 또한 메서드가 접근 정책을 강제함 (좌표 값은 개별적으로 읽고 설정 두 값을 한꺼번에 해야함)
public interface Point {
	double getX();
	double getY();
	void setCartesian(double x, double y);
	double getR();
	double getTheta();
	void setPolar(double r, double theta);
}
```

```java
// 구체적인 클래스
// 자동차 연료 상태를 구체적인 숫자 값으로 알려줌
public interface Vehicle {
	double getFuelTankCapacityInGallons();
	double getGallonsOfGasoline();
}

// 추상적인 클래스
// 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려줌
public interface Vehicle {
	double getPercentFuelRemaining();
}
```

## 자료/객체 비대칭

- **객체**는 추상화 뒤로 자료를 숨긴 채 **자료를 다루는 함수만 공개**한다.
- **자료구조**는 **자료를 그대로 공개**하며 별다른 함수는 제공하지 않는다.

**절차적인 도형 클래스 (p.120)**

- 각 도형 클래스(Square, Rectangle, Circle)는 자료구조이며 아무 메서드도 제공하지 않는다.
    - 새로운 함수를 추가할 때 도형 클래스는 아무 영향도 받지 않음
- 도형이 동작하는 방식은 Geometry 클래스에서 구현한다.
    - 새 도형을 추가하고 싶다면 클래스에 속한 함수를 모두 고쳐야 함

**객체 지향적인 도형 클래스 (p.121)**

- 각 도형 클래스(Square, Rectangle, Circle)는 area() 함수를 제공한다.
    - 새 도형을 추가해도 기존 함수에 아무런 영향을 미치지 않음
- Geometry 클래스는 필요 없다.
    - 새 함수를 추가하고 싶다면 도형 클래스를 전부 고쳐야 함

**정리**

- 객체와 자료 구조는 근본적으로 양분된다.
- 객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.

cf. 모든 것이 객체일 수 없고, 때로는 단순한 자료 구조와 절차적인 코드가 적합한 상황도 있다.

## 디미터 법칙

- 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙
- 객체는 조회 함수로 내부 구조를 공개하면 안 된다는 의미
- “클래스 C의 메서드 f는 아래 객체의 메서드만 호출해야 한다”
    - 클래스 C
    - f가 생성한 객체
    - f 인수로 넘어온 객체
    - C 인스턴스 변수에 저장된 객체
    - 하지만, f가 호출한 메서드가 반환하는 객체의 메서드는 호출하면 안됨

```java
// 디미터 법칙 위반
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

### 기차 충돌

- 여러 객차가 한 줄로 이어진 기차처럼 보이는 코드
- 일반적으로 조잡..하므로 피하는 편이 좋다.

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

⬇️

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

❓디미터 법칙을 위반하는가

- ctxt, Options, ScratchDir이 객체라면 내부 구조를 숨겨야 하므로 디미터 법칙 위반
    - ctxt 객체는 Options를 포함하고 Options가 ScratchDir를 포함... 하는 사실을 알기 때문
- ctxt, Options, ScratchDir이 자료 구조라면 디미터 법칙이 적용되지 않음
    - 자료 구조는 당연히 내부 구조를 노출하기 때문
    - ex. final String outputDir = ctxt.options.scratchDir.absolutePath;

### 잡종 구조

- 절반은 객체, 절반은 자료 구조인 구조
- 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다..
- 되도록 피하는 편이 좋다

### 구조체 감추기

- ctxt, Options, ScratchDir이 객체라면 내부 구조를 감춰야 한다.
- 임시 디렉터리의 절대 경로가 필요한 이유: 임시 파일을 생성하기 위해
    - ctxt 객체에 임시 파일을 생성하라고 시키자!!

```java
// ctxt는 내부 구조를 드러내지 않으며, 모듈에서 해당 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없음
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

→ 디미터 법칙 위반하지 않음!

## 자료 전달 객체

- DTO(Data Transfer Object): 공개 변수만 있고 함수가 없는 클래스, 자료 구조체의 전형적인 형태
    - 데이터베이스와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때 유용
    - 데이터베이스에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 단계에서 가장 처음으로 사용하는 구조체
- 빈(bean): 일반적인 자료 구조체의 형태
    - 비공개(private) 변수를 get/set 함수로 조작한다.
    - 별다른 이익은 없음..

### 활성 레코드

- DTO의 특수한 형태, 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과
- 공개 변수 또는 비공개 변수에 get/set 함수가 있는 자료 구조지만, save나 find와 같은 탐색 함수도 제공

❗활성 레코드에 비즈니스 규칙 메서드를 추가해 객체로 취급하지 말 것 → 잡종 구조가 나옴

❗활성 레코드는 자료 구조로 취급, 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성

## 결론

시스템 구현 시, 새로운 자료 타입을 추가하는 유연성이 필요하다면 객체가 더 적합

새로운 동작을 추가하는 유연성이 필요하다면 자료구조와 절차적인 코드가 더 적합

→ 자료구조와 객체를 잘 선택하여 개발하자!
