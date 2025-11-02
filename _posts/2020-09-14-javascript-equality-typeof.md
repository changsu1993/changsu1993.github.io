---
title: JavaScript 타입 비교 완벽 가이드 - == vs === vs typeof
date: 2020-09-14 00:01:00 +0900
categories: [개발, JavaScript]
tags: [javascript, 타입비교, ==, ===, typeof, instanceof, 동등연산자, 형변환]
author: changsu
toc: true
comments: true
description: JavaScript의 동등 연산자(==, ===)와 타입 검사(typeof, instanceof)를 완벽하게 이해합니다. 실무에서 발생하는 흔한 버그를 예방하는 방법을 알아봅니다.
---

## 들어가며

JavaScript에서 값을 비교하는 것은 간단해 보이지만, **타입 강제 변환**(Type Coercion) 때문에 예상치 못한 결과가 나올 수 있습니다.

`==`와 `===`의 차이를 정확히 이해하지 못하면 **치명적인 버그**가 발생할 수 있습니다.

> `0 == false`는 `true`이지만, `0 === false`는 `false`입니다!
{: .prompt-warning }

## 동등 연산자: == vs ===

### === (엄격한 동등 비교)

**`===`**는 **타입과 값**이 **모두 같은지** 검사합니다.

#### 기본 원리

```javascript
// 타입과 값이 모두 같아야 true
3 === 3           // true (number, 3)
'hello' === 'hello'   // true (string, 'hello')
true === true     // true (boolean, true)

// 타입이 다르면 무조건 false
3 === '3'         // false (number vs string)
true === 1        // false (boolean vs number)
null === undefined // false (null vs undefined)
```

#### 작동 방식

```
1. 타입 비교
   ↓
2. 타입이 다르면 → false
   ↓
3. 타입이 같으면 값 비교
   ↓
4. 값이 같으면 → true
```

#### 특수한 경우

```javascript
// NaN은 자기 자신과도 같지 않음
NaN === NaN       // false

// 객체는 참조 비교
{} === {}         // false (서로 다른 객체)
[] === []         // false (서로 다른 배열)

const obj1 = { name: 'John' };
const obj2 = obj1;
obj1 === obj2     // true (같은 참조)

// 0과 -0
0 === -0          // true
```

> 💡 **권장**: 대부분의 경우 `===`를 사용하는 것이 안전합니다!
{: .prompt-tip }

### == (느슨한 동등 비교)

**`==`**는 **타입 강제 변환** 후 값을 비교합니다.

#### 기본 원리

```javascript
// 타입이 달라도 형 변환 후 비교
11 == '11'        // true (string '11'을 number 11로 변환)
true == 1         // true (boolean true를 number 1로 변환)
false == 0        // true (boolean false를 number 0로 변환)

// 예상치 못한 결과들
0 == ''           // true
0 == false        // true
'' == false       // true
null == undefined // true
```

#### 형 변환 규칙

| 비교 | 변환 규칙 | 예시 |
|------|----------|------|
| **number vs string** | string → number | `1 == '1'` → `true` |
| **boolean vs 다른 타입** | boolean → number | `true == 1` → `true` |
| **null vs undefined** | 특별 규칙 | `null == undefined` → `true` |
| **object vs primitive** | object.valueOf() | `[1] == 1` → `true` |

#### 위험한 예제들

```javascript
// ⚠️ 혼란스러운 결과들
console.log(false == '0');        // true
console.log(false == undefined);  // false
console.log(false == null);       // false
console.log(null == undefined);   // true

console.log(' \t\r\n ' == 0);     // true (공백 문자열 → 0)
console.log([1] == 1);            // true
console.log([1, 2] == '1,2');     // true
```

#### 특별 규칙: null과 undefined

```javascript
// null과 undefined는 서로만 같음
null == undefined     // true
null == 0            // false
undefined == 0       // false

// ===로는 다름
null === undefined   // false
```

### 비교 표

| 비교식 | == | === |
|--------|:--:|:---:|
| `5 == 5` | ✅ | ✅ |
| `5 == '5'` | ✅ | ❌ |
| `0 == false` | ✅ | ❌ |
| `0 == ''` | ✅ | ❌ |
| `null == undefined` | ✅ | ❌ |
| `NaN == NaN` | ❌ | ❌ |
| `{} == {}` | ❌ | ❌ |

### 언제 == 를 사용할까?

