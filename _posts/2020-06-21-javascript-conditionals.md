---
title: JavaScript 조건문 완벽 가이드 - if, else, switch
description: JavaScript의 조건문(if, else if, else)과 비교 연산자, 논리 연산자를 상세히 알아봅니다. truthy/falsy 개념, 삼항 연산자, switch문까지 조건 제어의 모든 것을 마스터합니다.
author: changsu
date: 2020-06-21 14:19:36 +0900
categories: [Programming, JavaScript]
tags: [javascript, if, else, switch, conditional, comparison, logical-operator, 조건문]
---

## 조건문이란?

**조건문(Conditional Statement)**은 "만약 ~하면 ~한다"를 프로그래밍 언어로 표현한 것입니다. 특정 조건이 참(true)인지 거짓(false)인지에 따라 다른 코드를 실행합니다.

### 조건문의 필요성

```javascript
// 조건문 없이 (모든 경우를 처리할 수 없음)
let age = 20;
console.log("성인입니다");

// 조건문 사용 (상황에 따라 다른 동작)
let age = 20;
if (age >= 18) {
  console.log("성인입니다");
} else {
  console.log("미성년자입니다");
}
```

## if 문

**if 문**은 가장 기본적인 조건문으로, 조건이 참일 때만 코드를 실행합니다.

### 기본 구조

```javascript
if (조건) {
  // 조건이 true일 때 실행되는 코드
}
```

### 기본 예제

```javascript
let answer = 1 + 1;

if (answer > 1) {
  alert("1보다 큰 숫자!");
}
// alert가 실행됨 (2 > 1이므로 true)
```

**동작 과정**:
1. `answer` 변수에 `1 + 1`의 결과인 `2`가 저장됨
2. `answer > 1` 조건 확인 (`2 > 1`은 `true`)
3. 조건이 참이므로 중괄호 `{}` 안의 코드 실행

### 문자열 비교

```javascript
let answer = "비밀";

if (answer === "비밀") {
  alert("맞았습니다!");
  alert("축하해요!");
}
```

## 비교 연산자

조건문에서 사용하는 비교 연산자입니다.

### 비교 연산자 종류

| 연산자 | 의미 | 예시 | 결과 |
|--------|------|------|------|
| `>` | 크다 | `5 > 3` | `true` |
| `<` | 작다 | `5 < 3` | `false` |
| `>=` | 크거나 같다 | `5 >= 5` | `true` |
| `<=` | 작거나 같다 | `3 <= 5` | `true` |
| `===` | 같다 (엄격) | `5 === 5` | `true` |
| `!==` | 같지 않다 (엄격) | `5 !== 3` | `true` |
| `==` | 같다 (타입 변환) | `"5" == 5` | `true` |
| `!=` | 같지 않다 (타입 변환) | `"5" != 5` | `false` |

### === vs ==

```javascript
// === (엄격한 비교 - 타입까지 비교)
console.log(5 === 5);      // true
console.log(5 === "5");    // false (타입이 다름)
console.log(0 === false);  // false (타입이 다름)

// == (느슨한 비교 - 타입 변환 후 비교)
console.log(5 == "5");     // true (문자열을 숫자로 변환)
console.log(0 == false);   // true (불리언을 숫자로 변환)
console.log(null == undefined);  // true

// ✅ 권장: 항상 === 사용
// ❌ 비권장: == 사용 (예상치 못한 동작 가능)
```

### 비교 연산자 예제

```javascript
let score = 85;

if (score >= 90) {
  console.log("A 등급");
}

if (score > 100) {
  console.log("범위 초과");  // 실행되지 않음
}

let name = "홍길동";
if (name === "홍길동") {
  console.log("본인 확인");
}
```

## if-else 문

조건이 거짓일 때 실행할 코드를 **else** 블록에 작성합니다.

### 기본 구조

```javascript
if (조건) {
  // 조건이 true일 때
} else {
  // 조건이 false일 때
}
```

### 기본 예제

```javascript
let answer = 3 + 3;

if (answer > 5) {
  alert("5보다 큰 숫자!");
} else {
  alert("5보다 작거나 같은 숫자!");
}
// "5보다 큰 숫자!" 출력 (6 > 5)
```

### 실전 예제

