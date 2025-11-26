---
title: JavaScript Scope 완벽 가이드 - 변수의 유효 범위 이해하기
date: 2020-07-04 20:28:00 +0900
categories: [개발, JavaScript]
tags: [javascript, scope, closure]
author: changsu
toc: true
comments: true
description: JavaScript Scope의 개념부터 실전 활용까지. Block Scope, Global Scope, 좋은 Scoping 습관을 실전 예제와 함께 알아봅니다. let, const, var의 스코프 차이를 이해하고 클로저와 네임스페이스 패턴을 활용한 변수 충돌 방지 기법을 배웁니다. 전역 변수 오염을 막는 실무 코딩 습관을 익힙니다.
---

## 들어가며

JavaScript에서 **Scope**란 **변수가 어디까지 쓰일 수 있는지**의 범위를 의미합니다.

`is not defined` 에러를 경험해보셨나요? 변수를 분명히 선언했는데도 이 에러가 발생한다면, 해당 변수가 선언된 영역(block)에 접근할 수 없기 때문입니다.

> Scope는 문법이 아니라 **개념**입니다. 변수의 생명주기와 접근 가능 범위를 결정하는 핵심 개념이죠.
{: .prompt-info }

## Block이란?

Scope를 이해하기 전에 먼저 **Block**의 개념을 알아야 합니다.

**Block**은 중괄호(`{}`, curly brace)로 감싸진 코드 영역을 말합니다.

```javascript
// Function Block
function hi() {
  return 'i am block';
}

// For Loop Block
for (let i = 0; i < 10; i++) {
  count++;
}

// If Statement Block
if (i === 1) {
  let j = 'one';
  console.log(j);
}
```

### Block의 특징

| 특징 | 설명 |
|------|------|
| **코드 영역** | `{}`로 감싸진 모든 코드 영역 |
| **변수 범위** | Block 내부에서 선언된 변수는 해당 Block 내부에서만 사용 가능 |
| **지역 변수** | Block 내부에서 정의된 변수를 Local(지역) 변수라고 부름 |

> Block 내부에서 선언된 변수는 Block 외부에서 접근할 수 없습니다.
{: .prompt-warning }

## Global(전역) Scope

**Scope**는 변수가 선언되고 사용할 수 있는 공간입니다. Scope 외부(block 밖)에서는 특정 scope의 변수에 접근할 수 없습니다.

Block 밖인 **Global Scope**에서 만든 변수를 **Global Variable(전역 변수)**라고 하며, 코드 어디서든 접근 가능합니다.

```javascript
const color = 'red';
console.log(color);  // 'red'

function returnColor() {
  console.log(color);  // 'red' - 함수 내부에서도 접근 가능
  return color;
}

console.log(returnColor());  // 'red'
```

### Global vs Local Scope

| 구분 | 선언 위치 | 접근 범위 | 생명주기 |
|------|----------|----------|----------|
| **Global Scope** | Block 외부 | 코드 어디서든 | 프로그램 종료까지 |
| **Local Scope** | Block 내부 | 해당 Block 내부만 | Block 종료까지 |

> Global 변수는 `returnColor` 함수의 Block에서도 접근이 가능하기 때문에 정상적으로 'red'를 반환합니다.
{: .prompt-info }

## Scope의 오염

Global 변수를 쓰면 여기저기서 접근하기 쉬워서 편리하다고 생각할 수 있지만, 너무 남용하면 프로그램에 심각한 문제를 일으킬 수 있습니다.

### Namespace란?

**Namespace**는 변수 이름을 사용할 수 있는 범위를 의미합니다. Scope과 같은 의미이며, 특히 변수 이름에 대해 이야기할 때 사용합니다.

### Global 변수의 문제점

```javascript
const satellite = 'The Moon';
const galaxy = 'The Milky Way';
let stars = 'North Star';

const callMyNightSky = () => {
  stars = 'Sirius';  // ⚠️ let 키워드 없이 재할당

  return 'Night Sky: ' + satellite + ', ' + stars + ', ' + galaxy;
};

console.log(callMyNightSky());  // Night Sky: The Moon, Sirius, The Milky Way
console.log(stars);              // 'Sirius' - Global 변수가 변경됨!
```

**문제 발생 과정:**

1. `stars`라는 Global 변수가 있음
2. `callMyNightSky` 함수에서 새로운 변수를 선언하려 했지만 `let` 키워드를 깜빡함
3. `callMyNightSky`를 호출하면 Global 변수 `stars`에 "Sirius"가 할당됨
4. 다른 함수에서 `stars`를 사용할 때 예상치 못한 "Sirius" 값을 사용하게 됨

