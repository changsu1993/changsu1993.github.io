---
title: JavaScript Number와 Math 완벽 가이드 - 숫자 처리의 모든 것
description: JavaScript의 Number 타입과 Math 객체를 마스터하세요. 반올림, 난수 생성, 소수점 처리, 숫자 포맷팅 등 실무에서 바로 쓸 수 있는 모든 것을 다룹니다.
author: cotes
date: 2020-07-04 18:11:22 +0900
categories: [JavaScript, Basics]
tags: [javascript, number, math]
---

## Number 타입 기본

JavaScript에서 숫자는 **64비트 부동소수점** 형식으로 저장됩니다. 정수와 실수를 구분하지 않습니다.

```javascript
const integer = 42;        // 정수
const float = 3.14;        // 실수
const negative = -10;      // 음수
const exponential = 2e3;   // 지수 표기법 (2000)

console.log(typeof integer);  // "number"
console.log(typeof float);    // "number"
```

## Math 객체 완벽 가이드

`Math` 객체는 수학 계산을 위한 **내장 객체**입니다. 생성자가 없어 `new Math()`처럼 사용할 수 없습니다.

### 1. 반올림 메서드

#### Math.round() - 반올림

```javascript
console.log(Math.round(2.5));   // 3
console.log(Math.round(2.49));  // 2
console.log(Math.round(2));     // 2
console.log(Math.round(2.82));  // 3
console.log(Math.round(-2.5));  // -2 (음수는 0에 가깝게)
```

#### Math.ceil() - 올림

```javascript
console.log(Math.ceil(2.1));   // 3
console.log(Math.ceil(2.9));   // 3
console.log(Math.ceil(2));     // 2
console.log(Math.ceil(-2.1));  // -2 (음수는 0 방향으로)
```

#### Math.floor() - 내림

```javascript
console.log(Math.floor(2.1));   // 2
console.log(Math.floor(2.9));   // 2
console.log(Math.floor(2));     // 2
console.log(Math.floor(-2.1));  // -3 (음수는 더 작은 정수로)
```

#### Math.trunc() - 소수점 제거 (ES6)

```javascript
console.log(Math.trunc(2.9));   // 2
console.log(Math.trunc(-2.9));  // -2 (부호 유지, 소수만 제거)

// floor와의 차이
console.log(Math.floor(-2.9));  // -3
console.log(Math.trunc(-2.9));  // -2
```

### 2. 최대/최소값

#### Math.max() / Math.min()

```javascript
console.log(Math.max(1, 5, 3, 9, 2));     // 9
console.log(Math.min(1, 5, 3, 9, 2));     // 1

// 배열의 최대/최소값
const numbers = [1, 5, 3, 9, 2];
console.log(Math.max(...numbers));  // 9 (스프레드 연산자)
console.log(Math.min(...numbers));  // 1

// apply 사용 (ES5)
console.log(Math.max.apply(null, numbers));  // 9
```

### 3. 절댓값과 부호

#### Math.abs() - 절댓값

```javascript
console.log(Math.abs(-5));     // 5
console.log(Math.abs(5));      // 5
console.log(Math.abs(-3.14));  // 3.14
```

#### Math.sign() - 부호 확인 (ES6)

```javascript
console.log(Math.sign(5));     // 1 (양수)
console.log(Math.sign(-5));    // -1 (음수)
console.log(Math.sign(0));     // 0
console.log(Math.sign(-0));    // -0
console.log(Math.sign(NaN));   // NaN
```

### 4. 제곱과 제곱근

#### Math.pow() - 거듭제곱

```javascript
console.log(Math.pow(2, 3));   // 8 (2^3)
console.log(Math.pow(5, 2));   // 25 (5^2)
console.log(Math.pow(4, 0.5)); // 2 (√4)

// ES7 지수 연산자 (**)
console.log(2 ** 3);   // 8
console.log(5 ** 2);   // 25
```

#### Math.sqrt() - 제곱근

```javascript
console.log(Math.sqrt(4));    // 2
console.log(Math.sqrt(9));    // 3
console.log(Math.sqrt(2));    // 1.4142135623730951
console.log(Math.sqrt(-1));   // NaN (음수는 불가능)
```

#### Math.cbrt() - 세제곱근 (ES6)