```javascript
// 성인/미성년자 판별
let age = 17;

if (age >= 18) {
  console.log("성인입니다");
} else {
  console.log("미성년자입니다");
}

// 짝수/홀수 판별
let number = 7;

if (number % 2 === 0) {
  console.log("짝수입니다");
} else {
  console.log("홀수입니다");
}

// 로그인 상태 확인
let isLoggedIn = true;

if (isLoggedIn) {
  console.log("환영합니다!");
} else {
  console.log("로그인이 필요합니다");
}
```

## else if 문

여러 조건을 순차적으로 확인할 때 사용합니다.

### 기본 구조

```javascript
if (조건1) {
  // 조건1이 true일 때
} else if (조건2) {
  // 조건1이 false이고 조건2가 true일 때
} else if (조건3) {
  // 조건1, 2가 false이고 조건3이 true일 때
} else {
  // 모든 조건이 false일 때
}
```

### 기본 예제

```javascript
let answer = 3 + 3;

if (answer > 15) {
  alert("15보다 큰 숫자!");
} else if (answer > 10) {
  alert("10보다 큰 숫자!");
} else if (answer > 5) {
  alert("5보다 큰 숫자!");
} else {
  alert("5보다 작거나 같은 숫자!");
}
// "5보다 큰 숫자!" 출력 (6 > 5)
```

### 조건문 순서의 중요성

```javascript
let score = 85;

// ✅ 올바른 순서 (큰 값부터)
if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");  // 이것이 실행됨
} else if (score >= 70) {
  console.log("C");
} else {
  console.log("D");
}

// ❌ 잘못된 순서 (작은 값부터)
if (score >= 70) {
  console.log("C");  // 이것이 실행됨 (의도와 다름)
} else if (score >= 80) {
  console.log("B");  // 도달하지 못함
} else if (score >= 90) {
  console.log("A");  // 도달하지 못함
}
```

### 실전 예제

```javascript
// 성적 등급
function getGrade(score) {
  if (score >= 90) {
    return "A";
  } else if (score >= 80) {
    return "B";
  } else if (score >= 70) {
    return "C";
  } else if (score >= 60) {
    return "D";
  } else {
    return "F";
  }
}

console.log(getGrade(95));  // "A"
console.log(getGrade(75));  // "C"

// 시간대별 인사말
function getGreeting(hour) {
  if (hour < 12) {
    return "좋은 아침입니다";
  } else if (hour < 18) {
    return "좋은 오후입니다";
  } else {
    return "좋은 저녁입니다";
  }
}

console.log(getGreeting(9));   // "좋은 아침입니다"
console.log(getGreeting(14));  // "좋은 오후입니다"
```

## 논리 연산자

여러 조건을 결합할 때 사용합니다.

### 논리 연산자 종류

| 연산자 | 의미 | 설명 |
|--------|------|------|
| `&&` | AND (그리고) | 모든 조건이 true여야 true |
| `\|\|` | OR (또는) | 하나라도 true면 true |
| `!` | NOT (부정) | true를 false로, false를 true로 |

### AND 연산자 (&&)

```javascript
// 두 조건 모두 true여야 실행
let age = 25;
let hasLicense = true;

if (age >= 18 && hasLicense) {
  console.log("운전 가능");
}

// 범위 확인
let score = 75;
if (score >= 70 && score < 80) {
  console.log("C 등급");
}
```

### OR 연산자 (||)

```javascript
// 하나라도 true면 실행
let isWeekend = true;
let isHoliday = false;

if (isWeekend || isHoliday) {
  console.log("휴일입니다");
}

// 여러 값 중 하나 확인
let fruit = "사과";
if (fruit === "사과" || fruit === "바나나" || fruit === "오렌지") {
  console.log("과일입니다");
}
```

### NOT 연산자 (!)

```javascript
let isLoggedIn = false;

if (!isLoggedIn) {
  console.log("로그인이 필요합니다");
}

// 부정 확인
let isEmpty = "";
if (!isEmpty) {
  console.log("비어있습니다");
}
```

### 복합 논리 연산

```javascript
let age = 25;
let hasTicket = true;
let isVIP = false;

// 복잡한 조건
if ((age >= 18 && hasTicket) || isVIP) {
  console.log("입장 가능");
}

// 우선순위: ! > && > ||
let result = !false && true || false;
console.log(result);  // true
```

## truthy와 falsy

JavaScript는 불리언이 아닌 값도 조건문에서 true/false로 평가합니다.

### falsy 값 (false로 평가)

