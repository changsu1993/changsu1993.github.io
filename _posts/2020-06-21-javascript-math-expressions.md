---
title: JavaScript 수학 표현식과 연산자 완벽 가이드
description: JavaScript의 산술 연산자, 증감 연산자, 할당 연산자와 연산자 우선순위를 상세히 알아봅니다. 전위/후위 연산자의 차이와 Math 객체 활용법을 마스터합니다.
author: changsu
date: 2020-06-21 02:01:22 +0900
categories: [Programming, JavaScript]
tags: [javascript, operator, math]
---

## JavaScript 수학 계산

JavaScript에서는 숫자를 사용하여 다양한 수학 계산을 수행할 수 있습니다.

### 기본 산술 연산

```javascript
let a = 1.3;
let b = 2;
let c = -10;

console.log(a + b);              // 3.3 (덧셈)
console.log(b * c / 10);         // -2 (곱셈과 나눗셈)
console.log(a + 10);             // 11.3
console.log(450 - 30);           // 420 (뺄셈)
console.log(a + 10 * b * b / 2 + 3);  // 24.3 (복합 연산)
```

## 산술 연산자

JavaScript는 다양한 산술 연산자를 제공합니다.

### 기본 산술 연산자

| 연산자 | 이름 | 설명 | 예시 | 결과 |
|--------|------|------|------|------|
| `+` | 덧셈 | 두 값을 더함 | `5 + 3` | `8` |
| `-` | 뺄셈 | 첫 번째 값에서 두 번째 값을 뺌 | `5 - 3` | `2` |
| `*` | 곱셈 | 두 값을 곱함 | `5 * 3` | `15` |
| `/` | 나눗셈 | 첫 번째 값을 두 번째 값으로 나눔 | `6 / 3` | `2` |
| `%` | 나머지 | 나눗셈의 나머지 | `7 % 3` | `1` |
| `**` | 거듭제곱 | 첫 번째 값을 두 번째 값만큼 제곱 | `2 ** 3` | `8` |

### 나머지 연산자 활용

```javascript
// 짝수/홀수 판별
console.log(10 % 2);  // 0 (짝수)
console.log(11 % 2);  // 1 (홀수)

// 배수 판별
console.log(15 % 5);  // 0 (5의 배수)
console.log(16 % 5);  // 1 (5의 배수 아님)

// 순환 인덱스
const colors = ['red', 'green', 'blue'];
for (let i = 0; i < 10; i++) {
  console.log(colors[i % 3]);  // 배열을 순환하며 접근
}
```

### 거듭제곱 연산자 (ES2016)

```javascript
console.log(2 ** 3);   // 8 (2³)
console.log(5 ** 2);   // 25 (5²)
console.log(10 ** -1); // 0.1 (10⁻¹)

// Math.pow()와 동일
console.log(Math.pow(2, 3));  // 8
```

## 증감 연산자

증감 연산자는 변수의 값을 1씩 증가 또는 감소시킵니다.

### 기본 사용법

```javascript
let num = 1;
num++;  // num = num + 1; 과 동일

console.log(num);  // 2

num--;  // num = num - 1; 과 동일
console.log(num);  // 1
```

### 후위 증감 연산자 (Postfix)

변수를 **먼저 사용한 후** 증가/감소합니다.

```javascript
let num = 1;
let newNum = num++;  // 1. newNum에 num(1) 할당 → 2. num 증가

console.log(num);     // 2 (증가됨)
console.log(newNum);  // 1 (할당 시점의 값)
```

**단계별 실행 과정**:

```javascript
// let newNum = num++; 를 풀어쓰면:
let num = 1;
let newNum = num;  // 1단계: 현재 값 할당
num++;             // 2단계: 증가
```

### 전위 증감 연산자 (Prefix)

변수를 **먼저 증가/감소한 후** 사용합니다.

```javascript
let num = 1;
let newNum = ++num;  // 1. num 증가 → 2. newNum에 num(2) 할당

console.log(num);     // 2 (증가됨)
console.log(newNum);  // 2 (증가된 값)
```