```javascript
console.log(Math.cbrt(8));    // 2 (∛8)
console.log(Math.cbrt(27));   // 3 (∛27)
console.log(Math.cbrt(-8));   // -2 (음수 가능)
```

### 5. 랜덤 함수

#### Math.random() - 난수 생성

**0 이상 1 미만**의 부동소수점 난수를 반환합니다.

```javascript
console.log(Math.random());
// 0.697009826327621 (매번 다른 값)
```

#### 실용적인 랜덤 숫자 생성 패턴

```javascript
// 1. 0~9 사이의 정수
const random0to9 = Math.floor(Math.random() * 10);

// 2. 1~10 사이의 정수
const random1to10 = Math.floor(Math.random() * 10) + 1;

// 3. 최소값~최대값 사이의 정수 (범용 함수)
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

console.log(getRandomInt(1, 6));   // 주사위: 1~6
console.log(getRandomInt(1, 100)); // 1~100

// 4. 배열에서 랜덤 요소 선택
function getRandomElement(array) {
  return array[Math.floor(Math.random() * array.length)];
}

const fruits = ['🍎', '🍌', '🍇', '🍊'];
console.log(getRandomElement(fruits));  // 랜덤 과일

// 5. 배열 셔플 (Fisher-Yates)
function shuffle(array) {
  const shuffled = [...array];
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }
  return shuffled;
}

console.log(shuffle([1, 2, 3, 4, 5]));
// [3, 1, 5, 2, 4] (매번 다른 순서)
```

### 6. 삼각함수

```javascript
// 라디안 단위 사용
console.log(Math.sin(Math.PI / 2));  // 1 (sin 90°)
console.log(Math.cos(0));            // 1 (cos 0°)
console.log(Math.tan(Math.PI / 4));  // 1 (tan 45°)

// 도 → 라디안 변환
function toRadians(degrees) {
  return degrees * (Math.PI / 180);
}

console.log(Math.sin(toRadians(90)));  // 1

// 역삼각함수
console.log(Math.asin(1));   // π/2
console.log(Math.acos(1));   // 0
console.log(Math.atan(1));   // π/4
console.log(Math.atan2(1, 1)); // π/4 (2인자 버전)
```

## Math 상수

```javascript
console.log(Math.PI);      // 3.141592653589793 (원주율)
console.log(Math.E);       // 2.718281828459045 (자연로그 밑)
console.log(Math.LN2);     // 0.6931471805599453 (ln 2)
console.log(Math.LN10);    // 2.302585092994046 (ln 10)
console.log(Math.LOG2E);   // 1.4426950408889634 (log₂ e)
console.log(Math.LOG10E);  // 0.4342944819032518 (log₁₀ e)
console.log(Math.SQRT2);   // 1.4142135623730951 (√2)
console.log(Math.SQRT1_2); // 0.7071067811865476 (√½)
```

## Number 객체와 메서드

### 1. 숫자 변환

```javascript
// Number() - 타입 변환
console.log(Number('123'));      // 123
console.log(Number('12.34'));    // 12.34
console.log(Number(''));         // 0
console.log(Number('hello'));    // NaN
console.log(Number(true));       // 1
console.log(Number(false));      // 0

// parseInt() - 정수 파싱
console.log(parseInt('123'));      // 123
console.log(parseInt('123.99'));   // 123 (소수점 버림)
console.log(parseInt('123px'));    // 123 (숫자 부분만)
console.log(parseInt('px123'));    // NaN

// 진법 지정
console.log(parseInt('1010', 2));  // 10 (2진수)
console.log(parseInt('FF', 16));   // 255 (16진수)

// parseFloat() - 실수 파싱
console.log(parseFloat('3.14'));     // 3.14
console.log(parseFloat('3.14.15'));  // 3.14
console.log(parseFloat('3.14px'));   // 3.14
```

### 2. 소수점 처리

#### toFixed() - 고정 소수점

```javascript
const num = 3.14159;

console.log(num.toFixed(2));   // "3.14" (문자열!)
console.log(num.toFixed(0));   // "3"
console.log(num.toFixed(5));   // "3.14159"

// 숫자로 변환하려면
console.log(Number(num.toFixed(2)));     // 3.14
console.log(parseFloat(num.toFixed(2))); // 3.14
console.log(+num.toFixed(2));            // 3.14
```

