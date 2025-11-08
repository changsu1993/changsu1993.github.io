---
title: JavaScript Primitive Type과 Reference Type 완벽 가이드
description: 원시 타입과 참조 타입의 근본적인 차이부터 Wrapper Object, Auto-Boxing, 생성자 함수까지. JavaScript 타입 시스템의 핵심 개념을 깊이 있게 이해합니다.
author: changsu
date: 2020-09-06 02:25:34 +0900
categories: [Programming, JavaScript]
tags: [javascript, primitive-type, reference-type, wrapper-object, auto-boxing, constructor-function, first-class-function, 원시타입, 참조타입]
---

JavaScript의 타입 시스템을 제대로 이해하기 위해서는 Primitive Type과 Reference Type의 차이를 명확히 알아야 합니다.

## 타입의 분류

JavaScript는 두 가지 범주의 자료형을 제공합니다:

```
JavaScript 타입
├── Primitive Type (원시 타입)
│   ├── Boolean
│   ├── String
│   ├── Number
│   ├── Undefined
│   ├── Null
│   └── Symbol
│
└── Reference Type (참조 타입)
    ├── Object
    ├── Array
    ├── Function
    └── Date, RegExp 등
```

## Primitive Type (원시 타입)

### 정의

**원시 타입**은 객체가 아닌 데이터로, **값 자체**로 저장됩니다.

```javascript
let num = 42;        // 42라는 값이 메모리에 직접 저장
let str = "hello";   // "hello"라는 값이 메모리에 직접 저장
```

### 핵심 특징

#### 1. 불변성 (Immutability)

원시 타입의 값은 **절대 변경할 수 없습니다**.

```javascript
let str = "hello";
str.toUpperCase();   // "HELLO" 반환

console.log(str);    // "hello" (원본은 변경되지 않음)
```

변수에 새 값을 할당하면, 기존 값이 변경되는 것이 아니라 **새로운 값으로 대체**됩니다:

```javascript
let x = 10;
x = 20;  // 10이 20으로 변하는 게 아니라, x가 새로운 값 20을 참조
```

#### 2. 값에 의한 할당 (Pass by Value)

원시 타입은 할당 시 **값이 복사**됩니다:

```javascript
let a = 10;
let b = a;    // a의 값(10)이 b에 복사됨

b = 20;

console.log(a);  // 10 (변경되지 않음)
console.log(b);  // 20
```

#### 3. 메모리 저장 방식

```
메모리 주소    값
0x001         10      ← a가 참조
0x002         10      ← b가 참조 (별도 복사본)
```

## Primitive Type의 종류

### 1. Boolean

논리적인 요소를 나타내며, `true`와 `false` 두 가지 값만 가능합니다.

```javascript
let isActive = true;
let isComplete = false;

// Truthy & Falsy
Boolean(1);          // true
Boolean(0);          // false
Boolean("");         // false
Boolean("hello");    // true
```

### 2. String

텍스트 데이터를 표현합니다. **16비트 부호 없는 정수 값 요소들의 집합**입니다.

```javascript
let name = "Alice";
let greeting = 'Hello';
let template = `Hi, ${name}`;

// 각 문자는 인덱스로 접근 가능
name[0];  // "A"
```

### 3. Number

**64비트 부동소수점 형식**을 사용합니다. JavaScript에는 정수만을 위한 별도 타입이 없습니다.

```javascript
let integer = 42;
let float = 3.14;
let negative = -100;

// 특수 값
let infinity = Infinity;
let negInfinity = -Infinity;
let notANumber = NaN;

console.log(1 / 0);         // Infinity
console.log("text" * 2);    // NaN
```

### 4. Undefined

변수가 선언되었지만 **값이 할당되지 않은** 상태입니다.

```javascript
let x;
console.log(x);  // undefined

function test(param) {
  console.log(param);  // 인자가 없으면 undefined
}
test();  // undefined
```

### 5. Null

**의도적으로 값이 비어있음**을 표현합니다.

```javascript
let user = null;  // 명시적으로 "값 없음"을 할당

// typeof의 유명한 버그
typeof null;  // "object" (역사적 이유로 인한 버그)
```

