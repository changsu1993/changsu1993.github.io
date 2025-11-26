---
title: JavaScript Expression vs Statement 완벽 가이드 - 표현식과 문장의 차이
description: JavaScript의 핵심 개념인 Expression과 Statement의 차이를 이해합니다. 함수 선언 vs 표현식, IIFE, 블록 문장과 객체 리터럴까지 상세히 알아봅니다.
author: changsu
date: 2020-09-16 18:36:32 +0900
categories: [Programming, JavaScript]
tags: [javascript, expression, statement, function]
---

JavaScript 코드는 크게 두 가지 문법적 카테고리로 나뉩니다: **Expression(표현식)**과 **Statement(문장)**. 이 둘의 차이를 이해하는 것은 JavaScript를 깊이 있게 이해하는 핵심입니다.

## 핵심 차이점

> 표현식(Expression)은 문장(Statement)처럼 동작할 수 있지만, 문장(Statement)은 표현식(Expression)처럼 동작할 수 없다.

```javascript
// Expression은 Statement가 될 수 있음
1 + 1;  // 표현식 문장 (Expression Statement)

// Statement는 Expression이 될 수 없음
if (true) {}  // 값으로 사용 불가
```

## Expression (표현식)

### 정의

**표현식은 값을 산출해내는 코드**를 의미합니다. 값이 도출되기 때문에 **함수의 인자로 전달**할 수 있습니다.

```javascript
// 모두 표현식
1;
5 + 5;
'Hello world';
myFunc('a', 'b');
myVariable;
true && false;
null;
```

### 특징: 값이 필요한 곳에 사용 가능

표현식은 JavaScript 코드 중 **값이 들어가는 곳이면 어디든** 넣을 수 있습니다.

```javascript
console.log(true && 1 * 5);  // 5

// 함수 인자로 사용
myFunc(1 + 2);               // 3을 전달

// 삼항 연산자에서 사용
const result = true ? 10 + 5 : 20 - 5;

// 배열 요소로 사용
const arr = [1 + 1, 2 * 3, Math.random()];
```

### 표현식은 상태를 바꿀 필요가 없음

```javascript
const a = 1;  // 이건 Statement (변수 선언문)

// 아래는 모두 Expression (a의 값을 변경하지 않음)
a * 2;   // 2
a + 1;   // 2
a - 1;   // 0

console.log(a);  // 1 (변경되지 않음)
```

### 함수 호출도 표현식

함수 호출은 표현식이지만, 함수 내부에서 상태를 변경하는 문장을 포함할 수 있습니다.

```javascript
let externalValue = 10;

// 함수 호출은 표현식
const result = changeValue();  // 표현식

function changeValue() {
  externalValue = 20;  // 이건 Statement (상태 변경)
  return 30;           // return도 Statement
}

console.log(result);         // 30 (반환값)
console.log(externalValue);  // 20 (부수 효과)
```

**더 나은 접근 - 순수 함수**:

```javascript
// ✅ 명시적이고 예측 가능한 코드
const getValue = () => {
  return 10;  // 명시적 반환
};

const result = getValue();
```

또는 매개변수로 전달:

```javascript
const calculate = (n) => {
  return n * 2;  // 명시적 반환
};

const result = calculate(10);  // 20
```

## Statement (문장)

### 정의

**문장은 무언가를 수행**합니다. Statement는 **값이 들어와야 할 곳에 사용할 수 없습니다**.

```javascript
// ❌ Statement는 값으로 사용 불가
const x = if (true) { 1 };  // Syntax Error
console.log(for (let i = 0; i < 10; i++) {});  // Syntax Error
```

### JavaScript의 문장 종류

```javascript
// 조건문
if (condition) { }
if (condition) { } else { }

// 반복문
while (condition) { }
do { } while (condition);
for (let i = 0; i < 10; i++) { }
for (const key in object) { }

// 제어문
switch (value) { }
break;
continue;

// 변수 선언
let a;
const b = 1;
var c;

// 기타
debugger;
with (object) { }  // deprecated (사용 권장하지 않음)
```

## 함수 선언 vs 함수 표현식

### 1. 함수 선언 (Function Declaration) - Statement

```javascript
function greet(name) {
  return `Hello, ${name}`;
}
```

**특징**:
- 호이스팅됨 (선언 전에 호출 가능)
- 이름이 필수
- Statement이므로 값으로 사용 불가

### 2. 함수 표현식 (Function Expression) - Expression

#### 익명 함수 표현식

```javascript
const greet = function() {
  return "Hello!";
};

console.log(greet());  // "Hello!"
```

#### 네임드 함수 표현식

```javascript
const greet = function myName() {
  return "Hello!";
};

console.log(greet.name);  // "myName"
```

### 함수 선언 vs 표현식 판별 규칙