```javascript
// ✅ null/undefined 체크 (유일하게 권장되는 경우)
if (value == null) {
  // value가 null 또는 undefined
}

// 위 코드는 아래와 동일
if (value === null || value === undefined) {
  // value가 null 또는 undefined
}

// ❌ 그 외의 경우는 === 사용 권장
```

> ⚠️ **Best Practice**: `null`/`undefined` 체크 외에는 항상 `===`를 사용하세요!
{: .prompt-danger }

## typeof 연산자

**`typeof`**는 피연산자의 **타입을 문자열로 반환**합니다.

### 기본 문법

```javascript
typeof operand
typeof(operand)  // 괄호는 선택사항
```

### 반환되는 타입

```javascript
// Primitive Types
typeof 'hello'        // 'string'
typeof 42             // 'number'
typeof 3.14           // 'number'
typeof true           // 'boolean'
typeof undefined      // 'undefined'
typeof Symbol('id')   // 'symbol'
typeof 9007199254740991n  // 'bigint'

// Object Types
typeof {}             // 'object'
typeof []             // 'object' ⚠️
typeof null           // 'object' ⚠️ (역사적 버그)
typeof function(){}   // 'function'
```

### 주의사항

#### 1. null은 'object'

```javascript
typeof null  // 'object' ⚠️

// ✅ null 체크는 명시적으로
if (value === null) {
  console.log('null입니다');
}

// ✅ 또는 truthiness 체크
if (!value && typeof value === 'object') {
  console.log('null입니다');
}
```

#### 2. 배열도 'object'

```javascript
typeof []  // 'object'

// ✅ 배열 체크는 Array.isArray()
Array.isArray([])        // true
Array.isArray({})        // false

// ✅ 또는 instanceof
[] instanceof Array      // true
{} instanceof Array      // false
```

#### 3. NaN은 'number'

```javascript
typeof NaN  // 'number' ⚠️

// ✅ NaN 체크는 Number.isNaN()
Number.isNaN(NaN)       // true
Number.isNaN(123)       // false

// ❌ isNaN()은 형 변환 수행 (사용 주의)
isNaN('hello')          // true (문자열 → NaN으로 변환)
Number.isNaN('hello')   // false (변환하지 않음)
```

### 실전 활용

#### 타입 검증

```javascript
function validateInput(value) {
  // 문자열 검증
  if (typeof value === 'string') {
    return value.trim();
  }

  // 숫자 검증 (NaN 제외)
  if (typeof value === 'number' && !Number.isNaN(value)) {
    return value;
  }

  // 객체 검증 (null 제외)
  if (typeof value === 'object' && value !== null) {
    return Object.assign({}, value);
  }

  throw new Error('Invalid input type');
}
```

#### 안전한 함수 호출

```javascript
function safeCall(func, ...args) {
  if (typeof func === 'function') {
    return func(...args);
  }
  console.warn('Not a function:', func);
  return null;
}

// 사용 예
safeCall(console.log, 'Hello');  // 'Hello' 출력
safeCall('not a function');       // Warning 출력
```

#### 타입별 처리

```javascript
function processValue(value) {
  switch (typeof value) {
    case 'string':
      return value.toUpperCase();
    case 'number':
      return value * 2;
    case 'boolean':
      return !value;
    case 'object':
      if (value === null) {
        return 'null';
      }
      if (Array.isArray(value)) {
        return value.length;
      }
      return Object.keys(value).length;
    case 'undefined':
      return 'No value';
    default:
      return value;
  }
}

console.log(processValue('hello'));    // 'HELLO'
console.log(processValue(10));         // 20
console.log(processValue(true));       // false
console.log(processValue([1, 2, 3]));  // 3
```

## instanceof 연산자

**`instanceof`**는 객체가 **특정 클래스의 인스턴스**인지 확인합니다.

### 기본 문법

```javascript
object instanceof Constructor
```

### 기본 예제

```javascript
// 클래스 인스턴스 확인
class User {}
const user = new User();

console.log(user instanceof User);     // true
console.log(user instanceof Object);   // true (프로토타입 체인)

// 내장 생성자
const arr = [1, 2, 3];
console.log(arr instanceof Array);     // true
console.log(arr instanceof Object);    // true

const date = new Date();
console.log(date instanceof Date);     // true
console.log(date instanceof Object);   // true
```

### 상속 관계 확인

```javascript
class Animal {}
class Dog extends Animal {}

const myDog = new Dog();

console.log(myDog instanceof Dog);      // true
console.log(myDog instanceof Animal);   // true
console.log(myDog instanceof Object);   // true
```