> 함수가 몇십 개 있는 상황에서 Global 변수를 남용하면 어디서 어떻게 값이 수정될지 추적하기 어렵습니다.
{: .prompt-warning }

## 좋은 Scoping 습관

Global 변수가 여기저기서 수정되는 것을 방지하기 위해 **변수들은 Block Scope으로 최대한 나눠놔야 합니다.**

### Tight Scoping의 장점

**타이트한 Scope**(Tightly Scoping)의 변수는 코드 품질을 향상시킵니다:

| 장점 | 설명 |
|------|------|
| **가독성 향상** | 코드가 Block으로 명확하게 구분되어 이해하기 쉬움 |
| **유지보수 용이** | 각각의 기능별로 Block이 나뉘어 있어 수정이 쉬움 |
| **메모리 절약** | Block이 끝나면 Local 변수가 소멸되어 메모리 효율적 |
| **버그 예방** | 변수의 영향 범위가 제한되어 예상치 못한 수정 방지 |

> 💡 **핵심 원칙**: Global 변수는 최대한 사용하지 말고, `{}` 내에서 `let`, `const`를 사용하여 변수를 새로 선언하세요.
{: .prompt-tip }

## 실전 예제: Scope 오염 해결

### ❌ 잘못된 예: 같은 이름의 변수 사용

```javascript
function logSkyColor() {
  const dusk = true;
  let myColor = 'blue';

  if (dusk) {
    let myColor = 'pink';  // ⚠️ 같은 이름의 변수 재선언
    console.log(myColor);   // 'pink'
  }

  console.log(myColor);     // 'blue'
}

console.log(myColor);       // ReferenceError: myColor is not defined
```

**문제점 분석:**

- `if`문과 `function` 모두 Block Scope를 잘 사용했지만, 같은 이름의 변수를 사용하여 혼란 발생
- `if`문 내부의 `myColor`와 함수 레벨의 `myColor`는 서로 다른 변수지만, 코드 가독성이 떨어짐
- 함수 외부에서 `myColor`를 참조하면 에러 발생 (정상 동작이지만 의도 파악이 어려움)

### ✅ 개선된 예: 명확한 변수명 사용

```javascript
function logSkyColor() {
  const dusk = true;
  let skyColor = 'blue';

  if (dusk) {
    let duskColor = 'pink';      // ✅ 의미 있는 다른 이름 사용
    console.log(duskColor);       // 'pink'
  }

  console.log(skyColor);          // 'blue'
}

logSkyColor();
```

> 새로운 Block에서 변수를 선언할 때는 항상 **명확하고 다른 이름**으로 변수를 선언해야 합니다.
{: .prompt-tip }

## Scope Best Practices

### 1. 변수 선언 우선순위

```javascript
// ✅ 좋은 예
function calculateTotal(items) {
  const TAX_RATE = 0.1;  // Block 내부에서 const 사용
  let subtotal = 0;      // Block 내부에서 let 사용

  items.forEach(item => {
    subtotal += item.price;  // 상위 Block의 변수 접근
  });

  return subtotal * (1 + TAX_RATE);
}
```

### 2. Global 변수 최소화

```javascript
// ❌ 나쁜 예
let totalPrice = 0;
let totalQuantity = 0;

function addItem(price, quantity) {
  totalPrice += price;
  totalQuantity += quantity;
}

// ✅ 좋은 예
function createCart() {
  let items = [];

  return {
    addItem(price, quantity) {
      items.push({ price, quantity });
    },
    getTotal() {
      return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    }
  };
}

const cart = createCart();
```

### 3. 함수 매개변수 활용

```javascript
// ❌ 나쁜 예
let userName = 'John';

function greet() {
  console.log(`Hello, ${userName}!`);
}

// ✅ 좋은 예
function greet(userName) {
  console.log(`Hello, ${userName}!`);
}

greet('John');
```

## 핵심 정리

### Scope 개념 요약

| 개념 | 설명 |
|------|------|
| **Scope** | 변수가 유효한 범위 |
| **Block** | `{}`로 감싸진 코드 영역 |
| **Global Scope** | 어디서든 접근 가능한 최상위 범위 |
| **Local Scope** | Block 내부에서만 접근 가능한 범위 |
| **Namespace** | 변수 이름을 사용할 수 있는 범위 |

### 좋은 Scoping 원칙

1. **Global 변수는 최소화**하라
2. **변수는 사용하는 곳에서 가장 가까운 Block 내부에 선언**하라
3. **의미 있고 명확한 변수명**을 사용하라
4. **`const`를 우선적으로 사용**하고, 재할당이 필요한 경우에만 `let`을 사용하라