> 값이 들어올 곳에 함수를 선언하면 **표현식**, 그렇지 않으면 **선언문**

```javascript
// 함수 선언
function a() {}  // 전역 레벨

if (true) {
  function b() {}  // 블록 최상위 레벨
}

function outer() {
  function inner() {}  // 블록 최상위 레벨
}

// 함수 표현식
const func = function() {};  // 값으로 할당

myFunc(function() {});  // 함수 인자

function outer() {
  return function inner() {};  // 반환값
}

// ❌ 에러
function() {}  // SyntaxError: 함수 선언문은 이름이 필요
```

### 복잡한 예제

```javascript
function a() {
  return function b() {
    function c() {}  // 함수 선언 (블록 최상위)
  }  // 함수 b는 표현식 (반환값)
}

// a: 함수 선언
// b: 네임드 함수 표현식 (return의 값)
// c: 함수 선언 (함수 b 블록의 최상위)
```

## 표현식 문장 (Expression Statement)

표현식 뒤에 **세미콜론**을 붙이면 표현식 문장이 됩니다.

```javascript
// 표현식
1 + 1

// 표현식 문장
1 + 1;

// 표현식으로 사용 가능
console.log(1 + 1);  // ✅

// 표현식 문장으로 사용 불가
console.log(1 + 1;);  // ❌ Syntax Error
```

### 예제

```javascript
// 표현식: 값이 필요한 곳에 사용
function add(x) {
  return x;
}

add(1 + 1);  // ✅ 표현식을 인자로 전달

true ? 1 + 1 : 2 + 2;  // ✅ 표현식을 삼항 연산자에 사용

// 표현식 문장: 값이 필요한 곳에 사용 불가
1 + 1;  // 표현식 문장 (독립적으로만 사용 가능)
```

## 세미콜론 vs 콤마 연산자

### 세미콜론 (;)

여러 줄의 문장을 한 줄에 넣을 수 있습니다.

```javascript
const a = 1; function b() {}; const c = 2;
```

### 콤마 연산자 (,)

여러 개의 **표현식**을 연결하고, **마지막 표현식만** 반환합니다.

```javascript
console.log((1 + 2, 3, 4));  // 4

console.log((1, 6 / 3, function() {}));  // function() {}

console.log((1, true ? 1 + 1 : 2 + 2));  // 2
```

**괄호 필요성**:

```javascript
// ✅ 괄호로 묶어서 하나의 값으로 전달
console.log((1, 2, 3));  // 3

// ❌ 괄호 없으면 각각을 인자로 전달
console.log(1, 2, 3);  // 1 2 3 (세 개의 인자)
```

### 함수에서의 콤마 연산자

```javascript
function getLastValue() {
  return 1, 2, 3, 4;
}

console.log(getLastValue());  // 4
```

모든 표현식이 왼쪽에서 오른쪽으로 평가되지만, **마지막 값만 반환**됩니다.

## IIFE (Immediately Invoked Function Expression)

**즉시 실행 함수 표현식**: 정의와 동시에 실행되는 함수

### 기본 형태

```javascript
// ❌ 함수 선언문은 즉시 실행 불가
function() {}  // SyntaxError

// ✅ 괄호로 묶으면 표현식이 됨
(function() {})  // function() {}를 반환

// ✅ 즉시 실행
(function() {
  console.log("실행됨!");
})();  // "실행됨!"
```

### 활용 예제

```javascript
// 반환값 받기
const result = (function() {
  return 1 + 2;
})();

console.log(result);  // 3

// 인자 전달
const greeting = (function(name) {
  return `Hello, ${name}!`;
})("Alice");

console.log(greeting);  // "Hello, Alice!"

// 로그 출력
console.log((function() {
  return "즉시 실행 결과";
})());  // "즉시 실행 결과"
```

### IIFE의 활용

```javascript
// 1. 변수 스코프 격리 (ES6 이전)
(function() {
  var privateVar = "외부에서 접근 불가";
  console.log(privateVar);  // "외부에서 접근 불가"
})();

// console.log(privateVar);  // ReferenceError

// 2. 모듈 패턴
const myModule = (function() {
  let privateCounter = 0;

  return {
    increment() {
      privateCounter++;
    },
    getCount() {
      return privateCounter;
    }
  };
})();

myModule.increment();
console.log(myModule.getCount());  // 1
```

> 💡 **참고**: ES6에서는 블록 스코프(`let`, `const`)와 모듈 시스템이 도입되어 IIFE의 필요성이 줄어들었습니다.

## Object Literal vs Block Statement

`{}`는 문맥에 따라 **객체 리터럴** 또는 **블록 문장**이 될 수 있습니다.

### Label (레이블)

```javascript
// 레이블: 반복문 제어에 유용
outerLoop: {
  for (let i = 0; i < 2; i++) {
    for (let n = 0; n < 2; n++) {
      break outerLoop;  // 바깥 루프까지 탈출
    }
  }
}

console.log("루프 종료");
```