**단계별 실행 과정**:

```javascript
// let newNum = ++num; 를 풀어쓰면:
let num = 1;
num++;             // 1단계: 먼저 증가
let newNum = num;  // 2단계: 증가된 값 할당
```

### 전위 vs 후위 비교

```javascript
// 후위 증감 (Postfix)
let a = 5;
let b = a++;
console.log(a);  // 6
console.log(b);  // 5

// 전위 증감 (Prefix)
let x = 5;
let y = ++x;
console.log(x);  // 6
console.log(y);  // 6
```

### 실전 예제

```javascript
// 카운터 구현
let count = 0;
console.log(count++);  // 0 (출력 후 증가)
console.log(count);    // 1

// 배열 순회
let arr = [10, 20, 30];
let i = 0;
console.log(arr[i++]);  // 10 (인덱스 0 출력 후 i는 1)
console.log(arr[i++]);  // 20 (인덱스 1 출력 후 i는 2)
console.log(arr[i++]);  // 30 (인덱스 2 출력 후 i는 3)

// 반복문에서 활용
for (let j = 0; j < 5; j++) {
  console.log(j);  // 0, 1, 2, 3, 4
}
```

## 할당 연산자

변수에 값을 할당하면서 연산을 수행하는 복합 할당 연산자입니다.

### 복합 할당 연산자

| 연산자 | 예시 | 동일 표현 |
|--------|------|-----------|
| `+=` | `x += 5` | `x = x + 5` |
| `-=` | `x -= 3` | `x = x - 3` |
| `*=` | `x *= 2` | `x = x * 2` |
| `/=` | `x /= 4` | `x = x / 4` |
| `%=` | `x %= 3` | `x = x % 3` |
| `**=` | `x **= 2` | `x = x ** 2` |

### 사용 예제

```javascript
let score = 100;

score += 50;   // score = score + 50
console.log(score);  // 150

score -= 30;   // score = score - 30
console.log(score);  // 120

score *= 2;    // score = score * 2
console.log(score);  // 240

score /= 8;    // score = score / 8
console.log(score);  // 30

score %= 7;    // score = score % 7
console.log(score);  // 2
```

### 문자열 연결

```javascript
let message = "Hello";
message += " ";       // "Hello "
message += "World";   // "Hello World"
console.log(message); // "Hello World"

let name = "Alice";
name += "!";
console.log(name);    // "Alice!"
```

## 연산자 우선순위

JavaScript는 수학과 동일하게 연산자 우선순위를 따릅니다.

### 우선순위 표

| 우선순위 | 연산자 | 설명 |
|----------|--------|------|
| 1 | `()` | 괄호 |
| 2 | `**` | 거듭제곱 |
| 3 | `++`, `--` | 증감 |
| 4 | `*`, `/`, `%` | 곱셈, 나눗셈, 나머지 |
| 5 | `+`, `-` | 덧셈, 뺄셈 |
| 6 | `=`, `+=`, `-=`, ... | 할당 |

### 계산 순서 예제

```javascript
// 수학 계산: 1 + 2 * 2 = ?
console.log(1 + 2 * 2);  // 5 (곱셈이 먼저)

// 괄호 사용
console.log((1 + 2) * 2);  // 6 (괄호 안이 먼저)

// 복잡한 계산
let result = 10 + 5 * 2 - 3;
// 1. 5 * 2 = 10
// 2. 10 + 10 = 20
// 3. 20 - 3 = 17
console.log(result);  // 17

// 거듭제곱 포함
console.log(2 + 3 ** 2);    // 11 (3² = 9 먼저)
console.log((2 + 3) ** 2);  // 25 (5² = 25)
```

### 괄호로 명확하게

```javascript
// ❌ 헷갈리는 코드
let x = 5 + 3 * 2 - 1;

// ✅ 명확한 코드
let y = 5 + (3 * 2) - 1;  // 의도를 명확히 표현
```

## Math 객체

