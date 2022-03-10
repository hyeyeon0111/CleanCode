# Chapter 15. JUnit 들여다보기

## JUnit 프레임워크

### ComparisonCompactor 모듈

- 문자열 비교 오류를 파악할 때 유용한 코드
- 두 문자열을 받아 차이를 반환
- ABCDE, ABXDE → <...B[X]D...> 반환

### ComparisonCompactor 테스트 코드

```java
// 15-1. ComparisonCompactorTest.java
public class ComparisonCompactorTest extends TestCase {
    public void testMessage() {
        String failure = new ComparisonCompactor(0, "b", "c").compact("a");
        assertTrue("a expected:<[b]> but was:<[c]>".equals(failure));
    }

    public void testStartSame() {
        String failure = new ComparisonCompactor(1, "ba", "bc").compact(null);
        assertEquals("expected:<b[a]> but was:<b[c]>", failure);
    }

    public void testEndSame() {
        String failure = new ComparisonCompactor(1, "ab", "cb").compact(null);
        assertEquals("expected:<[a]b> but was:<[c]b>", failure);
    }

    public void testSame() {
        String failure = new ComparisonCompactor(1, "ab", "ab").compact(null);
        assertEquals("expected:<ab> but was:<ab>", failure);
    }

    public void testNoContextStartAndEndSame() {
        String failure = new ComparisonCompactor(0, "abc", "adc").compact(null);
        assertEquals("expected:<...[b]...> but was:<...[d]...>", failure);
    }

    public void testStartAndEndContext() {
        String failure = new ComparisonCompactor(1, "abc", "adc").compact(null);
        assertEquals("expected:<a[b]c> but was:<a[d]c>", failure);
    }

    public void testStartAndEndContextWithEllipses() {
        String failure = new ComparisonCompactor(1, "abcde", "abfde").compact(null);
        assertEquals("expected:<...b[c]d...> but was:<...b[f]d...>", failure);
    }

    public void testComparisonErrorStartSameComplete() {
        String failure = new ComparisonCompactor(2, "ab", "abc").compact(null);
        assertEquals("expected:<ab[]> but was:<ab[c]>", failure);
    }

    public void testComparisonErrorEndSameComplete() {
        String failure = new ComparisonCompactor(0, "bc", "abc").compact(null);
        assertEquals("expected:<[]...> but was:<[a]...>", failure);
    }

    public void testComparisonErrorEndSameCompleteContext() {
        String failure = new ComparisonCompactor(2, "bc", "abc").compact(null);
        assertEquals("expected:<[]bc> but was:<[a]bc>", failure);
    }

    // ...
}
```

- 코드 커버리지 분석 수행 결과 100% 나옴
- 테스트 케이스가 모든 행, if 문, for 문을 실행한다는 의미

### ComparisonComparactor 원본

```java
// 15-2. ComparisonComparactor.java (원본)
package junit.framework;

public class ComparisonCompactor {

    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";

    private int fContextLength;
    private String fExpected;
    private String fActual;
    private int fPrefix;
    private int fSuffix;

    public ComparisonCompactor(int contextLength, String expected, String actual) {
        fContextLength = contextLength;
        fExpected = expected;
        fActual = actual;
    }

    public String compact(String message) {
        if(fExpected == null || fActual == null || areStringEqual()) {
            return Assert.format(message, fExpected, fActual);
        }

        findCommonPrefix();
        findCommonSuffix();
        String expected = compactString(fExpected);
        String actual = compactString(fActual);
        return Assert.format(message, expected, actual);
    }

    private String compactString(String source) {
        String result = DELTA_START + source.substring(fPrefix, source.length() - fSuffix + 1) + DELTA_END;

        if(fPrefix > 0) {
            result = computeCommonPrefix() + result;
        }
        if(fSuffix > 0) {
            result = result + computeCommonSuffix();
        }
        return result;
    }

    private void findCommonPrefix() {
        fPrefix = 0;
        int end = Math.min(fExpected.length(), fActual.length());

        for(; fPrefix<end; fPrefix++) {
            if(fExpected.charAt(fPrefix) != fActual.charAt(fPrefix)) {
                break;
            }
        }
    }

    private void findCommonSuffix() {
        int expectedSuffix = fExpected.length() - 1;
        int actualSuffix = fActual.length() - 1;

        for(;
            actualSuffix >= fPrefix && expectedSuffix >= fPrefix;
            actualSuffix--, expectedSuffix--) {
            if(fExpected.charAt(expectedSuffix) != fActual.charAt(actualSuffix)) {
                break;
            }
        }
        fSuffix = fExpected.length() - expectedSuffix;
    }

    private String computeCommonPrefix() {
        return (fPrefix > fContextLength ? ELLIPSIS : "")
            + fExpected.substring(Math.max(0, fPrefix - fContextLength), fPrefix);
    }

    private String computeCommonSuffix() {
        int end = Math.min(fExpected.length() - fSuffix + 1 + fContextLength, fExpected.length());
        return fExpected.substring(fExpected.length() - fSuffix + 1, end)
            + (fExpected.length() - fSuffix + 1 < fExpected.length() - fContextLength ? ELLIPSIS : ""); 
    }

    private boolean areStringEqual() {
        return fExpected.equals(fActual);
    }
}
```

- 긴 표현식 몇 개와 이상한 +1 등이 눈에 띈다
- 하지만 전반적으로 상당히 훌륭한 모듈