```javascript
// 6가지 falsy 값
if (false) { }       // 실행 안됨
if (0) { }           // 실행 안됨
if ("") { }          // 실행 안됨 (빈 문자열)
if (null) { }        // 실행 안됨
if (undefined) { }   // 실행 안됨
if (NaN) { }         // 실행 안됨
```

### truthy 값 (true로 평가)

```javascript
// falsy 외의 모든 값
if (true) { }        // 실행됨
if (1) { }           // 실행됨
if ("hello") { }     // 실행됨 (비어있지 않은 문자열)
if ([]) { }          // 실행됨 (빈 배열도 truthy)
if ({}) { }          // 실행됨 (빈 객체도 truthy)
if (42) { }          // 실행됨
```

### 실전 활용

```javascript
// 값 존재 확인
let userName = "";

if (userName) {
  console.log(`환영합니다, ${userName}님`);
} else {
  console.log("이름을 입력해주세요");
}

// 배열 길이 확인
let items = [];

if (items.length) {
  console.log("항목이 있습니다");
} else {
  console.log("항목이 없습니다");
}

// 기본값 설정
let name = userName || "손님";
console.log(name);  // "손님"
```

## 삼항 연산자

간단한 if-else를 한 줄로 작성할 수 있습니다.

### 기본 구조

```javascript
조건 ? true일_때_값 : false일_때_값
```

### 기본 예제

```javascript
let age = 20;

// if-else 사용
let message1;
if (age >= 18) {
  message1 = "성인";
} else {
  message1 = "미성년자";
}

// 삼항 연산자 사용 (권장)
let message2 = age >= 18 ? "성인" : "미성년자";

console.log(message2);  // "성인"
```

### 실전 예제

```javascript
// 짝수/홀수 판별
let num = 7;
let type = num % 2 === 0 ? "짝수" : "홀수";
console.log(type);  // "홀수"

// 가격 할인
let price = 50000;
let finalPrice = price >= 30000 ? price * 0.9 : price;
console.log(finalPrice);  // 45000

// 상태 표시
let isOnline = true;
let status = isOnline ? "온라인" : "오프라인";
console.log(status);  // "온라인"

// 함수 반환값
function getDiscount(isMember) {
  return isMember ? 0.1 : 0;
}

console.log(getDiscount(true));   // 0.1
console.log(getDiscount(false));  // 0
```

### 중첩 삼항 연산자

```javascript
// 성적 등급 (가독성 주의)
let score = 85;
let grade = score >= 90 ? "A" :
            score >= 80 ? "B" :
            score >= 70 ? "C" : "D";

console.log(grade);  // "B"

// ✅ 복잡한 경우 if-else 사용 권장
function getGrade(score) {
  if (score >= 90) return "A";
  if (score >= 80) return "B";
  if (score >= 70) return "C";
  return "D";
}
```

## switch 문

여러 값 중 하나와 일치하는지 확인할 때 사용합니다.

### 기본 구조

```javascript
switch (표현식) {
  case 값1:
    // 값1과 일치할 때
    break;
  case 값2:
    // 값2와 일치할 때
    break;
  default:
    // 일치하는 값이 없을 때
}
```

### 기본 예제

```javascript
let day = 3;

switch (day) {
  case 1:
    console.log("월요일");
    break;
  case 2:
    console.log("화요일");
    break;
  case 3:
    console.log("수요일");
    break;
  case 4:
    console.log("목요일");
    break;
  case 5:
    console.log("금요일");
    break;
  case 6:
  case 7:
    console.log("주말");
    break;
  default:
    console.log("잘못된 요일");
}
// "수요일" 출력
```

### break의 중요성

```javascript
let grade = "B";

// ❌ break 없으면 fall-through 발생
switch (grade) {
  case "A":
    console.log("우수");
    // break가 없어서 계속 실행됨
  case "B":
    console.log("보통");
  case "C":
    console.log("미흡");
  default:
    console.log("재수강");
}
// "보통", "미흡", "재수강" 모두 출력됨

// ✅ break로 중단
switch (grade) {
  case "A":
    console.log("우수");
    break;
  case "B":
    console.log("보통");
    break;
  case "C":
    console.log("미흡");
    break;
  default:
    console.log("재수강");
}
// "보통"만 출력됨
```

### 여러 case 묶기

```javascript
let fruit = "바나나";

switch (fruit) {
  case "사과":
  case "배":
  case "포도":
    console.log("국산 과일");
    break;
  case "바나나":
  case "파인애플":
  case "망고":
    console.log("수입 과일");
    break;
  default:
    console.log("알 수 없는 과일");
}
// "수입 과일" 출력
```

