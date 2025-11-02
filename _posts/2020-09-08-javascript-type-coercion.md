---
title: JavaScript Type Coercion 완벽 가이드 - 형 변환의 모든 것
description: JavaScript의 암묵적 형 변환(Type Coercion)을 마스터하세요. 숫자/문자열 변환, Truthy/Falsy, 객체 변환, 그리고 실무에서 피해야 할 위험한 패턴까지 완벽 정리합니다.
author: cotes
date: 2020-09-08 22:20:16 +0900
categories: [JavaScript, Core Concepts]
tags: [javascript, type-coercion, truthy-falsy, implicit-conversion, type-conversion]
---

## Type Coercion이란?

**Type Coercion(형 변환)**은 JavaScript가 **예상치 못한 타입을 받았을 때 자동으로 타입을 변환**하는 것을 말합니다. 이는 JavaScript의 주요 기능이지만, 동시에 **가장 많은 버그를 유발**하는 원인이기도 합니다.

```javascript
1 + "1" + 1    // "111" 😱
2 * "2"        // 4
true + true    // 2
1 + false      // 1
2 - true       // 1
```

> **주의**: 형 변환은 편리하지만, 예측 불가능한 결과를 초래할 수 있습니다!
{: .prompt-warning }

## 명시적 vs 암묵적 형 변환

### 명시적 형 변환 (Explicit Coercion)

개발자가 **의도적으로** 타입을 변환하는 것입니다.

```javascript
// ✅ 명시적 형 변환
const num = Number("123");        // 123
const str = String(123);          // "123"
const bool = Boolean(1);          // true

const num2 = parseInt("42px");    // 42
const float = parseFloat("3.14"); // 3.14
```

### 암묵적 형 변환 (Implicit Coercion)

JavaScript 엔진이 **자동으로** 타입을 변환하는 것입니다.

```javascript
// ⚠️ 암묵적 형 변환
const result1 = "5" - 2;    // 3 (문자열 → 숫자)
const result2 = "5" + 2;    // "52" (숫자 → 문자열)
const result3 = !!"hello";  // true (문자열 → 불리언)
```

## 숫자 표현식에서의 형 변환

### 문자열을 숫자로 변환

**숫자 문자**만 포함된 문자열은 숫자로 변환되지만, **숫자가 아닌 문자**가 포함되면 `NaN`이 됩니다.

```javascript
// ✅ 성공적인 변환
Number("2")      // 2
Number("3")      // 3
Number("1.")     // 1
Number("1.23")   // 1.23
Number("0")      // 0
Number("012")    // 12 (8진수 아님!)

2 * "2"          // 4 (자동 변환)
2 * Number("2")  // 4 (명시적 변환)

// ❌ NaN 반환
Number("1,")     // NaN
Number("1+1")    // NaN
Number("1a")     // NaN
Number("ab")     // NaN
Number("one")    // NaN
Number("text")   // NaN
```

### 다른 타입을 숫자로 변환

```javascript
Number(true)       // 1
Number(false)      // 0
Number(null)       // 0
Number(undefined)  // NaN
Number("")         // 0
Number(" ")        // 0

// 연산에서 자동 변환
1 + true    // 2
1 + false   // 1
2 - true    // 1
1 * ""      // 0
```

## + 연산자의 특별한 동작

`+` 연산자는 **두 가지 기능**을 수행합니다:
1. **수학적 덧셈**
2. **문자열 연결 (String Concatenation)**

### 규칙

- **피연산자 중 하나라도 문자열**이면 → 문자열 연결
- **둘 다 숫자**면 → 수학적 덧셈