JavaScript는 복잡한 수학 계산을 위한 `Math` 객체를 제공합니다.

### 상수

```javascript
console.log(Math.PI);     // 3.141592653589793 (π)
console.log(Math.E);      // 2.718281828459045 (자연로그 밑)
console.log(Math.SQRT2);  // 1.4142135623730951 (√2)
```

### 반올림 메서드

```javascript
let num = 4.7;

console.log(Math.round(num));  // 5 (반올림)
console.log(Math.floor(num));  // 4 (내림)
console.log(Math.ceil(num));   // 5 (올림)
console.log(Math.trunc(num));  // 4 (정수 부분만)

// 음수에서의 차이
let negative = -4.7;
console.log(Math.floor(negative));  // -5
console.log(Math.trunc(negative));  // -4
```

### 최대/최소값

```javascript
console.log(Math.max(5, 10, 3, 8));   // 10
console.log(Math.min(5, 10, 3, 8));   // 3

// 배열에서 최대/최소
let numbers = [5, 10, 3, 8];
console.log(Math.max(...numbers));    // 10
console.log(Math.min(...numbers));    // 3
```

### 절대값과 부호

```javascript
console.log(Math.abs(-5));    // 5 (절대값)
console.log(Math.abs(5));     // 5

console.log(Math.sign(-10));  // -1 (음수)
console.log(Math.sign(0));    // 0
console.log(Math.sign(10));   // 1 (양수)
```

### 거듭제곱과 제곱근

```javascript
console.log(Math.pow(2, 3));   // 8 (2³)
console.log(Math.sqrt(16));    // 4 (√16)
console.log(Math.cbrt(27));    // 3 (∛27, 세제곱근)
```

### 난수 생성

```javascript
// 0 이상 1 미만의 난수
console.log(Math.random());  // 예: 0.4563421

// 1부터 10 사이의 정수 난수
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}
console.log(getRandomInt(1, 10));

// 배열에서 무작위 요소 선택
let colors = ['red', 'green', 'blue'];
let randomColor = colors[Math.floor(Math.random() * colors.length)];
console.log(randomColor);
```

### 삼각함수

```javascript
// 라디안 단위
console.log(Math.sin(Math.PI / 2));  // 1 (sin 90°)
console.log(Math.cos(0));            // 1 (cos 0°)
console.log(Math.tan(Math.PI / 4));  // 1 (tan 45°)

// 도(degree)를 라디안으로 변환
function toRadians(degrees) {
  return degrees * (Math.PI / 180);
}

console.log(Math.sin(toRadians(90)));  // 1
```

## 실전 예제

### 1. 총점과 평균 계산

```javascript
function calculateGrade(scores) {
  const total = scores.reduce((sum, score) => sum + score, 0);
  const average = total / scores.length;

  return {
    total: total,
    average: Math.round(average * 10) / 10,  // 소수점 첫째 자리
    grade: average >= 90 ? 'A' :
           average >= 80 ? 'B' :
           average >= 70 ? 'C' :
           average >= 60 ? 'D' : 'F'
  };
}

const scores = [85, 92, 78, 95, 88];
console.log(calculateGrade(scores));
// { total: 438, average: 87.6, grade: 'B' }
```

### 2. 할인가 계산

```javascript
function calculateDiscount(price, discountRate) {
  const discountAmount = price * (discountRate / 100);
  const finalPrice = price - discountAmount;

  return {
    originalPrice: price,
    discount: discountRate + '%',
    discountAmount: Math.round(discountAmount),
    finalPrice: Math.round(finalPrice)
  };
}

console.log(calculateDiscount(50000, 20));
// { originalPrice: 50000, discount: '20%',
//   discountAmount: 10000, finalPrice: 40000 }
```

### 3. 거리 계산

```javascript
function calculateDistance(x1, y1, x2, y2) {
  const dx = x2 - x1;
  const dy = y2 - y1;
  return Math.sqrt(dx ** 2 + dy ** 2);
}

console.log(calculateDistance(0, 0, 3, 4));  // 5
console.log(calculateDistance(1, 1, 4, 5));  // 5
```

