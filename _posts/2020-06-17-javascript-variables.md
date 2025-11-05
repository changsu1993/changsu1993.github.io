---
title: JavaScript Variables 완벽 가이드 - var, let, const 차이와 활용법
description: JavaScript 변수 선언 방법(var, let, const)의 차이점과 스코프, 호이스팅, TDZ 개념을 이해하고, 올바른 변수 명명 규칙과 사용 패턴을 학습합니다.
author: changsu
date: 2020-06-17 00:38:03 +0900
categories: [JavaScript, Basics]
tags: [javascript, variables, var, let, const, scope, hoisting, tdz, naming-convention]
---

## 1. 변수란?

변수(Variable)는 데이터를 저장하는 컨테이너입니다. 사람이 이름, 나이, 직업 등의 정보를 기억하듯이, 컴퓨터도 변수를 사용해 데이터를 기억합니다.

```javascript
// 사람의 정보를 변수에 저장
let name = "김개발";
let job = "Frontend Developer";
let age = 28;
```

### 변수의 구성 요소

- **변수명(Variable Name)**: 데이터를 식별하는 이름 (예: `name`, `job`)
- **값(Value)**: 변수에 저장된 실제 데이터 (예: `"김개발"`, `"Frontend Developer"`)
- **할당 연산자(=)**: 값을 변수에 저장하는 연산자

## 2. 변수 선언 방법: var, let, const

JavaScript는 세 가지 키워드로 변수를 선언할 수 있습니다.

### 2.1 기본 문법

```javascript
var name = "홍길동";      // ES5 방식
let age = 30;            // ES6+ 방식 (재할당 가능)
const PI = 3.14159;      // ES6+ 방식 (재할당 불가)
```

### 2.2 var vs let vs const 비교표

| 특징 | var | let | const |
|------|-----|-----|-------|
| **재할당** | 가능 | 가능 | 불가능 |
| **재선언** | 가능 | 불가능 | 불가능 |
| **스코프** | 함수 스코프 | 블록 스코프 | 블록 스코프 |
| **호이스팅** | undefined로 초기화 | TDZ (초기화 전 접근 불가) | TDZ (초기화 전 접근 불가) |
| **권장 사용** | ❌ 비권장 | ✅ 값 변경 필요 시 | ✅ 값 고정 시 |

## 3. var의 문제점

### 3.1 재선언 가능 (문제 발생 가능)

```javascript
var name = "김개발";
var name = "박코딩";  // 에러 없이 재선언 가능 (버그 유발 가능)
console.log(name);    // "박코딩"
```

### 3.2 함수 스코프 (예상치 못한 동작)

```javascript
if (true) {
  var x = 10;
}
console.log(x);  // 10 (블록 밖에서도 접근 가능)

for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 예상: 0, 1, 2
// 실제: 3, 3, 3 (var는 함수 스코프이므로 같은 i 참조)
```

### 3.3 호이스팅으로 인한 혼란

```javascript
console.log(message);  // undefined (에러 아님)
var message = "Hello";
```

## 4. let과 const (ES6+)

### 4.1 let - 재할당 가능한 변수

```javascript
let count = 0;
count = 1;      // ✅ 재할당 가능
count = 2;      // ✅ 가능

let count = 3;  // ❌ SyntaxError: Identifier 'count' has already been declared
```

#### let 사용 예시

```javascript
// 카운터
let score = 0;
score += 10;
score += 5;
console.log(score);  // 15

// 반복문
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 출력: 0, 1, 2 (각 반복마다 새로운 i 생성)
```

### 4.2 const - 재할당 불가능한 상수

```javascript
const API_KEY = "abc123xyz";
API_KEY = "new_key";  // ❌ TypeError: Assignment to constant variable

const user = { name: "김개발" };
user = {};           // ❌ 재할당 불가
user.name = "박코딩"; // ✅ 객체 속성 변경은 가능
```

> **Tip**: `const`는 재할당을 막지만, 객체/배열의 내부 값은 변경 가능합니다.
{: .prompt-tip }

#### const와 객체/배열