### 보이스카우트 규칙에 따라 처음보다 깨끗하게 코드를 개선해보자

- 멤버 변수 앞에 붙인 접두어 f 제거
- compact 함수 시작부에 있는 조건문 캡슐화
    - shouldNotCompact 메서드로 만들기
- compact 함수에서 사용하는 expected 지역변수와 this.expected 이름 명확하게 바꾸기
    - expected → compactExpected
    - actual → compactActual
- 2번 조건문 반전시키기 (부정문은 긍정문보다 이해하기 약간 어려움)
    - shouldNotCompact → canBeCompacted
- 함수 이름 바꾸기
    - compact → formatCompactedComparison
    - 문자열 형식을 맞추는 작업만 수행
- compactExpectedAndActual 메서드 만들기
    - 예상 문자열과 실제 문자열을 진짜로 압축하는 부분을 빼냄
- 함수 사용방식 일관적으로 맞추기
    - findCommonPrefix, findCommonSuffix를 변경해 접두어 값과 접미어 값을 반환
    - 멤버 변수 이름도 prefix → prefixIndex로 정확하게 바꾸기
- 숨겨진 시간적인 결합 (hidden temporal coupling)
    - findCommonSuffix는 findCommonPrefix가 계산하는 prefixIndex에 의존
    - 두 함수를 잘못된 순서로 호출하면 디버깅해야함!
    1. 시간 결합을 외부에 노출하고자 prefixIndex를 findCommonSuffix의 인수로 넘김 → 별로,,
    2. findCommonSuffix를 findCommonPrefixAndSuffix로 바꾸고 findCommonPrefixAndSuffix에서 가장 먼저 findCommonPrefix 호출
        
        → 두 함수를 호출하는 순서가 분명해짐
        
- findCommonPrefixAndSuffix 함수 정리
- 변수 이름 수정
    - suffixIndex (접미어 길이) → suffixLength
    - prefixIndex (접두어 길이) → prefixLength
- 불필요한 if문 제거
    - compactString 함수 내 if 문
- 이것저것 손본 결과.. 최종 코드

### ComparisonComparactor 최종 코드

```java
// 15-5. ComparisonCompactor.java (최종 버전)
package junit.framework;

public class ComparisonCompactor {

    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";

    private int contextLength;
    private String expected;
    private String actual;
    private int prefixLength;
    private int suffixLength;

    public ComparisonCompactor(int contextLength, String expected, String actual) {
        this.contextLength = contextLength;
        this.expected = expected;
        this.actual = actual;
    }

    public String compact(String message) {
        String compactExpected = expected;
        String compactActual = actual;

        if(shouldBeCompacted()) {
            findCommonPrefixAndSuffix();
            compactExpected = compact(expected);
            compactActual = compact(actual);
        }
        return Assert.format(message, compactExpected, compactActual);
    }

    private boolean shouldBeCompacted() {
        return !shouldNotBeCompacted();
    }

    private boolean shouldNotBeCompacted() {
        return expected == null || actual == null || expected.equals(actual);
    }

    private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        suffixLength = 0;

        for(; !suffixOverlapsPrefix(); suffixLength++) {
            if(charFromEnd(expected, suffixLength) != charFromEnd(actual, suffixLength)) {
                break;
            }
        }
    }

    private char charFromEnd(String s, int i) {
        return s.charAt(s.length() - i - 1);
    }

    private boolean suffixOverlapsPrefix() {
        return actual.length() - suffixLength <= prefixLength 
            || expected.length() - suffixLength <= prefixLength;
    }

    private void findCommonPrefix() {
        prefixLength = 0;
        int end = Math.min(expected.length(), actual.length());

        for(; prefixLength < end; prefixLength++) {
            if(expected.charAt(prefixLength) != actual.charAt(prefixLength)) {
                break;
            }
        }
    }

    private String compact(String s) {
        return new StringBuilder()
            .append(startingEllipsis())
            .append(startingContext())
            .append(DELTA_START)
            .append(delta(s))
            .append(DELTA_END)
            .append(endingContext())
            .append(endingEllipsis())
            .toString();
    }

    private String startingEllipsis() {
        return prefixLength > contextLength ? ELLIPSIS : "";
    }

    private String startingContext() {
        int contextStart = Math.max(0, prefixLength - contextLength);
        int contextEnd = prefixLength;
        return expected.substring(contextStart, contextEnd);
    }

    private String delta(String s) {
        int deltaStart = prefixLength;
        int deltaEnd = s.length() - suffixLength;
        return s.substring(deltaStart, deltaEnd);
    }

    private String endingContext() {
        int contextStart = expected.length() - suffixLength;
        int contextEnd = Math.min(contextStart + contextLength, expected.length());
        return expected.substring(contextStart, contextEnd);
    }

    private String endingEllipsis() {
        return (suffixLength > contextLength ? ELLIPSIS : "");
    }
}
```

- 분석 함수와 조합 함수로 나뉜다.
    - 분석 함수가 먼저 나오고 조합 함수가 이어서 나옴
- 초반에 변경했던 코드를 되돌리기도 했다.
    - 리팩터링 하다 보면 흔히 생기는 일이다.

## 결론

- 리팩터링을 통해 보이스카우트 규칙을 지켰고 모듈은 처음보다 조금 더 깨끗해졌다.