**Undefined vs Null**:

```javascript
let a;           // undefined - 값이 할당되지 않음
let b = null;    // null - 의도적으로 빈 값 할당

a == b;   // true (값만 비교)
a === b;  // false (타입까지 비교)
```

### 6. Symbol (ES6)

**유일하고 변경 불가능한** 값입니다. 주로 객체의 고유한 프로퍼티 키로 사용됩니다.

```javascript
const sym1 = Symbol("description");
const sym2 = Symbol("description");

console.log(sym1 === sym2);  // false (각각 유일함)

// 객체 프로퍼티 키로 사용
const obj = {
  [sym1]: "value1",
  [sym2]: "value2"
};
```

## Reference Type (참조 타입)

### 정의

**Reference Type**은 객체 형식의 타입으로, 메모리에 **주소(참조)**를 저장합니다.

```javascript
const obj = { name: "Alice" };  // 객체의 메모리 주소가 저장됨
const arr = [1, 2, 3];          // 배열의 메모리 주소가 저장됨
```

### 핵심 특징

#### 참조에 의한 할당 (Pass by Reference)

```javascript
const obj1 = { value: 10 };
const obj2 = obj1;    // obj1의 참조(주소)가 복사됨

obj2.value = 20;

console.log(obj1.value);  // 20 (같은 객체를 참조)
console.log(obj2.value);  // 20
```

```
메모리 주소    값
0x100         { value: 20 }
                ↑
           obj1, obj2 모두 참조
```

## 함수 (Function)

함수는 **특별한 프로퍼티를 가진 객체**입니다.

### 함수의 프로퍼티

```javascript
const myFunc = function (param1, param2) {
  return param1 + param2;
};

console.log(myFunc.name);    // "myFunc"
console.log(myFunc.length);  // 2 (매개변수 개수)
```

### 함수에 프로퍼티 추가

일반 객체처럼 프로퍼티를 추가할 수 있습니다:

```javascript
myFunc.customProperty = "custom value";
myFunc.counter = 0;

console.log(myFunc.customProperty);  // "custom value"
```

### 1급 객체 (First-Class Object)

JavaScript의 함수는 다음 조건을 만족하여 **1급 객체**입니다:

```javascript
// 1. 변수에 할당 가능
const func = function() { return "hi"; };

// 2. 다른 함수의 인자로 전달 가능
function execute(fn) {
  return fn();
}
execute(func);  // "hi"

// 3. 다른 함수의 반환값으로 사용 가능
function createMultiplier(factor) {
  return function(num) {
    return num * factor;
  };
}

const double = createMultiplier(2);
console.log(double(5));  // 10
```

이는 JavaScript 객체들이 갖는 특성과 동일합니다.

## 메서드 (Method)

메서드는 **객체의 프로퍼티로 할당된 함수**입니다.

```javascript
const calculator = {
  value: 0,

  // 메서드
  add: function(num) {
    this.value += num;
    return this;
  },

  // ES6 단축 문법
  subtract(num) {
    this.value -= num;
    return this;
  }
};

calculator.add(10).subtract(3);
console.log(calculator.value);  // 7
```

## 생성자 함수 (Constructor Function)

### 정의

**생성자 함수**는 `new` 키워드와 함께 호출되어 새로운 객체를 생성하는 함수입니다.

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;

  this.greet = function() {
    console.log(`Hi, I'm ${this.name}`);
  };
}

const alice = new Person("Alice", 25);
const bob = new Person("Bob", 30);

alice.greet();  // "Hi, I'm Alice"

console.log(alice instanceof Person);  // true
console.log(alice instanceof Object);  // true
```

### new 키워드의 역할

`new`를 사용하면 다음 과정이 자동으로 수행됩니다:

```javascript
// new Person("Alice", 25)는 내부적으로:

// 1. 빈 객체 생성
const newObj = {};

// 2. 새 객체의 __proto__를 생성자 함수의 prototype에 연결
newObj.__proto__ = Person.prototype;

// 3. 생성자 함수를 새 객체를 this로 바인딩하여 실행
Person.call(newObj, "Alice", 25);