### 4. 원의 넓이와 둘레

```javascript
function calculateCircle(radius) {
  return {
    area: Math.round(Math.PI * radius ** 2 * 100) / 100,
    circumference: Math.round(2 * Math.PI * radius * 100) / 100
  };
}

console.log(calculateCircle(5));
// { area: 78.54, circumference: 31.42 }
```

### 5. 카운트다운 타이머

```javascript
let countdown = 10;

const timer = setInterval(() => {
  console.log(countdown--);

  if (countdown < 0) {
    clearInterval(timer);
    console.log("발사!");
  }
}, 1000);
```

## 부동 소수점 주의사항

JavaScript는 부동 소수점 연산에서 정밀도 문제가 발생할 수 있습니다.

```javascript
// ❌ 예상치 못한 결과
console.log(0.1 + 0.2);  // 0.30000000000000004

// ✅ 해결 방법 1: 정수로 변환 후 계산
console.log((0.1 * 10 + 0.2 * 10) / 10);  // 0.3

// ✅ 해결 방법 2: toFixed() 사용
console.log((0.1 + 0.2).toFixed(2));  // "0.30" (문자열)
console.log(parseFloat((0.1 + 0.2).toFixed(2)));  // 0.3 (숫자)

// ✅ 해결 방법 3: Number.EPSILON 사용
function areEqual(a, b) {
  return Math.abs(a - b) < Number.EPSILON;
}

console.log(areEqual(0.1 + 0.2, 0.3));  // true
```

## 모범 사례

### 1. 명확한 변수명 사용

```javascript
// ❌ 나쁜 예
let x = 10000;
let y = x * 0.2;

// ✅ 좋은 예
let price = 10000;
let tax = price * 0.2;
```

### 2. 매직 넘버 피하기

```javascript
// ❌ 나쁜 예
let total = price * 0.9;

// ✅ 좋은 예
const DISCOUNT_RATE = 0.9;
let total = price * DISCOUNT_RATE;
```

### 3. 안전한 나눗셈

```javascript
// ❌ 나쁜 예
let result = a / b;  // b가 0이면?

// ✅ 좋은 예
let result = b !== 0 ? a / b : 0;

// 또는
function safeDivide(a, b) {
  if (b === 0) {
    console.error('0으로 나눌 수 없습니다.');
    return null;
  }
  return a / b;
}
```

### 4. 적절한 반올림

```javascript
// 금액 계산 시 소수점 둘째 자리까지
function roundMoney(amount) {
  return Math.round(amount * 100) / 100;
}

console.log(roundMoney(12.345));  // 12.35
console.log(roundMoney(12.344));  // 12.34
```

## 마치며

JavaScript의 수학 표현식과 연산자를 이해하면 다양한 계산 로직을 구현할 수 있습니다:

1. **산술 연산자**: `+`, `-`, `*`, `/`, `%`, `**`
2. **증감 연산자**: 전위(`++x`)와 후위(`x++`)의 차이 이해
3. **할당 연산자**: `+=`, `-=`, `*=`, `/=` 등으로 코드 간결화
4. **연산자 우선순위**: 수학 규칙과 동일, 괄호로 명확하게
5. **Math 객체**: 복잡한 계산을 위한 다양한 메서드 활용

부동 소수점 정밀도 문제를 인식하고, 적절한 반올림과 안전한 연산을 통해 정확한 계산 로직을 구현하세요.

## 참고 자료

- [MDN - Arithmetic operators](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators#%EC%82%B0%EC%88%A0_%EC%97%B0%EC%82%B0%EC%9E%90)
- [MDN - Increment (++)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Increment)
- [MDN - Assignment operators](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators#%ED%95%A0%EB%8B%B9_%EC%97%B0%EC%82%B0%EC%9E%90)
- [MDN - Math](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Math)
- [MDN - Operator precedence](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Operator_precedence)

