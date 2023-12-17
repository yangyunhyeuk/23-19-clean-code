## 15장 JUnit 들여다보기

---

### 1) 인코딩을 피하라

- 멤버 변수 앞에 붙은 접두어 f의 경우처럼 변수 이름에 범위를 명시할 필요하는 없다. 이 f는 중복되는 정보로 모두 제거하는 편이 좋다.

```java
// 변경 전
private int fContextLength;
private String fExpected;
private String fActual;
private int fPrefix;
private int fSuffix;
```

```java
// 변경 후
private int contextLength;
private String expected;
private String actual;
private int prefix;
private int suffix;
```

### 2) 조건을 캡슐화하라

- 의도를 명확히 하려면 조건문을 캡슐화해야 한다.
    - 조건문을 메서드로 뽑아내 적절한 이름을 붙인다.

```java
// 변경 전
...
if (expected == null || actual == null || areStringsEqual()) {
        return Assert.format(message, expected, actual);
    }
...
```

```java
// 변경 후
...
if (shouldNotCompact()) {
        return Assert.format(message, expected, actual);
    }
...

private boolean shouldNotCompact() {
    return expected == null || actual == null || areStringsEqual();
}
```

### 3) 가능하다면 표준 명명법을 사용하자

- compact 함수 내에서 사용되는 지역 변수의 경우 명확한 변수명을 지어주어 의미를 확실히 하자

```java
// 변경 전
String expected = compactString(this.expected);
String actual = compactString(this.actual);
```

```java
// 변경 후 
String compactExpected = compactString(expected);
String compactActual = compactString(actual);
```

### 4) 부정문은 긍정문보다 이해하기 약간 어렵다.

- 그러므로 첫 문장 조건문을 긍정으로 만들어주자

```java
// 변경 후 
public String compact(String message) {
    if (canBeCompacted()) {
        findCommonPrefix();
        findCommonSuffix();
        String compactExpected = compactString(expected);
        String compactActual = compactString(actual);
        return Assert.format(message, compactExpected, compactActual);
    } else {
        return Assert.format(message, expected, actual);
    }       
}

private boolean canBeCompacted() {
    return expected != null && actual != null && !areStringsEqual();
}
```

### 5) 이름으로 부수 효과를 설명하라

- 문자열을 압축하는 함수명이 뭐냐에 따라서 보는 이의 이해 과정이 수월할 수도, 어려울 수도 있다.
    - 만약 false라는 조건에 해당되면 압축이 안될 상황에 대해 설명할 수 없기 때문이다.

```java
// 변경 전
public String compact(String message) {...}

// 변경 후
public String formatCompactedComparison(String message) {...}
```

### 6) 함수는 단일 책임을 가져야 한다.

- 말 그대로이다

```java
// 변경 전
...

private String compactExpected;
private String compactActual;

...

public String formatCompactedComparison(String message) {
    if (canBeCompacted()) {
		    findCommonPrefix();
		    findCommonSuffix();
		    compactExpected = compactString(expected);
		    compactActual = compactString(actual);
        return Assert.format(message, compactExpected, compactActual);
    } else {
        return Assert.format(message, expected, actual);
    }       
}
```

```java
// 변경 후 
...

private String compactExpected;
private String compactActual;

...

public String formatCompactedComparison(String message) {
    if (canBeCompacted()) {
        compactExpectedAndActual();
        return Assert.format(message, compactExpected, compactActual);
    } else {
        return Assert.format(message, expected, actual);
    }       
}

private compactExpectedAndActual() {
    findCommonPrefix();
    findCommonSuffix();
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}
```

### 7) 일관성 부족

- 새 함수에서 마지막 두줄은 변수를 반환했지만, 첫째 줄과 둘째 줄은 반환값이 없다. 함수 사용 방식이 일관적이지 못한 부분임을 확인할 수 있다. 그래서 접두어 값과 접미어 값을 반환할 수 있게 수정해준다.

```java
// 변경 전 
private compactExpectedAndActual() {
    findCommonPrefix();
    findCommonSuffix();
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}
```

```java
// 변경 후 
private compactExpectedAndActual() {
    prefixIndex = findCommonPrefix();
    sufixIndex = findCommonSuffix();
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}
```

### 8) 서술적인 이름을 사용하라

- prefix에 좀 더 명확한 의미를 추가한다.
    - prefixIndex

### 9) 숨겨진 시각적인 결합이 존재한다.

- findCommandSuffix는 findCommonPrefix가 prefixIndex를 계산한다는 사실에 의존한다.
- findCommandSuffix는 findCommonPrefix가 prefixIndex를 잘못된 순서로 호출하면 밤샘 디버깅이라는 고생길이 열린다. 그래서 시간 결합을 외부에 노출하고자 findCommonSuffix를 고쳐 prefixIndex를 인수로 넘겼다.

```java
// 변경 전
private compactExpectedAndActual() {
    prefixIndex = findCommonPrefix();
    sufixIndex = findCommonSuffix();
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}
```

```java
// 변경 후 
private compactExpectedAndActual() {
    prefixIndex = findCommonPrefix();
    sufixIndex = findCommonSuffix(prefixIndex);
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}
```

### 10) 명확한 이름을 정의하자

- 코드를 수정하다보니, 변수의 역할에 기반하여 의미가 퇴색되는 경우가 존재한다.
    - 이 경우 적절한 이름으로 다시 정의해준다.

```java
// 변경 전
suffixIndex // 접미어의 길이

// 변경 후 
suffixLength // 접미어의 길이
```

### 11) 경계 조건을 캡슐화하자

- 배열의 인덱스 처리하는 로직을 캡슐화
    - 해당 클래스 내 private 메서드로 따로 빼서 처리

```java
// 예시
public class ComparisonCompactor {
...

private char charFromEnd(String s, int i){
	return s.charAt(s.length() -i -1);
}

...
}
```

### 12) 수정하며 쓸모가 없어진 죽은 코드는 제거하자

- 다음 예시는 로직을 수정하며 suffixLength의 길이가 1 이상일 때 처리되어, 0인 경우는 없는 경우에 해당

```java
if (suffixLength > 0)
```


--- 


- 모듈은 일련의 분석 함수와 일련의 조합 함수로 나뉜다.
    - 전체 함수는 위상적으로 정렬했으므로 각 함수가 사용한 직후에 정의된다. 분석 함수가 먼저 나오고 조합 함수가 뒤이어 나온다.
- 코드를 리팩토링하다보면 원래 했던 변경을 되돌리는 경우가 흔하다. 리팩토링은 코드가 어느 수준에 이를 때까지 수 많은 시행 착오를 반복하는 작업이기 때문이다.