#### toPrecision() - 전체 자릿수 지정

```javascript
const num = 123.456;

console.log(num.toPrecision(4));   // "123.5" (전체 4자리)
console.log(num.toPrecision(2));   // "1.2e+2" (지수 표기법)
console.log(num.toPrecision(7));   // "123.4560"
```

#### toExponential() - 지수 표기법

```javascript
const num = 123456;

console.log(num.toExponential());    // "1.23456e+5"
console.log(num.toExponential(2));   // "1.23e+5"
```

### 3. Number 상수

```javascript
console.log(Number.MAX_VALUE);          // 1.7976931348623157e+308
console.log(Number.MIN_VALUE);          // 5e-324
console.log(Number.MAX_SAFE_INTEGER);   // 9007199254740991 (2^53 - 1)
console.log(Number.MIN_SAFE_INTEGER);   // -9007199254740991
console.log(Number.POSITIVE_INFINITY);  // Infinity
console.log(Number.NEGATIVE_INFINITY);  // -Infinity
console.log(Number.NaN);                // NaN
console.log(Number.EPSILON);            // 2.220446049250313e-16 (부동소수점 오차)
```

### 4. Number 검증 메서드

```javascript
// Number.isNaN() - NaN 확인
console.log(Number.isNaN(NaN));       // true
console.log(Number.isNaN('hello'));   // false (타입 변환 안 함)
console.log(isNaN('hello'));          // true (전역 isNaN은 타입 변환)

// Number.isFinite() - 유한수 확인
console.log(Number.isFinite(42));       // true
console.log(Number.isFinite(Infinity)); // false
console.log(Number.isFinite('42'));     // false (타입 변환 안 함)

// Number.isInteger() - 정수 확인
console.log(Number.isInteger(42));      // true
console.log(Number.isInteger(42.0));    // true
console.log(Number.isInteger(42.1));    // false

// Number.isSafeInteger() - 안전한 정수 확인
console.log(Number.isSafeInteger(42));                    // true
console.log(Number.isSafeInteger(9007199254740991));     // true
console.log(Number.isSafeInteger(9007199254740992));     // false
```

## NaN 처리

### NaN이 발생하는 경우

```javascript
console.log(0 / 0);              // NaN
console.log(Math.sqrt(-1));      // NaN
console.log(parseInt('hello'));  // NaN
console.log(Number('abc'));      // NaN
console.log(NaN + 5);            // NaN (NaN과의 연산은 항상 NaN)
```

### NaN 확인하기

```javascript
const value = NaN;

// ❌ 직접 비교는 불가능
console.log(value === NaN);  // false (NaN은 자기 자신과도 다름!)

// ✅ Number.isNaN() 사용 (권장)
console.log(Number.isNaN(value));  // true

// ✅ Object.is() 사용
console.log(Object.is(value, NaN));  // true

// ⚠️ 전역 isNaN()은 타입 변환함
console.log(isNaN('hello'));        // true
console.log(Number.isNaN('hello')); // false
```

## 부동소수점 오차 처리

JavaScript는 IEEE 754 표준을 따르므로 **부동소수점 오차**가 발생합니다.

### 문제

```javascript
console.log(0.1 + 0.2);              // 0.30000000000000004 😱
console.log(0.1 + 0.2 === 0.3);      // false
console.log(0.3 - 0.1);              // 0.19999999999999998
```

### 해결 방법

```javascript
// 1. toFixed() 사용
const result = (0.1 + 0.2).toFixed(1);  // "0.3"
console.log(parseFloat(result) === 0.3); // true

// 2. Number.EPSILON 활용
function isEqual(a, b) {
  return Math.abs(a - b) < Number.EPSILON;
}
console.log(isEqual(0.1 + 0.2, 0.3));  // true

// 3. 정수로 변환 후 계산
function addDecimals(a, b) {
  const multiplier = 10;
  return (a * multiplier + b * multiplier) / multiplier;
}
console.log(addDecimals(0.1, 0.2));  // 0.3

// 4. 라이브러리 사용 (큰 프로젝트)
// - decimal.js
// - big.js
// - bignumber.js
```

## 실무 활용 예제

### 1. 가격 포맷팅