```javascript
// 문자열 연결
1 + "2"       // "12"
1 + "ab"      // "1ab"
"" + 123      // "123"

// 수학적 덧셈
1 + 2         // 3
1 + 2 + 3     // 6

// 혼합 - 왼쪽에서 오른쪽으로 평가
1 + 2 + "3"   // "33"  (1 + 2 = 3, 3 + "3" = "33")
(1 + 2) + "3" // "33"  (동일)

1 + "2" + 3   // "123" (1 + "2" = "12", "12" + 3 = "123")
(1 + "2") + 3 // "123" (동일)

// 다른 연산자는 숫자로 변환
"5" - 2       // 3
"5" * 2       // 10
"10" / 2      // 5
"5" % 2       // 1
```

### 실무 팁: 숫자를 문자열로 빠르게 변환

```javascript
const num = 123;

// ✅ 명시적 (권장)
const str1 = String(num);      // "123"
const str2 = num.toString();   // "123"

// ⚠️ 암묵적 (짧지만 혼란스러울 수 있음)
const str3 = num + "";         // "123"
const str4 = `${num}`;         // "123"
```

### 실무 팁: 문자열을 숫자로 빠르게 변환

```javascript
const str = "123";

// ✅ 명시적 (권장)
const num1 = Number(str);      // 123
const num2 = parseInt(str);    // 123

// ⚠️ 암묵적 (짧지만 혼란스러울 수 있음)
const num3 = +str;             // 123 (단항 + 연산자)
const num4 = str - 0;          // 123
const num5 = str * 1;          // 123
```

## 객체의 형 변환

### toString() 메서드

모든 JavaScript 객체는 `toString()` 메서드를 상속받습니다. 객체가 **문자열로 변환**될 때 호출됩니다.

```javascript
const obj = {};
obj.toString();  // "[object Object]"

"abc" + {}       // "abc[object Object]"

// toString 커스터마이징
const custom = {
  toString: () => "example"
};

"test " + custom;  // "test example"
custom + " good!"; // "example good!"
```

### 수학적 표현식에서의 객체

객체가 **수학적 표현식**에 사용되면, `toString()`의 반환값을 **숫자로 변환**하려 시도합니다.

```javascript
const obj1 = {
  toString: () => 2
};

1 * obj1;      // 2
2 + obj1;      // 4
2 / obj1;      // 1
"text" + obj1; // "text2" (문자열 연결)

const obj2 = {
  toString: () => "text"
};

1 + obj2;      // "1text" (숫자가 문자열로 변환)
2 * obj2;      // NaN (문자열을 숫자로 변환 실패)

const obj3 = {
  toString: () => "3"
};

1 + obj3;      // "13" (문자열 연결)
2 * obj3;      // 6 ("3" → 3 변환 성공)
```

### valueOf() 메서드

`valueOf()`는 객체의 **원시 값(primitive value)**을 반환합니다. 수학적 연산에서 우선적으로 사용됩니다.

```javascript
const obj = {
  valueOf: () => 5
};

1 + obj;  // 6
2 * obj;  // 10

// valueOf가 있으면 우선 사용
const obj2 = {
  valueOf: () => 2,
  toString: () => 3
};

"a" + obj2;  // "a2" (valueOf 우선)
2 + obj2;    // 4
2 * obj2;    // 4
```

> **우선순위**: 수학적 연산에서는 `valueOf()` > `toString()` 순서로 호출됩니다.
{: .prompt-tip }

### 실전 예제: Date 객체

```javascript
const date = new Date();

// toString() 사용
String(date);  // "Tue Jan 15 2024 10:30:00 GMT+0900"

// valueOf() 사용 (타임스탬프)
Number(date);  // 1705284600000
+date;         // 1705284600000

// 연산
date - 0;      // 1705284600000 (숫자)
date + "";     // "Tue Jan 15 2024 10:30:00 GMT+0900" (문자열)
```

## 배열의 형 변환

배열의 `toString()`은 `join()`과 유사하게 동작합니다.

