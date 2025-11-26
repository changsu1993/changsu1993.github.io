---
title: JavaScript 데이터 타입 완벽 가이드 - 원시 타입과 참조 타입
description: JavaScript의 원시 타입(String, Number, Boolean, Undefined, Null, Symbol, BigInt)과 참조 타입(Object, Array, Function)을 상세히 알아봅니다. typeof 연산자, 타입 변환, 타입 체크 방법을 마스터합니다.
author: changsu
date: 2020-06-21 19:07:52 +0900
categories: [Programming, JavaScript]
tags: [javascript, data-types, type-conversion]
---

## JavaScript 데이터 타입

JavaScript는 **동적 타입 언어(Dynamically Typed Language)**로, 변수의 타입을 선언하지 않고 실행 시점에 자동으로 결정됩니다.

### 데이터 타입 분류

JavaScript의 데이터 타입은 크게 **원시 타입(Primitive Type)**과 **참조 타입(Reference Type)**으로 나뉩니다.

```javascript
// 원시 타입 (Primitive Types) - 7가지
// 1. String
// 2. Number
// 3. Boolean
// 4. Undefined
// 5. Null
// 6. Symbol (ES6)
// 7. BigInt (ES2020)

// 참조 타입 (Reference Type)
// Object (Array, Function, Date, RegExp 등 포함)
```

## typeof 연산자

**typeof 연산자**는 값의 데이터 타입을 문자열로 반환합니다.

### 기본 사용법

```javascript
let msg = "message";
console.log(typeof msg);   // "string"
console.log(typeof 100);   // "number"
console.log(typeof true);  // "boolean"
console.log(typeof undefined); // "undefined"
console.log(typeof Symbol()); // "symbol"
console.log(typeof 10n);   // "bigint"
```

### typeof 반환값

| 값 | typeof 결과 |
|-----|-------------|
| `"hello"` | `"string"` |
| `123` | `"number"` |
| `true` | `"boolean"` |
| `undefined` | `"undefined"` |
| `null` | `"object"` ⚠️ |
| `Symbol()` | `"symbol"` |
| `10n` | `"bigint"` |
| `{}`, `[]` | `"object"` |
| `function() {}` | `"function"` |

### typeof의 특이점

```javascript
// null은 object로 반환됨 (JavaScript의 버그)
console.log(typeof null);  // "object" ⚠️

// 배열도 object로 반환됨
console.log(typeof []);    // "object"
console.log(typeof {});    // "object"

// 함수는 function으로 반환됨
console.log(typeof function() {});  // "function"

// NaN은 number로 반환됨
console.log(typeof NaN);   // "number"
```

## 원시 타입 (Primitive Types)

### 1. String (문자열)