```javascript
// const 배열
const numbers = [1, 2, 3];
numbers.push(4);        // ✅ 가능
numbers[0] = 10;        // ✅ 가능
numbers = [4, 5, 6];    // ❌ TypeError

// const 객체
const person = { name: "김개발", age: 28 };
person.age = 29;        // ✅ 가능
person.job = "Developer"; // ✅ 가능
person = {};            // ❌ TypeError
```

#### 객체를 완전히 고정하려면

```javascript
const config = Object.freeze({
  API_URL: "https://api.example.com",
  TIMEOUT: 5000
});

config.API_URL = "new_url";  // ❌ 무시됨 (strict mode에서는 에러)
```

## 5. 블록 스코프 vs 함수 스코프

### 5.1 블록 스코프 (let, const)

```javascript
{
  let blockVar = "블록 안";
  const blockConst = "블록 안";
  console.log(blockVar);     // "블록 안"
}
console.log(blockVar);       // ❌ ReferenceError

if (true) {
  let message = "Hello";
}
console.log(message);        // ❌ ReferenceError
```

### 5.2 함수 스코프 (var)

```javascript
function example() {
  var funcVar = "함수 안";
  if (true) {
    var funcVar2 = "블록 안이지만 함수 스코프";
  }
  console.log(funcVar2);  // "블록 안이지만 함수 스코프" (접근 가능)
}
console.log(funcVar);     // ❌ ReferenceError (함수 밖에서는 접근 불가)
```

## 6. 호이스팅(Hoisting)

### 6.1 var 호이스팅

```javascript
console.log(a);  // undefined
var a = 10;

// 실제로는 이렇게 동작
var a;           // 선언이 끌어올려짐
console.log(a);  // undefined
a = 10;          // 할당은 원래 위치
```

### 6.2 let/const와 TDZ (Temporal Dead Zone)

```javascript
console.log(b);  // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 20;

// TDZ: 스코프 시작부터 초기화 전까지의 구간
{
  // TDZ 시작
  console.log(x);  // ❌ ReferenceError
  let x = 10;      // TDZ 종료
  console.log(x);  // 10
}
```

> **Warning**: let과 const도 호이스팅되지만, TDZ로 인해 선언 전 접근 시 에러가 발생합니다.
{: .prompt-warning }

## 7. 변수 선언과 초기화

### 7.1 선언(Declaration)과 할당(Assignment)

```javascript
// 선언만
let name;
console.log(name);  // undefined

// 나중에 할당
name = "김개발";
console.log(name);  // "김개발"

// 선언과 동시에 할당
let age = 28;

// const는 선언과 동시에 초기화 필수
const PI;  // ❌ SyntaxError: Missing initializer in const declaration
const E = 2.71828;  // ✅
```

### 7.2 다중 변수 선언

```javascript
// 한 줄에 여러 변수 선언
let x = 1, y = 2, z = 3;

// 가독성 좋은 방식
let firstName = "길동",
    lastName = "홍",
    age = 30;
```

## 8. 변수 명명 규칙(Naming Convention)

### 8.1 필수 규칙 (지키지 않으면 에러)

```javascript
// ✅ 올바른 변수명
let name;
let _privateVar;
let $jquery;
let user2;
let userName;

// ❌ 잘못된 변수명
let 2user;        // 숫자로 시작 불가
let user-name;    // 하이픈 사용 불가
let user name;    // 공백 사용 불가
let class;        // 예약어 사용 불가
```

### 8.2 명명 컨벤션 (관례)

#### camelCase (일반 변수, 함수)

```javascript
let firstName = "길동";
let userAge = 30;
let isLoggedIn = true;

function getUserInfo() { }
function calculateTotalPrice() { }
```

#### PascalCase (클래스, 생성자)

```javascript
class UserAccount { }
class ShoppingCart { }

function Person(name) {
  this.name = name;
}
```

#### UPPER_SNAKE_CASE (상수)

```javascript
const MAX_SIZE = 100;
const API_BASE_URL = "https://api.example.com";
const DEFAULT_TIMEOUT = 5000;
```

### 8.3 의미 있는 변수명

```javascript
// ❌ 나쁜 예
let a = 30;
let x = "김개발";
let temp = calculatePrice();

// ✅ 좋은 예
let userAge = 30;
let userName = "김개발";
let totalPrice = calculatePrice();
```