### 블록 문장 (Block Statement)

```javascript
// 블록 문장: 여러 문장을 그룹화
{
  let a = "hello";
  console.log(a);
  1 + 1;
}  // 마지막 표현식 값: 2

// 레이블을 붙인 블록 문장
myLabel: {
  let x = 10;
  console.log(x);
}

// ❌ 레이블은 변수로 사용 불가
myLabel: function a() {}
console.log(myLabel);  // ReferenceError: myLabel is not defined
```

### Object Literal vs Block Statement 구분

```javascript
// Object Literal (객체 리터럴)
console.log({ a: "b" });  // { a: "b" }

const obj = { a: "b" };   // ✅ 값으로 사용

// Block Statement (블록 문장)
{ let a = "b"; func(); 1 + 1 }  // 독립적으로만 사용

console.log({ let a = "b" });  // ❌ SyntaxError
const x = { let a = "b" };     // ❌ SyntaxError
```

**블록 문장은 값이 아니므로** 값이 필요한 곳에 사용할 수 없습니다.

### 흥미로운 예제

```javascript
{} + 1   // 1
{1} + 2   // 2
{1 + 1} + 2   // 2
{1 + 1} - 2   // 0
```

**이유**:
- `{}`는 블록 문장으로 해석됨
- 블록 문장은 값을 반환하지 않으므로 암묵적으로 `0`으로 강제 변환
- `0 + 1 = 1`, `0 + 2 = 2`, `0 - 2 = -2`... 가 아니라 블록이 무시되고 숫자 연산만 수행

```javascript
// 위 코드는 사실상 다음과 같이 해석됨
{}      // 빈 블록 (무시됨)
+1      // 단항 플러스 연산자: 1

{1}     // 블록 (무시됨)
+2      // 2

{1+1}   // 블록 (무시됨)
+2      // 2
```

## 실전 활용 팁

### 1. 표현식을 선호하세요

```javascript
// ❌ Statement 지향적
let result;
if (condition) {
  result = "yes";
} else {
  result = "no";
}

// ✅ Expression 지향적 (더 간결)
const result = condition ? "yes" : "no";
```

### 2. 함수형 프로그래밍에서는 표현식 중심

```javascript
// Statement 사용 (명령형)
const numbers = [1, 2, 3, 4, 5];
const doubled = [];
for (let i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}

// Expression 사용 (함수형, 선언적)
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
```

### 3. JSX에서는 표현식만 사용 가능

```jsx
// ❌ JSX 안에서 Statement 사용 불가
function MyComponent({ isLoggedIn }) {
  return (
    <div>
      {if (isLoggedIn) { <p>Welcome!</p> }}  // SyntaxError
    </div>
  );
}

// ✅ 표현식(삼항 연산자) 사용
function MyComponent({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <p>Welcome!</p> : <p>Please login</p>}
    </div>
  );
}

// ✅ 또는 논리 연산자
function MyComponent({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn && <p>Welcome!</p>}
    </div>
  );
}
```

### 4. 화살표 함수의 간결한 문법

```javascript
// 블록 사용 (Statement) - return 필요
const double = (n) => {
  return n * 2;
};

// 표현식 (Expression) - return 불필요
const double = (n) => n * 2;

// 객체 반환 시 괄호로 묶기 (블록과 구분)
const createUser = (name) => ({ name: name });  // ✅
const createUser = (name) => { name: name };    // ❌ undefined 반환
```

## 마치며

Expression과 Statement의 차이를 이해하는 것은 JavaScript를 제대로 다루는 핵심입니다.

### 핵심 요약

```markdown
✅ Expression (표현식)
- 값을 산출
- 값이 필요한 곳 어디든 사용 가능
- 함수 인자, 할당, 연산 등에 사용

✅ Statement (문장)
- 동작을 수행
- 값으로 사용 불가
- if, for, while, switch 등

✅ 함수
- 선언문: 최상위 레벨
- 표현식: 값이 필요한 곳
- IIFE: 즉시 실행 패턴

✅ 실전 팁
- 가능하면 표현식 사용 (더 함수형)
- JSX에서는 표현식만 가능
- 화살표 함수로 간결하게
```

이러한 개념을 명확히 이해하면, 더 간결하고 예측 가능한 JavaScript 코드를 작성할 수 있습니다.

## 참고 자료

- [MDN - Expressions and Operators](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Expressions_and_Operators)
- [MDN - Statements and Declarations](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements)
- [MDN - IIFE](https://developer.mozilla.org/ko/docs/Glossary/IIFE)
- [MDN - Arrow Functions](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [JavaScript: What is the meaning of this?](https://web.dev/javascript-this/)
- [You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/tree/1st-ed/scope%20%26%20closures)