### switch vs if-else

```javascript
// switch 사용 (값 비교에 적합)
let color = "red";

switch (color) {
  case "red":
    console.log("빨강");
    break;
  case "blue":
    console.log("파랑");
    break;
  case "green":
    console.log("초록");
    break;
}

// if-else 사용 (범위 비교에 적합)
let score = 85;

if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else {
  console.log("C");
}
```

## 실전 예제

### 1. 로그인 검증

```javascript
function validateLogin(username, password) {
  if (!username) {
    return "아이디를 입력하세요";
  }

  if (!password) {
    return "비밀번호를 입력하세요";
  }

  if (username.length < 4) {
    return "아이디는 4자 이상이어야 합니다";
  }

  if (password.length < 8) {
    return "비밀번호는 8자 이상이어야 합니다";
  }

  return "로그인 성공";
}

console.log(validateLogin("user", "pass"));  // "아이디는 4자 이상이어야 합니다"
console.log(validateLogin("user123", "password123"));  // "로그인 성공"
```

### 2. 요금 계산

```javascript
function calculateFee(age, isStudent) {
  if (age < 7) {
    return 0;  // 무료
  } else if (age < 13) {
    return 5000;  // 어린이
  } else if (age < 19) {
    return isStudent ? 7000 : 10000;  // 청소년
  } else if (age < 65) {
    return 12000;  // 성인
  } else {
    return 6000;  // 경로우대
  }
}

console.log(calculateFee(5, false));    // 0
console.log(calculateFee(16, true));    // 7000
console.log(calculateFee(30, false));   // 12000
```

### 3. 계절 판별

```javascript
function getSeason(month) {
  switch (month) {
    case 12:
    case 1:
    case 2:
      return "겨울";
    case 3:
    case 4:
    case 5:
      return "봄";
    case 6:
    case 7:
    case 8:
      return "여름";
    case 9:
    case 10:
    case 11:
      return "가을";
    default:
      return "잘못된 월";
  }
}

console.log(getSeason(3));   // "봄"
console.log(getSeason(12));  // "겨울"
```

## 모범 사례

### 1. === 사용

```javascript
// ❌ 나쁜 예
if (value == 0) { }

// ✅ 좋은 예
if (value === 0) { }
```

### 2. 조기 반환 (Early Return)

```javascript
// ❌ 나쁜 예 (중첩 깊음)
function processUser(user) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        return "처리 완료";
      }
    }
  }
  return "처리 실패";
}

// ✅ 좋은 예 (조기 반환)
function processUser(user) {
  if (!user) return "처리 실패";
  if (!user.isActive) return "처리 실패";
  if (!user.hasPermission) return "처리 실패";

  return "처리 완료";
}
```

### 3. 명확한 조건

```javascript
// ❌ 나쁜 예
if (x) { }

// ✅ 좋은 예
if (x !== null && x !== undefined) { }
if (x > 0) { }
```

### 4. 복잡한 조건 분리

```javascript
// ❌ 나쁜 예
if (user.age >= 18 && user.hasLicense && !user.isSuspended && user.points >= 0) {
  // ...
}

// ✅ 좋은 예
const isAdult = user.age >= 18;
const canDrive = user.hasLicense && !user.isSuspended && user.points >= 0;

if (isAdult && canDrive) {
  // ...
}
```

## 참고 자료

- [MDN - if...else](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/if...else)
- [MDN - Conditional (ternary) operator](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Conditional_operator)
- [MDN - switch](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/switch)
- [MDN - Logical operators](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Logical_OR)
- [MDN - Comparison operators](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators)
- [JavaScript.info - Conditional branching](https://javascript.info/ifelse)

## 마치며

JavaScript 조건문은 프로그램의 흐름을 제어하는 핵심 기능입니다:

1. **if-else**: 조건에 따라 다른 코드 실행
2. **비교 연산자**: `===`, `!==`, `>`, `<`, `>=`, `<=`
3. **논리 연산자**: `&&`, `||`, `!`
4. **truthy/falsy**: 불리언이 아닌 값의 조건 평가
5. **삼항 연산자**: 간단한 조건을 한 줄로 표현
6. **switch**: 여러 값 중 하나와 일치 확인

조건문 순서, `===` 사용, 조기 반환 등의 모범 사례를 따라 깔끔하고 읽기 쉬운 코드를 작성하세요.