```javascript
function formatPrice(price) {
  return price.toLocaleString('ko-KR') + '원';
}

console.log(formatPrice(1234567));  // "1,234,567원"

// 소수점 포함
function formatCurrency(price) {
  return new Intl.NumberFormat('ko-KR', {
    style: 'currency',
    currency: 'KRW'
  }).format(price);
}

console.log(formatCurrency(1234567.89));  // "₩1,234,568"
```

### 2. 퍼센트 계산

```javascript
function calculatePercentage(value, total) {
  return ((value / total) * 100).toFixed(2) + '%';
}

console.log(calculatePercentage(75, 200));  // "37.50%"

// 할인율 계산
function getDiscountRate(original, discounted) {
  return Math.round(((original - discounted) / original) * 100);
}

console.log(getDiscountRate(10000, 7000));  // 30
```

### 3. 평점 계산

```javascript
function calculateRating(ratings) {
  const sum = ratings.reduce((acc, rating) => acc + rating, 0);
  const avg = sum / ratings.length;
  return Math.round(avg * 10) / 10;  // 소수점 1자리 반올림
}

const appRatings = [4.5, 5, 3.8, 4.2, 4.7];
console.log(calculateRating(appRatings));  // 4.4

// 별점 표시
function getStarRating(rating) {
  const fullStars = Math.floor(rating);
  const hasHalfStar = rating % 1 >= 0.5;
  const emptyStars = 5 - fullStars - (hasHalfStar ? 1 : 0);

  return '★'.repeat(fullStars) +
         (hasHalfStar ? '☆' : '') +
         '☆'.repeat(emptyStars);
}

console.log(getStarRating(4.4));  // "★★★★☆"
console.log(getStarRating(3.7));  // "★★★☆☆"
```

### 4. 랜덤 이벤트 (로또, 추첨)

```javascript
// 로또 번호 생성 (1~45 중 6개)
function generateLotto() {
  const numbers = [];
  while (numbers.length < 6) {
    const num = Math.floor(Math.random() * 45) + 1;
    if (!numbers.includes(num)) {
      numbers.push(num);
    }
  }
  return numbers.sort((a, b) => a - b);
}

console.log(generateLotto());  // [3, 12, 23, 28, 35, 42]

// 가중치 랜덤 선택
function weightedRandom(items, weights) {
  const totalWeight = weights.reduce((sum, w) => sum + w, 0);
  let random = Math.random() * totalWeight;

  for (let i = 0; i < items.length; i++) {
    random -= weights[i];
    if (random <= 0) {
      return items[i];
    }
  }
}

const prizes = ['1등', '2등', '3등', '꽝'];
const chances = [1, 5, 10, 84];  // 1%, 5%, 10%, 84%
console.log(weightedRandom(prizes, chances));  // 대부분 '꽝'
```

### 5. 통계 계산

```javascript
// 평균
function average(numbers) {
  return numbers.reduce((sum, num) => sum + num, 0) / numbers.length;
}

// 중앙값
function median(numbers) {
  const sorted = [...numbers].sort((a, b) => a - b);
  const mid = Math.floor(sorted.length / 2);

  if (sorted.length % 2 === 0) {
    return (sorted[mid - 1] + sorted[mid]) / 2;
  }
  return sorted[mid];
}

// 표준편차
function standardDeviation(numbers) {
  const avg = average(numbers);
  const squaredDiffs = numbers.map(num => Math.pow(num - avg, 2));
  const variance = average(squaredDiffs);
  return Math.sqrt(variance);
}

const scores = [85, 90, 78, 92, 88];
console.log('평균:', average(scores));              // 86.6
console.log('중앙값:', median(scores));             // 88
console.log('표준편차:', standardDeviation(scores)); // 5.02
```

### 6. 범위 제한 (Clamp)

```javascript
function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max);
}

console.log(clamp(50, 0, 100));    // 50
console.log(clamp(-10, 0, 100));   // 0
console.log(clamp(150, 0, 100));   // 100

// 볼륨 조절 예제
function setVolume(volume) {
  return clamp(volume, 0, 100);
}

console.log(setVolume(120));  // 100
console.log(setVolume(-5));   // 0
```

### 7. 거리 계산 (좌표)