// 4. 객체 반환
return newObj;
```

### new 없이 호출하면?

`new` 없이 생성자 함수를 호출하면 일반 함수처럼 동작합니다:

```javascript
function Person(name) {
  this.name = name;
}

// new 없이 호출
const result = Person("Alice");

console.log(result);      // undefined
console.log(window.name); // "Alice" (전역 객체에 할당됨!)
```

이는 **의도하지 않은 전역 변수 오염**을 일으킬 수 있으므로 주의해야 합니다.

**해결책**: 생성자 함수 이름은 대문자로 시작하는 관례를 따르고, ES6 클래스 문법을 사용합니다.

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

// new 없이 호출하면 에러 발생
const person = Person("Alice");  // TypeError!
```

## Wrapper Object (래퍼 객체)

### 정의

원시 타입에 대한 **객체 형태의 래퍼**입니다.

```javascript
// 원시 타입
const primitiveStr = "hello";
typeof primitiveStr;  // "string"

// Wrapper Object
const objectStr = new String("hello");
typeof objectStr;     // "object"

primitiveStr === objectStr;  // false
```

### Wrapper Object 생성

`new` 키워드를 사용하여 래퍼 객체를 생성합니다:

```javascript
const str = new String("blue");
console.log(typeof str);  // "object"

// 내부 구조
console.log(str);
/*
String {
  0: "b",
  1: "l",
  2: "u",
  3: "e",
  length: 4
}
*/
```

### 생성자 함수 vs new 생성자 함수

```javascript
// 생성자 함수만 사용 - 원시 타입 반환
String(1234);        // "1234" (string)
String(true);        // "true" (string)
String(null);        // "null" (string)
String(undefined);   // "undefined" (string)

typeof String(123);  // "string"

// new 생성자 함수 - 객체 반환
const num = new Number(123);
typeof num;          // "object"
```

## Auto-Boxing (오토 박싱)

### 정의

원시 타입에서 **자동으로 래퍼 객체로 변환**되는 과정입니다.

### 작동 원리

원시 타입에서 프로퍼티나 메서드를 호출할 때, JavaScript는:

```javascript
const str = "hello";

// 1. str.length 접근 시도
str.length;

// 2. JavaScript 내부 과정:
// (a) 임시 래퍼 객체 생성
const temp = new String("hello");

// (b) 래퍼 객체의 length 접근
const result = temp.length;  // 5

// (c) 임시 객체 제거
// temp는 가비지 컬렉션 대상이 됨

// 3. 원본은 그대로 유지
console.log(str);  // "hello" (변경 없음)
```

### 실제 예제

```javascript
const name = "Alice";

// Auto-Boxing 발생
console.log(name.length);           // 5
console.log(name.toUpperCase());    // "ALICE"
console.log(name.charAt(0));        // "A"

// 원본은 여전히 원시 타입
console.log(typeof name);           // "string"
console.log(name === "Alice");      // true
```

### 프로퍼티 할당 시도

Auto-Boxing 때문에 원시 타입에 프로퍼티를 할당해도 **에러가 발생하지 않습니다**:

```javascript
const num = 42;

num.customProp = "test";  // 임시 래퍼 객체에 할당

console.log(num.customProp);  // undefined (이미 임시 객체는 제거됨)

// 과정:
// 1. 할당 시: new Number(42).customProp = "test" (임시 생성)
// 2. 할당 완료 후: 임시 객체 제거
// 3. 접근 시: new Number(42).customProp (새로운 임시 객체, 프로퍼티 없음)
```

### Wrapper Object가 없는 타입

`undefined`와 `null`은 래퍼 객체가 없어서 **에러가 발생**합니다:

```javascript
const value = null;
value.prop = "test";
// TypeError: Cannot set property 'prop' of null

const undef = undefined;
undef.prop = "test";
// TypeError: Cannot set property 'prop' of undefined
```

## 원시 타입 vs 참조 타입 비교

### 비교 연산

```javascript
// 원시 타입 - 값 비교
let a = "hello";
let b = "hello";
console.log(a === b);  // true (값이 같음)

// 참조 타입 - 참조 비교
let obj1 = { name: "Alice" };
let obj2 = { name: "Alice" };
console.log(obj1 === obj2);  // false (다른 객체)

let obj3 = obj1;
console.log(obj1 === obj3);  // true (같은 객체 참조)
```