> Scope를 올바르게 이해하고 활용하면 버그가 적고, 유지보수하기 쉬운 코드를 작성할 수 있습니다.
{: .prompt-info }

## 자주 묻는 질문 (FAQ)

### Q1. let, const, var의 차이점은 무엇인가요?

**A:** 스코프와 재할당 가능 여부가 다릅니다.

| 구분 | var | let | const |
|------|-----|-----|-------|
| **스코프** | Function Scope | Block Scope | Block Scope |
| **재할당** | 가능 | 가능 | 불가능 |
| **재선언** | 가능 | 불가능 | 불가능 |
| **호이스팅** | undefined | TDZ | TDZ |
| **권장도** | ❌ | ✅ | ✅✅ |

```javascript
// var - Function Scope
if (true) {
  var x = 10;
}
console.log(x); // 10 (접근 가능!)

// let/const - Block Scope
if (true) {
  let y = 20;
  const z = 30;
}
console.log(y); // ReferenceError
console.log(z); // ReferenceError
```

### Q2. 전역 변수는 왜 피해야 하나요?

**A:** 다음과 같은 문제가 발생할 수 있습니다:
1. **변수 충돌**: 다른 코드와 변수명이 겹칠 수 있음
2. **메모리 누수**: 전역 변수는 계속 메모리에 남음
3. **디버깅 어려움**: 어디서든 변경 가능해 추적이 어려움
4. **테스트 어려움**: 독립적인 테스트가 불가능

```javascript
// ❌ 나쁜 예
let count = 0; // 전역 변수
function increment() {
  count++;
}

// ✅ 좋은 예
function createCounter() {
  let count = 0; // 지역 변수
  return {
    increment() { count++; },
    getCount() { return count; }
  };
}
```

### Q3. 클로저(Closure)란 무엇인가요?

**A:** 함수가 자신이 생성될 때의 스코프를 기억하는 현상입니다.

```javascript
function makeCounter() {
  let count = 0; // 외부 함수의 변수

  return function() {
    count++; // 외부 변수에 접근!
    return count;
  };
}

const counter1 = makeCounter();
console.log(counter1()); // 1
console.log(counter1()); // 2

const counter2 = makeCounter();
console.log(counter2()); // 1 (독립적인 count)
```

**활용 사례:**
- 데이터 은닉 (private 변수)
- 콜백 함수
- 이벤트 핸들러

### Q4. Block Scope가 왜 중요한가요?

**A:** 코드의 예측 가능성과 안정성을 높입니다.

```javascript
// ❌ var의 문제점
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 출력: 3, 3, 3 (모두 같은 i 참조)

// ✅ let의 해결
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 출력: 0, 1, 2 (각각 독립적인 i)
```

### Q5. 함수 안에서 선언한 변수를 밖에서 사용할 수 있나요?

**A:** 아니요, 함수 스코프 때문에 불가능합니다.

```javascript
function test() {
  let message = "Hello";
}

test();
console.log(message); // ReferenceError
```

**해결 방법:**
1. 함수에서 값을 반환
2. 외부 변수 사용 (권장하지 않음)
3. 객체나 배열로 반환

```javascript
// ✅ 권장: 값 반환
function getMessage() {
  let message = "Hello";
  return message;
}

const result = getMessage();
console.log(result); // "Hello"
```

### Q6. TDZ(Temporal Dead Zone)란 무엇인가요?

**A:** `let`과 `const` 변수의 선언 전 접근 불가 구간입니다.

```javascript
console.log(x); // ReferenceError: TDZ
let x = 10;

// var는 TDZ가 없음 (undefined 반환)
console.log(y); // undefined
var y = 20;
```

**TDZ의 장점:**
- 변수 선언 전 사용을 방지
- 버그를 조기에 발견
- 코드의 명확성 향상

### Q7. 네임스페이스 패턴이 무엇인가요?

**A:** 전역 변수 충돌을 방지하는 패턴입니다.

```javascript
// ❌ 전역 변수 오염
let name = "App";
let version = "1.0";
let config = {};

// ✅ 네임스페이스 패턴
const MyApp = {
  name: "App",
  version: "1.0",
  config: {},
  init() {
    console.log(`${this.name} v${this.version}`);
  }
};

MyApp.init();
```

**모던 JavaScript 대안:**
- ES6 Modules (`import`/`export`)
- IIFE (즉시 실행 함수)

```javascript
// ES6 Module
// config.js
export const config = { ... };

// app.js
import { config } from './config.js';
```

## 참고 자료

- [MDN - Scope](https://developer.mozilla.org/ko/docs/Glossary/Scope)
- [MDN - Block statement](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/block)
- [JavaScript.info - Variable scope](https://ko.javascript.info/closure#ref-278)