### 생성자 함수

```javascript
function Person(name) {
  this.name = name;
}

const john = new Person('John');

console.log(john instanceof Person);    // true
console.log(john instanceof Object);    // true
```

### 주의사항

#### 1. Primitive 값은 false

```javascript
console.log(5 instanceof Number);       // false
console.log('hello' instanceof String); // false
console.log(true instanceof Boolean);   // false

// Wrapper 객체는 true
console.log(new Number(5) instanceof Number);     // true
console.log(new String('hello') instanceof String); // true
```

#### 2. 다른 window/frame의 객체

```javascript
// iframe의 배열은 현재 window의 Array와 다름
const iframe = document.createElement('iframe');
document.body.appendChild(iframe);

const iframeArray = iframe.contentWindow.Array;
const arr = new iframeArray();

console.log(arr instanceof Array);       // false
console.log(Array.isArray(arr));         // true ✅
```

### typeof vs instanceof 비교

```javascript
const arr = [1, 2, 3];
const obj = { name: 'John' };
const func = function() {};

// typeof
console.log(typeof arr);    // 'object' (배열인지 알 수 없음)
console.log(typeof obj);    // 'object'
console.log(typeof func);   // 'function'

// instanceof
console.log(arr instanceof Array);     // true
console.log(obj instanceof Array);     // false
console.log(func instanceof Function); // true
```

| 구분 | typeof | instanceof |
|------|--------|-----------|
| **반환값** | 문자열 | boolean |
| **용도** | 기본 타입 확인 | 객체 타입/상속 확인 |
| **Primitive** | ✅ | ❌ |
| **객체 구분** | ❌ (모두 'object') | ✅ |

## 고급: 타입 체크 유틸리티

### 정확한 타입 확인

```javascript
function getType(value) {
  return Object.prototype.toString.call(value).slice(8, -1);
}

console.log(getType([]));           // 'Array'
console.log(getType({}));           // 'Object'
console.log(getType(null));         // 'Null'
console.log(getType(undefined));    // 'Undefined'
console.log(getType(new Date()));   // 'Date'
console.log(getType(/regex/));      // 'RegExp'
console.log(getType(function(){})); // 'Function'
```

### 타입 체크 함수 모음

```javascript
const Type = {
  isString: (val) => typeof val === 'string',
  isNumber: (val) => typeof val === 'number' && !Number.isNaN(val),
  isBoolean: (val) => typeof val === 'boolean',
  isFunction: (val) => typeof val === 'function',
  isArray: (val) => Array.isArray(val),
  isObject: (val) => typeof val === 'object' && val !== null && !Array.isArray(val),
  isNull: (val) => val === null,
  isUndefined: (val) => typeof val === 'undefined',
  isDate: (val) => val instanceof Date && !isNaN(val),
  isRegExp: (val) => val instanceof RegExp,
  isEmpty: (val) => {
    if (val === null || val === undefined) return true;
    if (typeof val === 'string' || Array.isArray(val)) return val.length === 0;
    if (typeof val === 'object') return Object.keys(val).length === 0;
    return false;
  }
};

// 사용 예
console.log(Type.isString('hello'));     // true
console.log(Type.isNumber(42));          // true
console.log(Type.isArray([1, 2, 3]));    // true
console.log(Type.isEmpty({}));           // true
console.log(Type.isEmpty([]));           // true
console.log(Type.isEmpty(''));           // true
```

## 실전 패턴

### 1. 안전한 속성 접근

```javascript
function safeAccess(obj, path) {
  if (typeof obj !== 'object' || obj === null) {
    return undefined;
  }

  const keys = path.split('.');
  let result = obj;

  for (const key of keys) {
    if (typeof result !== 'object' || result === null) {
      return undefined;
    }
    result = result[key];
  }

  return result;
}

const user = {
  name: 'John',
  address: {
    city: 'Seoul'
  }
};

console.log(safeAccess(user, 'address.city'));     // 'Seoul'
console.log(safeAccess(user, 'address.country'));  // undefined
console.log(safeAccess(null, 'name'));             // undefined
```

### 2. 타입 변환 체크

```javascript
function toNumber(value) {
  // 이미 숫자면 그대로 반환
  if (typeof value === 'number') {
    return value;
  }

  // 문자열 → 숫자 변환
  if (typeof value === 'string') {
    const num = Number(value);
    if (!Number.isNaN(num)) {
      return num;
    }
  }

  // boolean → 숫자
  if (typeof value === 'boolean') {
    return value ? 1 : 0;
  }

  // 변환 실패
  return NaN;
}

console.log(toNumber(42));       // 42
console.log(toNumber('42'));     // 42
console.log(toNumber(true));     // 1
console.log(toNumber('hello'));  // NaN
```