### 함수 인자 전달

```javascript
// 원시 타입 - 값 복사
function changePrimitive(val) {
  val = 100;
}

let num = 10;
changePrimitive(num);
console.log(num);  // 10 (변경 안 됨)

// 참조 타입 - 참조 전달
function changeObject(obj) {
  obj.value = 100;
}

let myObj = { value: 10 };
changeObject(myObj);
console.log(myObj.value);  // 100 (변경됨!)
```

## 타입 확인

### typeof 연산자

```javascript
typeof 42;              // "number"
typeof "hello";         // "string"
typeof true;            // "boolean"
typeof undefined;       // "undefined"
typeof Symbol();        // "symbol"

typeof {};              // "object"
typeof [];              // "object" (배열도 객체)
typeof function(){};    // "function"

typeof null;            // "object" (역사적 버그!)
```

### instanceof 연산자

```javascript
const arr = [1, 2, 3];
const obj = { name: "Alice" };
const func = function() {};

arr instanceof Array;     // true
arr instanceof Object;    // true (배열은 객체이기도 함)

obj instanceof Object;    // true
func instanceof Function; // true
func instanceof Object;   // true (함수도 객체)

// 원시 타입에는 사용 불가
"hello" instanceof String;  // false
new String("hello") instanceof String;  // true
```

## 실전 사용 팁

### 1. Wrapper Object를 직접 생성하지 마세요

```javascript
// ❌ 나쁜 예
const str = new String("hello");
const num = new Number(42);

// ✅ 좋은 예
const str = "hello";
const num = 42;

// 타입 변환이 필요하면 new 없이 사용
const str2 = String(123);   // "123"
const num2 = Number("456"); // 456
```

### 2. const로 객체를 선언해도 내부는 변경 가능

```javascript
const obj = { count: 0 };

// ✅ 프로퍼티 변경 가능
obj.count = 10;

// ❌ 재할당 불가
obj = {};  // TypeError!
```

### 3. 깊은 복사 주의

```javascript
// 얕은 복사 - 참조만 복사됨
const original = { nested: { value: 10 } };
const copy = original;

copy.nested.value = 20;
console.log(original.nested.value);  // 20 (함께 변경됨)

// 깊은 복사 방법들
const deepCopy1 = JSON.parse(JSON.stringify(original));
const deepCopy2 = structuredClone(original);  // 최신 방법
```

### 4. 생성자 함수보다 클래스 사용

```javascript
// ✅ 현대적 접근
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hi, I'm ${this.name}`;
  }
}

const user = new User("Alice");
```

## 마치며

Primitive Type과 Reference Type의 차이를 이해하는 것은 JavaScript를 깊이 있게 다루는 첫걸음입니다.

### 핵심 요약

```markdown
✅ Primitive Type
- 값 자체로 저장
- 불변(Immutable)
- 값에 의한 전달
- 7가지: Boolean, String, Number, Undefined, Null, Symbol, BigInt

✅ Reference Type
- 참조(주소)로 저장
- 가변(Mutable)
- 참조에 의한 전달
- Object, Array, Function 등

✅ Auto-Boxing
- 원시 타입에서 메서드 사용 시 자동으로 래퍼 객체로 변환
- 사용 후 즉시 제거
- 원본에 영향 없음
```

이러한 개념을 정확히 이해하면 변수 할당, 함수 인자 전달, 객체 비교 등에서 발생할 수 있는 버그를 예방할 수 있습니다.

## 참고 자료

- [MDN - JavaScript 데이터 타입](https://developer.mozilla.org/ko/docs/Web/JavaScript/Data_structures)
- [MDN - 원시 값](https://developer.mozilla.org/ko/docs/Glossary/Primitive)
- [MDN - Object](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [JavaScript의 타입과 자료구조 (MDN)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Data_structures)
- [33 Concepts Every JavaScript Developer Should Know](https://github.com/leonardomso/33-js-concepts)
- [You Don't Know JS: Types & Grammar](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/types%20%26%20grammar/README.md)