따옴표(`""`, `''`, `` ` ``)로 감싼 텍스트입니다.

```javascript
let str1 = "Hello";         // 쌍따옴표
let str2 = 'World';         // 홑따옴표
let str3 = `Template`;      // 백틱 (ES6)

console.log(typeof str1);   // "string"
console.log(typeof str2);   // "string"
console.log(typeof str3);   // "string"

// 빈 문자열
let empty = "";
console.log(typeof empty);  // "string"

// 숫자처럼 보이지만 문자열
let numStr = "123";
console.log(typeof numStr); // "string"
```

### 2. Number (숫자)

정수와 부동소수점 숫자를 표현합니다.

```javascript
let int = 42;
let float = 3.14;
let negative = -10;

console.log(typeof int);    // "number"
console.log(typeof float);  // "number"

// 특수 숫자 값
console.log(typeof Infinity);   // "number"
console.log(typeof -Infinity);  // "number"
console.log(typeof NaN);        // "number" (Not a Number)

// NaN 예시
let result = "abc" * 3;
console.log(result);        // NaN
console.log(typeof result); // "number"
console.log(isNaN(result)); // true
```

### 3. Boolean (불리언)

참(`true`) 또는 거짓(`false`) 값입니다.

```javascript
let isTrue = true;
let isFalse = false;

console.log(typeof isTrue);   // "boolean"
console.log(typeof isFalse);  // "boolean"

// 비교 연산 결과
let result = 5 > 3;
console.log(result);        // true
console.log(typeof result); // "boolean"
```

### 4. Undefined

변수가 선언되었지만 값이 할당되지 않은 상태입니다.

```javascript
let msg;
console.log(msg);           // undefined
console.log(typeof msg);    // "undefined"

// 명시적 undefined 할당
let value = undefined;
console.log(value);         // undefined

// 함수에서 return이 없을 때
function test() {
  // return 없음
}
console.log(test());        // undefined
```

### 5. Null

의도적으로 "값이 없음"을 나타냅니다.

```javascript
let empty = null;
console.log(empty);         // null
console.log(typeof empty);  // "object" ⚠️ (버그)

// null 체크
console.log(empty === null);  // true (올바른 방법)
```

#### undefined vs null

```javascript
// undefined: 값이 할당되지 않음 (시스템이 할당)
let a;
console.log(a);  // undefined

// null: 명시적으로 비어있음을 표현 (개발자가 할당)
let b = null;
console.log(b);  // null

// 비교
console.log(a == b);   // true (타입 변환 후 같음)
console.log(a === b);  // false (타입이 다름)

console.log(typeof a); // "undefined"
console.log(typeof b); // "object"
```

### 6. Symbol (ES6)

유일하고 변경 불가능한 값입니다.

```javascript
let sym1 = Symbol();
let sym2 = Symbol();
let sym3 = Symbol("description");

console.log(typeof sym1);   // "symbol"
console.log(sym1 === sym2); // false (각각 유일함)

// 객체 프로퍼티 키로 사용
let obj = {
  [sym1]: "value1",
  [sym2]: "value2"
};

console.log(obj[sym1]);  // "value1"
```

### 7. BigInt (ES2020)

아주 큰 정수를 표현합니다.

```javascript
let bigNum = 9007199254740991n;  // 끝에 n 추가
let anotherBig = BigInt(9007199254740991);

console.log(typeof bigNum);  // "bigint"

// 일반 Number와 연산 불가
let num = 10;
// console.log(bigNum + num);  // TypeError

// BigInt끼리만 연산 가능
console.log(bigNum + 1n);  // 9007199254740992n
```

## 참조 타입 (Reference Type)

### Object (객체)

키-값 쌍으로 이루어진 데이터 집합입니다.

```javascript
let person = {
  name: "홍길동",
  age: 30,
  city: "서울"
};

console.log(typeof person);  // "object"
console.log(person.name);    // "홍길동"

// 빈 객체
let empty = {};
console.log(typeof empty);   // "object"
```

### Array (배열)

순서가 있는 데이터 목록입니다. 배열도 객체입니다.

```javascript
let fruits = ["사과", "바나나", "오렌지"];

console.log(typeof fruits);           // "object"
console.log(Array.isArray(fruits));  // true (배열 체크)

// 배열 확인 방법
console.log(fruits instanceof Array);  // true
console.log(fruits.constructor === Array); // true
```

### Function (함수)

함수도 객체이지만 typeof는 "function"을 반환합니다.

```javascript
function greet() {
  return "Hello";
}

console.log(typeof greet);  // "function"

// 함수 표현식
let add = function(a, b) {
  return a + b;
};

console.log(typeof add);    // "function"

// 화살표 함수
let multiply = (a, b) => a * b;
console.log(typeof multiply); // "function"
```

## truthy와 falsy

JavaScript는 Boolean이 아닌 값도 조건문에서 true/false로 평가합니다.

### falsy 값 (6가지)

다음 값들은 조건문에서 **false로 평가**됩니다:

```javascript
// 1. false
if (false) {
  // 실행되지 않음
}

// 2. 0
if (0) {
  // 실행되지 않음
}

// 3. "" (빈 문자열)
if ("") {
  // 실행되지 않음
}

// 4. null
if (null) {
  // 실행되지 않음
}

// 5. undefined
if (undefined) {
  // 실행되지 않음
}

// 6. NaN
if (NaN) {
  // 실행되지 않음
}
```

### truthy 값

falsy 값을 제외한 **모든 값은 true로 평가**됩니다:

```javascript
// 비어있지 않은 문자열
if ("hello") {
  console.log("실행됨");  // 실행됨
}

// 0이 아닌 숫자
if (1) {
  console.log("실행됨");  // 실행됨
}

if (-1) {
  console.log("실행됨");  // 실행됨
}

// 빈 배열도 truthy
if ([]) {
  console.log("실행됨");  // 실행됨
}

// 빈 객체도 truthy
if ({}) {
  console.log("실행됨");  // 실행됨
}

// 문자열 "0"도 truthy
if ("0") {
  console.log("실행됨");  // 실행됨
}
```

### 실전 활용

```javascript
// 값 존재 확인
let userName = "";

if (userName) {
  console.log(`환영합니다, ${userName}님`);
} else {
  console.log("이름을 입력해주세요");  // 실행됨
}

// 기본값 설정 (OR 연산자)
let name = userName || "손님";
console.log(name);  // "손님"

// Nullish coalescing (ES2020)
let name2 = userName ?? "손님";
console.log(name2);  // "손님"

// 0은 유효한 값일 때
let count = 0;
let result1 = count || 10;      // 10 (0은 falsy)
let result2 = count ?? 10;      // 0 (null/undefined만 체크)
```

## 타입 변환

### 명시적 변환 (Explicit Conversion)

#### String으로 변환

```javascript
let num = 123;

// String() 함수
let str1 = String(num);
console.log(str1, typeof str1);  // "123" "string"

// toString() 메서드
let str2 = num.toString();
console.log(str2, typeof str2);  // "123" "string"

// 템플릿 리터럴
let str3 = `${num}`;
console.log(str3, typeof str3);  // "123" "string"
```

#### Number로 변환

```javascript
let str = "123";

// Number() 함수
let num1 = Number(str);
console.log(num1, typeof num1);  // 123 "number"

// parseInt() - 정수 변환
let num2 = parseInt(str);
console.log(num2);  // 123

let num3 = parseInt("123.45");
console.log(num3);  // 123 (소수점 버림)

// parseFloat() - 실수 변환
let num4 = parseFloat("123.45");
console.log(num4);  // 123.45

// 단항 + 연산자
let num5 = +str;
console.log(num5, typeof num5);  // 123 "number"

// 변환 실패
console.log(Number("abc"));      // NaN
console.log(parseInt("abc"));    // NaN
console.log(parseInt("123abc")); // 123 (숫자 부분만 파싱)
```

#### Boolean으로 변환

```javascript
// Boolean() 함수
console.log(Boolean(1));        // true
console.log(Boolean(0));        // false
console.log(Boolean("hello"));  // true
console.log(Boolean(""));       // false

// !! (이중 부정)
console.log(!!"hello");  // true
console.log(!!"");       // false
console.log(!!0);        // false
console.log(!!1);        // true
```

### 암시적 변환 (Implicit Conversion)

```javascript
// 문자열 + 숫자 = 문자열
console.log("5" + 3);    // "53"
console.log(5 + "3");    // "53"

// 문자열 - 숫자 = 숫자
console.log("5" - 3);    // 2
console.log("10" * "2"); // 20
console.log("10" / "2"); // 5

// Boolean 변환
if (1) {
  console.log("1은 truthy");  // 실행됨
}

if ("hello") {
  console.log("문자열은 truthy");  // 실행됨
}

// 비교 연산
console.log("5" == 5);   // true (타입 변환)
console.log("5" === 5);  // false (타입까지 비교)
```

## 정확한 타입 체크

### null 체크

```javascript
let value = null;

// typeof는 정확하지 않음
console.log(typeof value);  // "object" ⚠️

// 정확한 체크
console.log(value === null);  // true ✅
```

### 배열 체크

```javascript
let arr = [1, 2, 3];

// typeof는 정확하지 않음
console.log(typeof arr);  // "object" ⚠️

// 정확한 체크
console.log(Array.isArray(arr));  // true ✅
console.log(arr instanceof Array); // true ✅
```

### 객체 타입 체크

```javascript
function getType(value) {
  return Object.prototype.toString.call(value).slice(8, -1);
}

console.log(getType(123));           // "Number"
console.log(getType("hello"));       // "String"
console.log(getType(true));          // "Boolean"
console.log(getType([]));            // "Array"
console.log(getType({}));            // "Object"
console.log(getType(null));          // "Null"
console.log(getType(undefined));     // "Undefined"
console.log(getType(function(){}));  // "Function"
console.log(getType(new Date()));    // "Date"
```

## 원시 타입 vs 참조 타입

### 값 복사 vs 참조 복사

```javascript
// 원시 타입: 값 복사
let a = 10;
let b = a;
b = 20;

console.log(a);  // 10 (변경되지 않음)
console.log(b);  // 20

// 참조 타입: 참조 복사
let obj1 = { name: "홍길동" };
let obj2 = obj1;
obj2.name = "김철수";

console.log(obj1.name);  // "김철수" (같이 변경됨)
console.log(obj2.name);  // "김철수"

// 배열도 마찬가지
let arr1 = [1, 2, 3];
let arr2 = arr1;
arr2.push(4);

console.log(arr1);  // [1, 2, 3, 4]
console.log(arr2);  // [1, 2, 3, 4]
```

### 깊은 복사

```javascript
// 객체 깊은 복사
let original = { name: "홍길동", age: 30 };

// 방법 1: Spread 연산자 (얕은 복사)
let copy1 = { ...original };
copy1.name = "김철수";
console.log(original.name);  // "홍길동" (변경 안됨)

// 방법 2: Object.assign() (얕은 복사)
let copy2 = Object.assign({}, original);

// 방법 3: JSON 사용 (깊은 복사, 단 함수/undefined는 복사 안됨)
let copy3 = JSON.parse(JSON.stringify(original));

// 배열 깊은 복사
let arr = [1, 2, 3];
let arrCopy1 = [...arr];
let arrCopy2 = arr.slice();
let arrCopy3 = JSON.parse(JSON.stringify(arr));
```

## 실전 예제

### 1. 타입 안전한 함수

```javascript
function add(a, b) {
  // 타입 체크
  if (typeof a !== "number" || typeof b !== "number") {
    throw new TypeError("인자는 숫자여야 합니다");
  }

  return a + b;
}

console.log(add(5, 3));     // 8
// console.log(add("5", 3));  // TypeError
```

### 2. 기본값 설정

```javascript
function greet(name) {
  // falsy 체크
  name = name || "손님";
  console.log(`안녕하세요, ${name}님!`);
}

greet("홍길동");  // "안녕하세요, 홍길동님!"
greet();          // "안녕하세요, 손님님!"
greet("");        // "안녕하세요, 손님님!"

// Nullish coalescing 사용 (더 정확)
function greet2(name) {
  name = name ?? "손님";
  console.log(`안녕하세요, ${name}님!`);
}

greet2(0);    // "안녕하세요, 0님!" (0도 유효한 값으로 취급)
greet2("");   // "안녕하세요, 님!" (빈 문자열도 유효)
```

### 3. 타입 변환 유틸리티

```javascript
function toNumber(value) {
  const num = Number(value);
  return isNaN(num) ? 0 : num;
}

console.log(toNumber("123"));   // 123
console.log(toNumber("abc"));   // 0
console.log(toNumber(null));    // 0

function toString(value) {
  return value == null ? "" : String(value);
}

console.log(toString(123));       // "123"
console.log(toString(null));      // ""
console.log(toString(undefined)); // ""
```

## 마치며

JavaScript의 데이터 타입을 정확히 이해하는 것은 버그 없는 코드 작성의 기초입니다:

1. **원시 타입**: String, Number, Boolean, Undefined, Null, Symbol, BigInt
2. **참조 타입**: Object (Array, Function 포함)
3. **typeof 연산자**: 타입 확인 (null, 배열 주의)
4. **truthy/falsy**: 조건문에서 Boolean으로 평가
5. **타입 변환**: 명시적 변환 권장
6. **값 복사 vs 참조 복사**: 원시 타입과 참조 타입의 차이

엄격한 비교(`===`)를 사용하고, 명시적 타입 변환을 통해 예상치 못한 버그를 방지하세요.

## 참고 자료

- [MDN - JavaScript data types](https://developer.mozilla.org/ko/docs/Web/JavaScript/Data_structures)
- [MDN - typeof](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/typeof)
- [MDN - Type coercion](https://developer.mozilla.org/ko/docs/Glossary/Type_coercion)
- [MDN - Primitive](https://developer.mozilla.org/ko/docs/Glossary/Primitive)
- [MDN - Symbol](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
- [MDN - BigInt](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
- [JavaScript.info - Data types](https://javascript.info/types)