```javascript
[].toString()        // ""
[].join()            // ""

[1, 2, 3].toString() // "1,2,3"
[1, 2, 3].join()     // "1,2,3"

// 연산에서의 배열
"a" + [1, 2, 3]      // "a1,2,3"
1 + [1, 2, 3]        // "11,2,3" (문자열 연결)
2 * [1, 2, 3]        // NaN ("1,2,3"을 숫자로 변환 실패)

// 빈 배열과 단일 요소 배열
2 * []               // 0
2 / [2]              // 1
2 + [2]              // "22" (문자열 연결)
2 + [1, 2]           // "21,2"

// 변환 과정
2 * []
→ 2 * [].toString()
→ 2 * ""
→ 2 * Number("")
→ 2 * 0
→ 0

2 / [2]
→ 2 / [2].toString()
→ 2 / "2"
→ 2 / Number("2")
→ 2 / 2
→ 1
```

## Truthy와 Falsy

모든 JavaScript 값은 **Boolean 컨텍스트**에서 `true` 또는 `false`로 변환됩니다.

### Falsy 값 (8개)

다음 값들만 `false`로 변환됩니다:

```javascript
Boolean(false)      // false
Boolean(0)          // false
Boolean(-0)         // false
Boolean(0n)         // false (BigInt)
Boolean("")         // false (빈 문자열)
Boolean(null)       // false
Boolean(undefined)  // false
Boolean(NaN)        // false
```

### Truthy 값

**Falsy 값을 제외한 모든 값**은 `true`로 변환됩니다.

```javascript
Boolean(true)       // true
Boolean(1)          // true
Boolean(-1)         // true
Boolean("0")        // true (문자열 "0"은 truthy!)
Boolean("false")    // true (문자열 "false"도 truthy!)
Boolean(" ")        // true (공백 포함 문자열)
Boolean([])         // true (빈 배열)
Boolean({})         // true (빈 객체)
Boolean(function(){}) // true (함수)
```

### 조건문에서의 활용

```javascript
// ✅ Truthy
if (1) { }            // 실행됨
if ("0") { }          // 실행됨
if ({}) { }           // 실행됨
if ([]) { }           // 실행됨
if (function() {}) { } // 실행됨

// ❌ Falsy
if (0) { }            // 실행 안 됨
if ("") { }           // 실행 안 됨
if (null) { }         // 실행 안 됨
if (undefined) { }    // 실행 안 됨

// 삼항 연산자
"string" ? 1 : 2      // 1 (truthy)
undefined ? 1 : 2     // 2 (falsy)
```

### 논리 연산자 활용

```javascript
// && (AND) - 첫 번째 falsy 또는 마지막 값 반환
true && "hello"       // "hello"
"hello" && "world"    // "world"
"hello" && 0          // 0
0 && "hello"          // 0

// || (OR) - 첫 번째 truthy 또는 마지막 값 반환
false || "hello"      // "hello"
0 || 100              // 100
null || undefined     // undefined
"" || "default"       // "default"

// 실무 예제: 기본값 설정
const name = userName || "Guest";
const count = userCount || 0;

// ?? (Nullish Coalescing) - null/undefined만 체크
const value1 = 0 ?? 100;        // 0 (0은 nullish 아님)
const value2 = null ?? 100;     // 100
const value3 = undefined ?? 100; // 100
```

## NaN (Not a Number)

### NaN의 특징

`NaN`은 JavaScript에서 **유일하게 자기 자신과 같지 않은 값**입니다.

```javascript
NaN === NaN  // false 😱

const result = 1 * "a";  // NaN
result == result         // false
result === result        // false
```

### NaN이 발생하는 경우

```javascript
// 숫자 변환 실패
Number("abc")     // NaN
parseInt("xyz")   // NaN
parseFloat("foo") // NaN

// 잘못된 수학 연산
0 / 0             // NaN
Infinity - Infinity  // NaN
Math.sqrt(-1)     // NaN

// NaN과의 연산
NaN + 5           // NaN
NaN * 10          // NaN
NaN - NaN         // NaN
```

### NaN 확인 방법