## 9. 변수 사용 Best Practices

### 9.1 기본 원칙

```javascript
// 1. 기본적으로 const 사용
const API_KEY = "abc123";
const config = { theme: "dark" };

// 2. 재할당이 필요할 때만 let 사용
let counter = 0;
counter++;

// 3. var는 사용하지 않기
// var oldStyle = "사용 금지";  // ❌
```

### 9.2 스코프 최소화

```javascript
// ❌ 나쁜 예: 전역 스코프 오염
let result;
function calculate() {
  result = 1 + 2;
}
calculate();
console.log(result);

// ✅ 좋은 예: 필요한 곳에서만 선언
function calculate() {
  const result = 1 + 2;
  return result;
}
const answer = calculate();
console.log(answer);
```

### 9.3 조기 반환 패턴

```javascript
// ❌ 나쁜 예
function processUser(user) {
  let result;
  if (user) {
    if (user.active) {
      result = user.name;
    }
  }
  return result;
}

// ✅ 좋은 예
function processUser(user) {
  if (!user) return null;
  if (!user.active) return null;
  return user.name;
}
```

## 10. 실전 예제

### 10.1 카운터 구현

```javascript
let count = 0;

function increment() {
  count++;
  console.log(`현재 카운트: ${count}`);
}

function decrement() {
  count--;
  console.log(`현재 카운트: ${count}`);
}

function reset() {
  count = 0;
  console.log("카운트 초기화");
}

increment();  // 현재 카운트: 1
increment();  // 현재 카운트: 2
decrement();  // 현재 카운트: 1
reset();      // 카운트 초기화
```

### 10.2 설정 관리

```javascript
// 변경 불가능한 설정
const APP_CONFIG = Object.freeze({
  NAME: "MyApp",
  VERSION: "1.0.0",
  API_URL: "https://api.example.com",
  TIMEOUT: 5000
});

// 변경 가능한 사용자 설정
let userSettings = {
  theme: "light",
  language: "ko",
  notifications: true
};

function updateTheme(newTheme) {
  userSettings.theme = newTheme;
  console.log(`테마 변경: ${newTheme}`);
}

updateTheme("dark");  // 테마 변경: dark
```

### 10.3 유효성 검사

```javascript
function validateUser(userData) {
  const { name, age, email } = userData;

  // 각 검사 결과를 변수에 저장
  const hasName = name && name.length > 0;
  const hasValidAge = age && age >= 0 && age <= 150;
  const hasValidEmail = email && email.includes("@");

  if (!hasName) {
    return { valid: false, error: "이름이 필요합니다" };
  }

  if (!hasValidAge) {
    return { valid: false, error: "유효하지 않은 나이입니다" };
  }

  if (!hasValidEmail) {
    return { valid: false, error: "유효하지 않은 이메일입니다" };
  }

  return { valid: true };
}

// 사용
const result = validateUser({
  name: "김개발",
  age: 28,
  email: "dev@example.com"
});

console.log(result);  // { valid: true }
```

## 정리

### var vs let vs const 선택 가이드

```javascript
// 1️⃣ 기본: const 사용
const userName = "김개발";
const colors = ["red", "blue"];

// 2️⃣ 재할당 필요: let 사용
let counter = 0;
let message = "초기값";

// 3️⃣ var: 절대 사용하지 않기
// var oldStyle;  // ❌
```

### 핵심 원칙

1. **const를 기본으로 사용**하고, 재할당이 필요할 때만 let 사용
2. **var는 사용하지 않기** (레거시 코드 제외)
3. **의미 있는 변수명** 사용 (a, b, x보다 userName, totalPrice)
4. **스코프를 최소화**하여 변수 선언
5. **camelCase 명명 규칙** 따르기

> **참고**: 모던 JavaScript 개발에서는 `const`와 `let`만 사용하는 것이 표준입니다.
{: .prompt-info }

## 참고 자료

- [MDN - var](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/var)
- [MDN - let](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/let)
- [MDN - const](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/const)
- [JavaScript.info - Variables](https://javascript.info/variables)