```javascript
// 두 점 사이의 거리 (피타고라스 정리)
function getDistance(x1, y1, x2, y2) {
  const dx = x2 - x1;
  const dy = y2 - y1;
  return Math.sqrt(dx * dx + dy * dy);
}

console.log(getDistance(0, 0, 3, 4));  // 5

// 위도/경도 거리 (Haversine 공식)
function getGeoDistance(lat1, lon1, lat2, lon2) {
  const R = 6371; // 지구 반지름 (km)
  const dLat = toRadians(lat2 - lat1);
  const dLon = toRadians(lon2 - lon1);

  const a = Math.sin(dLat / 2) ** 2 +
            Math.cos(toRadians(lat1)) * Math.cos(toRadians(lat2)) *
            Math.sin(dLon / 2) ** 2;

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c;
}

function toRadians(degrees) {
  return degrees * (Math.PI / 180);
}

// 서울 → 부산
console.log(getGeoDistance(37.5665, 126.9780, 35.1796, 129.0756));
// 약 325km
```

## 비교 정리표

| 메서드 | 기능 | 예시 | 결과 |
|--------|------|------|------|
| `Math.round()` | 반올림 | `Math.round(2.5)` | `3` |
| `Math.ceil()` | 올림 | `Math.ceil(2.1)` | `3` |
| `Math.floor()` | 내림 | `Math.floor(2.9)` | `2` |
| `Math.trunc()` | 소수점 제거 | `Math.trunc(2.9)` | `2` |
| `Math.random()` | 난수 생성 | `Math.random()` | `0~1` |
| `toFixed(n)` | 소수점 n자리 | `(3.14).toFixed(1)` | `"3.1"` |
| `parseInt()` | 정수 변환 | `parseInt("42px")` | `42` |
| `parseFloat()` | 실수 변환 | `parseFloat("3.14")` | `3.14` |

## Best Practices

### ✅ 권장 사항

```javascript
// 1. 타입 안전한 검증 사용
if (Number.isNaN(value)) { }      // ✅
if (isNaN(value)) { }             // ⚠️ 타입 변환 발생

// 2. 부동소수점 비교 시 epsilon 사용
if (Math.abs(a - b) < Number.EPSILON) { }  // ✅

// 3. 안전한 정수 범위 확인
if (Number.isSafeInteger(bigNumber)) { }  // ✅

// 4. 범위 제한 함수 사용
const volume = Math.min(Math.max(value, 0), 100);  // ✅

// 5. 소수점 처리 후 숫자로 변환
const price = +value.toFixed(2);  // ✅
```

### ❌ 피해야 할 사항

```javascript
// 1. NaN 직접 비교
if (value === NaN) { }  // ❌ 항상 false

// 2. 부동소수점 직접 비교
if (0.1 + 0.2 === 0.3) { }  // ❌ false

// 3. parseInt 진법 생략
parseInt('08');  // ❌ 엣날 환경에서 8진수로 해석될 수 있음
parseInt('08', 10);  // ✅

// 4. toFixed 결과를 숫자로 착각
const result = num.toFixed(2);  // ❌ 문자열!
console.log(result + 1);  // "3.141" (문자열 연결)
```

## 핵심 정리

### Math 객체 주요 메서드

1. **반올림**: `round()`, `ceil()`, `floor()`, `trunc()`
2. **최대/최소**: `max()`, `min()`
3. **제곱/제곱근**: `pow()`, `sqrt()`, `cbrt()`
4. **랜덤**: `random()`
5. **절댓값**: `abs()`
6. **삼각함수**: `sin()`, `cos()`, `tan()`

### Number 객체 주요 메서드

1. **변환**: `Number()`, `parseInt()`, `parseFloat()`
2. **소수점**: `toFixed()`, `toPrecision()`
3. **검증**: `isNaN()`, `isFinite()`, `isInteger()`

### 주의사항

1. **부동소수점 오차** 항상 고려
2. **NaN**은 자기 자신과 같지 않음
3. **toFixed()** 결과는 문자열
4. **안전한 정수 범위** 확인 (`Number.MAX_SAFE_INTEGER`)

## 참고 자료

- [MDN - Number](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number)
- [MDN - Math](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Math)
- [IEEE 754 Floating Point](https://en.wikipedia.org/wiki/IEEE_754)
- [JavaScript Number Precision](https://javascript.info/number)