```javascript
// ❌ 직접 비교 불가
const value = NaN;
value === NaN  // false (작동 안 함!)

// ✅ Number.isNaN() 사용 (권장)
Number.isNaN(NaN)      // true
Number.isNaN("hello")  // false
Number.isNaN(123)      // false

// ⚠️ 전역 isNaN()은 형 변환함 (비권장)
isNaN("hello")    // true (Number("hello") = NaN)
isNaN("123")      // false (Number("123") = 123)
isNaN(NaN)        // true

// ✅ Object.is() 사용
Object.is(NaN, NaN)  // true

// ✅ 자기 자신과 비교 (트릭)
function isNaNValue(value) {
  return value !== value;
}
isNaNValue(NaN)  // true
```

### 실무 예제: 안전한 숫자 변환

```javascript
function safeParseNumber(value) {
  const num = Number(value);
  return Number.isNaN(num) ? 0 : num;
}

console.log(safeParseNumber("123"));   // 123
console.log(safeParseNumber("abc"));   // 0
console.log(safeParseNumber("12.5"));  // 12.5
```

## 위험한 패턴과 피해야 할 코드

### ❌ 나쁜 예: 암묵적 형 변환 의존

```javascript
// ❌ 값의 존재 여부를 truthy로만 체크
function processNumber(num) {
  if (!num) {
    throw new Error("숫자가 필요합니다");
  }
  return num * 2;
}

processNumber(0);  // Error! (0은 유효한 숫자인데 falsy)

// ✅ 올바른 방법
function processNumber(num) {
  if (typeof num !== "number") {
    throw new Error("숫자가 필요합니다");
  }
  return num * 2;
}

processNumber(0);  // 0 (정상 작동)
```

### ❌ 나쁜 예: + 연산자로 문자열 연결

```javascript
// ❌ 예측 불가능
const price = 100;
const tax = 10;
const message = "합계: " + price + tax;  // "합계: 10010" 😱

// ✅ 올바른 방법
const message = `합계: ${price + tax}`;  // "합계: 110"
```

### ❌ 나쁜 예: == 연산자 사용

```javascript
// ❌ == 는 형 변환을 수행함
"0" == 0          // true
false == 0        // true
null == undefined // true
[] == false       // true 😱
[] == ![]         // true 😱😱

// ✅ === 사용 (권장)
"0" === 0          // false
false === 0        // false
null === undefined // false
[] === false       // false
```

### ❌ 나쁜 예: 배열/객체 비교

```javascript
// ❌ 형 변환이 발생함
if ([] == false) { }  // true
if ({} == false) { }  // false

// ✅ 명시적으로 체크
if (array.length === 0) { }
if (Object.keys(obj).length === 0) { }
```

## Best Practices

### ✅ 권장 사항

```javascript
// 1. 항상 === 사용
if (value === 0) { }        // ✅
if (value == 0) { }         // ❌

// 2. 명시적 형 변환
const num = Number(str);    // ✅
const num = +str;           // ⚠️ 짧지만 혼란스러울 수 있음

// 3. 타입 체크
if (typeof value === "number") { }  // ✅
if (value) { }                      // ❌ (0, "", false 등도 제외됨)

// 4. 안전한 기본값 설정
const name = userName ?? "Guest";   // ✅ (null/undefined만 체크)
const name = userName || "Guest";   // ⚠️ ("", 0도 제외됨)

// 5. NaN 체크
if (Number.isNaN(value)) { }  // ✅
if (isNaN(value)) { }         // ❌ (형 변환 수행)
if (value !== value) { }      // ⚠️ (작동하지만 의미 불명확)
```

### 형 변환 체크리스트

| 상황 | ❌ 피해야 할 코드 | ✅ 권장 코드 |
|------|----------------|------------|
| **동등 비교** | `value == 0` | `value === 0` |
| **타입 변환** | `+str` | `Number(str)` |
| **존재 여부** | `if (value) { }` | `if (value !== undefined) { }` |
| **NaN 체크** | `isNaN(x)` | `Number.isNaN(x)` |
| **기본값** | `x || default` | `x ?? default` |
| **문자열 연결** | `"" + num` | `String(num)` 또는 `` `${num}` `` |

