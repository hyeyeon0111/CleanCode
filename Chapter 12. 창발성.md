# Chapter 12. 창발성

## 창발적 설계로 깔끔한 코드를 구현하자

켄트 벡이 제시한 단순한 설계 규칙 네 가지 (중요도순)

1. 모든 테스트를 실행한다.
2. 중복을 없앤다.
3. 프로그래머 의도를 표현한다.
4. 클래스와 메서드 수를 최소로 줄인다.

## 단순한 설계 규칙 1: 모든 테스트를 실행하라

- 설계는 의도한 대로 돌아가는 시스템을 내놓아야 한다.
- 모든 테스트 케이스를 항상 통과하는 시스템은 ‘테스트가 가능한 시스템'이다.
- 테스트가 가능한 시스템을 만들려고 애쓰면 설계 품질이 더불어 높아진다.
    - 크기가 작고 목적 하나만 수행하는 클래스가 나온다.(SRP)
    - 테스트 케이스가 많을수록 개발자는 테스트가 쉽게 코드를 작성한다.
- 결합도가 높으면 테스트 케이스를작성하기 어렵다.
    - 테스트 케이스를 많이 작성하여 DIP, DI, 인터페이스, 추상화 등과 같은 도구를 사용해 결합도를 낮춘다.

→ “테스트 케이스를 만들고 계속 돌려라” 규칙을 따르면 시스템은 낮은 결합도와 높은 응집력이라는 객체 지향 방법론이 지향하는 목표를 저절로 달성한다.

## 단순한 설계 규칙 2~4: 리팩터링

- 코드와 클래스를 정리하자.
- 소프트웨어 설계 품질을 높이는 기법을 적용하자.
    - 응집도를 높이고, 결합도를 낮추고, 관심사를 분리하고, 시스템 관심사를 모듈로 나누고, 함수와 클래스 크기를 줄이고, 더 나은 이름을 선택하는 등 다양한 기법을 동원한다.
- 중복을 제거하고, 프로그래머 의도를 표현하고, 클래스와 메서드 수를 최소로 줄이는 단계이기도 하다.

## 중복을 없애라

- 중복은 커다란 적이다.
    - 추가 작업, 추가 위험, 불필요한 복잡도를 뜻하기 때문이다.
- 중복의 형태
    - 똑같은 코드
    - 구현 중복

```java
// 집합 클래스
int size() {}
boolean isEmpty() {}

// isEmpty 메서드에서 size 메서드를 이용 -> 코드를 중복해 구현할 필요가 없어진다!
boolean isEmpty() {
	return 0 == size();
}
```

- 단 몇 줄이라도 중복을 제거하겠다는 의지가 필요하다.

```java
// 중복 제거 전
public void scaleToOneDimension {
	// ...
	RenderedOp newImage = ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor);
	image.dispose();
	System.gc();
	image = newImage;
}

public synchronized void rotate(int degrees) {
	RenderedOp newImage = ImageUtilities.getRotatedImage(image, degrees);
	image.dispose();
	System.gc();
	image = newImage;
}
```

```java
// 중복 제거 후
public void scaleToOneDimension {
	// ...
	replaceImage(ImageUtilities.getScaledImage(image, scalingFactor, scalingFactor));
}

public synchronized void rotate(int degrees) {
	replaceImage(ImageUtilities.getRotatedImage(image, degrees));
}

private void replaceImage(RenderedOp newImage) {
	image.dispose();
	System.gc();
	image = newImage;
}
```

- 공통 코드를 새 메서드로 뽑고 나니 클래스가 SRP를 위반한다.
    - replaceImage 메서드를 다른 클래스로 옮기는게 좋음!
    - 다른 팀원이 메서드를 추상화해 다른 맥락에서 재사용할지도 모른다!
- ‘소규모 재사용’은 시스템 복잡도를 극적으로 줄여준다.
- TEMPLATE METHOD 패턴
    - 고차원 중복을 제거할 목적으로 자주 사용하는 기법

```java
// 템플릿 메소드 패턴 적용 전
public class VacationPolicy {
	public void accrueUSDivisionVacation() {
		// 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
		// 휴가 일수가 미국 최소 법정 일수를 만족하는지 확인하는 코드
		// 휴가 일수를 급혀 대장에 적용하는 코드
	}

	public void accrueEUDivisionVacation() {
		// 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
		// 휴가 일수가 유럽연합 최소 법정 일수를 만족하는지 확인하는 코드
		// 휴가 일수를 급혀 대장에 적용하는 코드
	}
}
```

```java
// 템플릿 메소드 패턴 적용 후
public class VacationPolicy {
	public void accrueVacation() {
		calculateBaseVacationHours();
		alterForLegalMinimums();
		applyToPayroll();
	}

	private void calculateBaseVacationHours() {...}
	abstract protected void alterForLegalMinimums();
	private void applyToPayroll() {...}
}

public class USVacationPolicy extends VacationPolicy {
	@Override protected void alterForLegalMinimums() {
		// 미국 최소 법정 일수를 사용한다.
	}
}

public class EUVacationPolicy extends VacationPolicy {
	@Override protected void alterForLegalMinimums() {
		// 유럽연합 최소 법정 일수를 사용한다.
	}
}
```

- 하위 클래스는 중복되지 않는 정보만 제공해 accrueVacation 알고리즘에서 빠진 ‘구멍'을 메운다.

## 표현하라

- 유지보수 개발자가 코드를 이해하기 쉽도록 코드는 개발자의 의도를 분명히 표현해야 한다.
1. 좋은 이름을 선택한다.
2. 함수와 클래스 크기를 가능한 줄인다.
3. 표준 명칭을 사용한다. ex. COMMAND, VISITOR 같은 표준 패턴은 클래스 이름에 패턴 이름을 넣어준다.
4. 단위 테스트 케이스를 꼼꼼히 작성한다. 테스트 케이스는 ‘예제로 보여주는 문서'다.
- 표현력을 높이는 가장 중요한 방법은 **노력!**

## 클래스와 메서드 수를 최소로 줄여라

- 중복 제거, 의도 표현, SRP 준수를 극단적으로 지키다 보면 작은 클래스와 메서드를 수없이 만들기도 한다.
- 그래서 함수와 클래스 수를 가능한 줄이라고 제안한다.

## 결론

- 위 설계 규칙을 따른다면 우수한 기법과 원칙을 단번에 활용할 수 있다.