### 3. 깊은 동등 비교

```javascript
function deepEqual(a, b) {
  // === 체크 (primitive, 같은 참조)
  if (a === b) return true;

  // null 체크
  if (a === null || b === null) return false;

  // 타입 체크
  if (typeof a !== typeof b) return false;

  // 배열 체크
  if (Array.isArray(a) && Array.isArray(b)) {
    if (a.length !== b.length) return false;
    return a.every((val, i) => deepEqual(val, b[i]));
  }

  // 객체 체크
  if (typeof a === 'object') {
    const keysA = Object.keys(a);
    const keysB = Object.keys(b);

    if (keysA.length !== keysB.length) return false;

    return keysA.every(key =>
      keysB.includes(key) && deepEqual(a[key], b[key])
    );
  }

  return false;
}

console.log(deepEqual({ a: 1 }, { a: 1 }));           // true
console.log(deepEqual([1, 2, 3], [1, 2, 3]));         // true
console.log(deepEqual({ a: { b: 1 } }, { a: { b: 1 } })); // true
```

## Falsy와 Truthy 값

JavaScript에서 `if` 문 등에서 boolean으로 변환되는 값들입니다.

### Falsy 값 (false로 변환)

```javascript
// 8가지 Falsy 값
false
0
-0
0n (BigInt zero)
'' (빈 문자열)
null
undefined
NaN

// 모두 false로 평가됨
if (0) {}              // 실행 안 됨
if ('') {}             // 실행 안 됨
if (null) {}           // 실행 안 됨
if (undefined) {}      // 실행 안 됨
if (NaN) {}            // 실행 안 됨
```

### Truthy 값 (true로 변환)

```javascript
// Falsy가 아닌 모든 값
true
1
-1
'0' (문자열 '0')
'false' (문자열 'false')
[] (빈 배열)
{} (빈 객체)
function(){}

// 모두 true로 평가됨
if (1) {}              // 실행됨
if ('hello') {}        // 실행됨
if ([]) {}             // 실행됨 ⚠️
if ({}) {}             // 실행됨 ⚠️
```

### 주의할 점

```javascript
// ⚠️ 빈 배열/객체는 truthy
if ([]) console.log('빈 배열은 truthy');   // 출력됨
if ({}) console.log('빈 객체는 truthy');   // 출력됨

// ✅ 배열 길이 체크
if (arr.length > 0) {}

// ✅ 객체 키 개수 체크
if (Object.keys(obj).length > 0) {}
```

## 핵심 정리

### 동등 연산자 선택 가이드

```
값을 비교할 때
├─ null/undefined 체크? → value == null
└─ 그 외 모든 경우 → value === other
```

### 타입 체크 선택 가이드

```
타입을 확인할 때
├─ Primitive 타입? → typeof
├─ 배열? → Array.isArray()
├─ null? → value === null
├─ NaN? → Number.isNaN()
├─ 클래스 인스턴스? → instanceof
└─ 정확한 타입? → Object.prototype.toString.call()
```

### Best Practices

1. **항상 `===` 사용** (null 체크 제외)
2. **`typeof`로 primitive 체크**
3. **`Array.isArray()`로 배열 체크**
4. **`instanceof`로 클래스 인스턴스 체크**
5. **Falsy 값 활용 시 주의** (빈 배열/객체는 truthy)

### 일반적인 실수

```javascript
// ❌ 잘못된 비교
if (value == true) {}        // 대신 if (value) {} 사용
if (arr == []) {}            // 항상 false
if (typeof value == 'object' && !value) {} // null 체크 실패

// ✅ 올바른 비교
if (value) {}                // truthy 체크
if (arr.length === 0) {}     // 배열 길이 체크
if (value === null) {}       // null 체크
```

> JavaScript의 타입 시스템을 완벽히 이해하면 예상치 못한 버그를 크게 줄일 수 있습니다!
{: .prompt-info }

## 참고 자료

- [MDN - Equality comparisons](https://developer.mozilla.org/ko/docs/Web/JavaScript/Equality_comparisons_and_sameness)
- [MDN - typeof](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/typeof)
- [MDN - instanceof](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/instanceof)
- [JavaScript Equality Table](https://dorey.github.io/JavaScript-Equality-Table/)