## 형 변환 치트 시트

### To String

```javascript
String(123)        // "123"
(123).toString()   // "123"
123 + ""           // "123" (암묵적)
`${123}`           // "123" (템플릿 리터럴)
```

### To Number

```javascript
Number("123")      // 123
parseInt("123")    // 123
parseFloat("12.3") // 12.3
+"123"             // 123 (암묵적)
"123" - 0          // 123 (암묵적)
"123" * 1          // 123 (암묵적)
```

### To Boolean

```javascript
Boolean(value)     // 명시적
!!value            // 암묵적 (이중 부정)
value ? true : false  // 삼항 연산자
```

## 실무 활용 패턴

### 1. 안전한 숫자 파싱

```javascript
function parseNumber(value, defaultValue = 0) {
  const num = Number(value);
  return Number.isNaN(num) ? defaultValue : num;
}

console.log(parseNumber("123"));     // 123
console.log(parseNumber("abc"));     // 0
console.log(parseNumber("abc", -1)); // -1
```

### 2. 안전한 배열 체크

```javascript
function isNonEmptyArray(value) {
  return Array.isArray(value) && value.length > 0;
}

console.log(isNonEmptyArray([]));        // false
console.log(isNonEmptyArray([1, 2, 3])); // true
console.log(isNonEmptyArray("array"));   // false
```

### 3. Nullish 체크

```javascript
function isNullish(value) {
  return value === null || value === undefined;
}

// 또는 간단하게
function isNullish(value) {
  return value == null;  // == 를 사용하면 null과 undefined 모두 체크
}

console.log(isNullish(null));      // true
console.log(isNullish(undefined)); // true
console.log(isNullish(0));         // false
console.log(isNullish(""));        // false
```

### 4. 안전한 JSON 파싱

```javascript
function safeParse(jsonString, defaultValue = {}) {
  try {
    const result = JSON.parse(jsonString);
    return result ?? defaultValue;
  } catch {
    return defaultValue;
  }
}

console.log(safeParse('{"name": "John"}'));  // { name: "John" }
console.log(safeParse('invalid json'));      // {}
console.log(safeParse('null', []));          // []
```

## 핵심 정리

### 기억해야 할 규칙

1. **+ 연산자**는 문자열이 하나라도 있으면 문자열 연결
2. **다른 연산자**(-. *, /, %)는 숫자로 변환 시도
3. **객체 변환** 시 `valueOf()` → `toString()` 순서로 호출
4. **Falsy 값**은 8개뿐 (false, 0, -0, 0n, "", null, undefined, NaN)
5. **NaN은 자기 자신과 다름** (유일한 값)
6. **===을 항상 사용**하여 형 변환 방지

### 안전한 코드 작성법

```javascript
// ✅ Do
- 명시적 형 변환 사용
- === 연산자 사용
- typeof로 타입 체크
- Number.isNaN() 사용
- ?? 연산자로 null/undefined만 체크

// ❌ Don't
- 암묵적 형 변환 의존
- == 연산자 사용
- truthy/falsy로만 검증
- 전역 isNaN() 사용
- 복잡한 형 변환 체인
```

> **결론**: 형 변환은 편리하지만 예측 불가능합니다. 항상 **명시적 변환**을 사용하고, **타입을 확실히 체크**하여 버그를 예방하세요!
{: .prompt-tip }

## 참고 자료

- [MDN - Type Coercion](https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion)
- [JavaScript Equality Table](https://dorey.github.io/JavaScript-Equality-Table/)
- [ECMA-262 Specification](https://tc39.es/ecma262/)
- [You Don't Know JS: Types & Grammar](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/types%20%26%20grammar/README.md)
